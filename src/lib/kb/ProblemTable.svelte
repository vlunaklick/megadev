<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Counted the desk's way (causes.ts): each report under its cause by tenant. The one-day moves
  // (desk-notes.json) stay counted where they are filed and get their own row, footnoted.
  let { ticket, issues, repo }: { ticket: number; issues: Record<string, number>; repo: string } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const shift = $derived(groups.map((g) => ({ cause: g.cause, n: readAs(g, notes.oneDayMove) })).filter((s) => s.n));
  const link = (p: string) => `https://github.com/${repo}/issues/${issues[p]}`;
</script>

<p class="sum">
  <strong>{(reports as Report[]).length}</strong> rows in the export · <strong>{merged.length}</strong> unique reports
  merged into #{ticket} · <strong>{groups.length + (shift.length ? 1 : 0)}</strong> problems
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if issues[g.cause]}<a href={link(g.cause)}>{repo}#{issues[g.cause]}</a>{:else}none{/if}</td>
      </tr>
    {/each}
    {#each shift as s (s.cause)}
      <tr>
        <td><code>shift</code></td>
        <td>{s.n}*</td>
        <td colspan="2">one-day moves, filed under <code>{s.cause}</code></td>
        <td><a href={link('shift')}>{repo}#{issues.shift}</a></td>
      </tr>
    {/each}
  </tbody>
</table>

{#each shift as s (s.cause)}
  <p class="note">* {s.n} reports read as a one-day move; counted under <code>{s.cause}</code> by tenant, as the desk files them.</p>
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
