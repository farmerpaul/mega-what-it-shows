<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Problem name → filed ticket URL.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));

  // The cause table files one-day moves by tenant; here they get their own row.
  const rows = $derived(
    [
      ...groupByCause(merged.filter((r) => !moved.includes(r)), tenantOf).map((g) => ({ name: String(g.cause), reports: g.reports })),
      { name: 'one-day', reports: moved },
    ].map(({ name, reports: rs }) => {
      const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
      return { name, count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, first: days[0], last: days[days.length - 1] };
    }),
  );
</script>

<p><strong>{merged.length}</strong> unique reports · <strong>{rows.length}</strong> problems</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as r (r.name)}
      <tr>
        <td><code>{r.name}</code></td>
        <td>{r.count}</td>
        <td>{r.tenants}</td>
        <td>{r.first} → {r.last}</td>
        <td><a href={issues[r.name]}>{issues[r.name]?.split('/').pop() ? `#${issues[r.name].split('/').pop()}` : ''}</a></td>
      </tr>
    {/each}
  </tbody>
</table>
