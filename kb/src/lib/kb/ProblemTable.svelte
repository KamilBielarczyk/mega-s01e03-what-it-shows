<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in a ticket: the causes from causes.ts, plus the reports the desk read as a
  // one-day move (desk-notes.json), taken out of the cause their tenant files them under.
  let { ticket, issues = {} }: { ticket: number; issues?: Record<string, string> } = $props();

  const ONE_DAY = 'one-day move';
  const problemOf = (r: Report) =>
    notes.oneDayMove.includes(r.id) ? ONE_DAY : (causeOf(r, tenantOf(r)) ?? 'unplaced');

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    [...Map.groupBy(merged, problemOf).entries()]
      .map(([problem, rs]) => {
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        return {
          problem,
          reports: rs.length,
          tenants: new Set(rs.map((r) => r.tenant)).size,
          channels: [...Map.groupBy(rs, (r) => r.channel).entries()]
            .sort((a, b) => b[1].length - a[1].length)
            .map(([c, xs]) => `${c} ×${xs.length}`)
            .join(' · '),
          firstSeen: days[0],
          lastSeen: days[days.length - 1],
        };
      })
      .sort((a, b) => b.reports - a.reports),
  );
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong>
  {rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>channels</th><th>tenants</th><th>seen</th><th>issue</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.reports}</td>
        <td class="muted">{row.channels}</td>
        <td>{row.tenants}</td>
        <td>{row.firstSeen} → {row.lastSeen}</td>
        <td>
          {#if issues[row.problem]}<a href={issues[row.problem]}>#{issues[row.problem].split('/').pop()}</a>
          {:else}<span class="none">none yet</span>{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
