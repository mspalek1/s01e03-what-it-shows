<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === 4471));
  const groups = $derived(groupByCause(merged, tenantOf));
  const issue = {
    parser: { number: 1, href: 'https://github.com/mspalek1/s01e03-what-it-shows/issues/1', label: 'Date parser assumes month-first' },
    hint: { number: 2, href: 'https://github.com/mspalek1/s01e03-what-it-shows/issues/2', label: 'Booking-v3 date hint conflicts with locale' },
  } as const;
</script>

<p class="sum"><strong>{merged.length}</strong> unique reports · <strong>{groups.length}</strong> causes</p>
<table>
  <thead><tr><th>Cause</th><th>Reports</th><th>Tenants</th><th>Intake tags</th><th>First–last seen</th><th>Issue</th></tr></thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const ticket = g.cause === 'parser' || g.cause === 'hint' ? issue[g.cause] : null}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{Object.entries(g.taggedAs).map(([tag, n]) => `${tag} ×${n}`).join(' · ')}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if ticket}<a href={ticket.href}>{ticket.label}</a>{:else}Unassigned{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>
{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}<p class="note">{moved} desk-noted reports mention a one-day move; they remain counted under <code>{g.cause}</code>.</p>{/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  table { border-collapse: collapse; width: 100%; margin: 1rem 0; font-size: 0.92rem; }
  th, td { border-bottom: 1px solid var(--line); padding: 0.35rem 0.5rem; text-align: left; vertical-align: top; }
  code { font-family: var(--mono); font-size: 0.88em; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
