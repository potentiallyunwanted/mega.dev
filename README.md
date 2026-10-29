# #4471, split

Ticket #4471 on the support desk was merged from 412 reports about dates, and marked known: one
cause, one fix queued. It holds more than one fault. This repo is the split.

## The tickets

Each is an issue here, carrying the number of reports it covers.

| Ticket | Reports | What it is |
|---|---|---|
| [#1](../../issues/1) | 379 | The shared parser reads dates month first; every tenant on file writes the day first |
| [#2](../../issues/2) | 33 | `booking-v3` prints an `MM/DD/YYYY` hint to day-first tenants, then reads what they type their way |

### Counted inside #1

Five reports describe a one-day move rather than a month and day swapping places. The desk had
already spotted them and keeps their ids in `desk-notes.json`; they stay counted under the parser
and footnoted, which is how the desk counts them. [#3](../../issues/3) was opened for them and then
closed back into #1.

They still deserve reading before the parser fix is called done. One of the five describes an
invite that arrived before its own booking was made, and another a single meeting held twice
because two cities saw two different days. Neither is a month-first parse.

## The page

- `kb/src/routes/kb/incidents/4471-what-it-holds/+page.svx` — the page, at the path it takes in the
  kb
- `kb/src/lib/kb/SplitTable.svelte` — the component it needs; the kb keeps words in the page and
  logic in `src/lib`
- `rendered/4471-what-it-holds.html` — the page as the desk sees it, self-contained

The page states no number it does not compute. Every figure in it is derived from the export at
render time by `SplitTable`, which calls the desk's own `causes.ts`. The page is signed, so it
carries a desk ref.

## Counting

The export ships **427 rows for 412 reports**: the ticket system exports in overlapping pages, so
fifteen reports arrive twice. Counts here are taken after de-duplicating by `id`, which is what
`uniqueById` in the kb exists for. Any count read straight off the file is fifteen too high.

Causes are not assigned by hand. `causeOf` decides them from two facts the intake process never
looks up — which tenant sent the report, and which booking form that tenant runs. Intake instead
guesses from the reporter's browser at the moment the report arrives, and its guess disagrees with
the computed cause on 38 of the 412, in both directions.

## Reproducing the numbers

The figures come from the kb export, unmodified, run through the kb's own code:

1. Drop the export's `src/` into a SvelteKit project with mdsvex, aliasing `#lib` to `src/lib`.
2. Add the two files under `kb/` above at the same paths.
3. Build. The page prerenders, and the table is computed from the data.
