<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, causeOf, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const rows = $derived((reports as Report[]).filter((r) => r.mergedInto === ticket));
  const merged = $derived(uniqueById(rows));
  const groups = $derived(groupByCause(merged, tenantOf));

  // The desk's own notes: reports it read as a one-day move, whatever cause they sit under.
  const moved = $derived(groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0));

  // Intake guesses a cause from the reporter's browser. How often is that guess not the cause?
  const intakeWrong = $derived(
    merged.filter((r) => r.meta.intakeTag !== (causeOf(r, tenantOf(r)) ?? 'unplaced')).length,
  );

  const channels = (g: { reports: Report[] }) => {
    const n: Record<string, number> = {};
    for (const r of g.reports) n[r.channel] = (n[r.channel] ?? 0) + 1;
    return Object.entries(n)
      .sort((a, b) => b[1] - a[1])
      .map(([c, k]) => `${c} ${k}`)
      .join(' · ');
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket}, carried in
  <strong>{rows.length}</strong> exported rows —
  <strong>{rows.length - merged.length}</strong> of them arrive twice.
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>how they came in</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td><strong>{g.reports.length}</strong></td>
        <td>{g.tenants}</td>
        <td class="muted">{channels(g)}</td>
        <td class="muted">{g.firstSeen} → {g.lastSeen}</td>
      </tr>
    {/each}
    <tr class="inside">
      <td><code>one-day move</code></td>
      <td><strong>{moved}</strong></td>
      <td>{new Set(merged.filter((r) => notes.oneDayMove.includes(r.id)).map((r) => r.tenant)).size}</td>
      <td class="muted">counted inside the row above it</td>
      <td class="muted">from the desk's own notes</td>
    </tr>
  </tbody>
</table>

<p class="note">
  Intake's guess at the cause is not the cause for <strong>{intakeWrong}</strong> of these
  {merged.length} reports, in both directions.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.9em; }
  .inside td { background: var(--note-bg); }
  .inside td:first-child { padding-left: 1.5rem; }
</style>
