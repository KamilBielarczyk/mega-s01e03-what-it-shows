<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type CauseGroup, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in a ticket, counted the desk's way (CauseTable): each report under its
  // tenant's cause, and the reports the desk read as a one-day move (desk-notes.json) counted
  // there too, with a row of their own.
  type Issue = { href: string; state: string };
  let { ticket, issues = {} }: { ticket: number; issues?: Record<string, Issue> } = $props();

  const ONE_DAY = 'one-day move';

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const causes = $derived(groupByCause(merged, tenantOf));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));

  const row = (problem: string, rs: Report[], alsoIn: CauseGroup[] = []) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return {
      problem,
      reports: rs.length,
      tenants: new Set(rs.map((r) => r.tenant)).size,
      firstSeen: days[0],
      lastSeen: days[days.length - 1],
      alsoIn: alsoIn.filter((g) => readAs(g, rs.map((r) => r.id)) > 0).map((g) => g.cause),
    };
  };
  const rows = $derived([
    ...causes.map((g) => row(g.cause, g.reports)),
    ...(moved.length ? [row(ONE_DAY, moved, causes)] : []),
  ]);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong>
  {rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>issue</th><th>state</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      {@const issue = issues[r.problem]}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.reports}{#if r.alsoIn.length}{' '}<span class="muted">(also in {r.alsoIn.join(', ')})</span>{/if}</td>
        <td>{r.tenants}</td>
        <td>{r.firstSeen} → {r.lastSeen}</td>
        <td>
          {#if issue}<a href={issue.href}>#{issue.href.split('/').pop()}</a>
          {:else}<span class="open">none yet</span>{/if}
        </td>
        <td class:open={issue?.state === 'open'}>{issue?.state ?? '—'}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each causes as g (g.cause)}
  {@const n = readAs(g, notes.oneDayMove)}
  {#if n}
    <p class="muted">{n} reports read as a one-day move; counted under <code>{g.cause}</code>, where their tenant files them.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .open { color: var(--warn); font-weight: 600; }
</style>
