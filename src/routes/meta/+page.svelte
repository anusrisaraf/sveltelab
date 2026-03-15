<script>
  import { base } from "$app/paths";
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import BarHorizontal from "$lib/BarHorizontal.svelte";

  let barData = [];

  onMount(async () => {
    const locData = await d3.csv(`${base}/loc.csv`, (row) => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth)
    }));
    const grouped = d3.rollups(locData, (v) => v.length, (d) => d.type);
    barData = grouped.map(([type, count]) => ({ label: type, value: count }));
  });
</script>

<svelte:head>
  <title>Meta — Code Stats</title>
</svelte:head>

<div class="page-inner">
  <h1>Codebase Meta Stats</h1>
  <p>
    This page shows a quick breakdown of lines of code per language in this
    project, based on an automatically generated <code>loc.csv</code>.
  </p>

  {#if barData.length === 0}
    <p>Loading code statistics…</p>
  {:else}
    <BarHorizontal data={barData} />
  {/if}
</div>

