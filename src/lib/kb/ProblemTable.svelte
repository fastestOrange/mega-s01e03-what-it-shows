<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByProblem, type Problem } from '#lib/kb/problems';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // tickets: problem → URL of the ticket filed for it.
  let { ticket, tickets = {} }: { ticket: number; tickets?: Partial<Record<Problem, string>> } = $props();

  const rows = $derived(uniqueById(reports as Report[]));
  const groups = $derived(groupByProblem(rows, ticket, tenantOf, notes.oneDayMove));
  const merged = $derived(groups.reduce((n, g) => n + g.reports.length, 0));
  const label = (url: string) => url.replace(/^.*\/issues\//, '#');
</script>

<p class="sum">
  <strong>{merged}</strong> reports merged into #{ticket} ({(reports as Report[]).length} rows in the export) ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.channels.join(', ')}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if g.problem !== 'unplaced' && tickets[g.problem]}
            <a href={tickets[g.problem]}>{label(tickets[g.problem]!)}</a>
          {:else}
            <span class="none">no ticket</span>
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .none { color: var(--warn); font-weight: 600; }
</style>
