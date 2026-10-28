# mega-s01e03-what-it-shows

Support ticket #4471 merged 412 date reports as one issue. They are three problems.

| Problem | Reports | Issue |
|---|---|---|
| parser: day-first dates read month first | 374 | [#1](https://github.com/KamilBielarczyk/mega-s01e03-what-it-shows/issues/1) |
| hint: booking-v3 form hints MM/DD/YYYY to day-first tenants | 33 | [#2](https://github.com/KamilBielarczyk/mega-s01e03-what-it-shows/issues/2) |
| one-day move: bookings shift by one day | 5 | [#3](https://github.com/KamilBielarczyk/mega-s01e03-what-it-shows/issues/3) |

## The kb page

- [`kb/src/routes/kb/incidents/4471-split/+page.svx`](kb/src/routes/kb/incidents/4471-split/+page.svx): the page, signed. It goes into the support kb at the same path.
- [`kb/src/lib/kb/ProblemTable.svelte`](kb/src/lib/kb/ProblemTable.svelte): the table it uses. It counts from the kb's data at render time, with the kb's own cause rule (`causes.ts`) and the desk's one-day list (`desk-notes.json`).
- [`rendered/4471-split.html`](rendered/4471-split.html) and [`rendered/4471-split.png`](rendered/4471-split.png): the page as the desk sees it, built with SvelteKit and mdsvex from the kb export of 2026-10-04. Desk ref DR-EZSA-PHJA.
