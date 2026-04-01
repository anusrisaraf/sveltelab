<script>
  import { base } from "$app/paths";
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import BarHorizontal from "$lib/BarHorizontal.svelte";
  import CommitScatter from "$lib/CommitScatter.svelte";

  /** Set to your repo’s commit URL prefix (Lab 7 handout uses vis-society/lab-7). */
  const COMMIT_URL_BASE = "https://github.com/vis-society/lab-7/commit/";

  let locData = [];
  let commits = [];
  let clickedCommits = [];
  let loaded = false;

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, (row) => ({
      ...row,
      line: Number(row.line),
      depth: Number(row.depth),
      length: Number(row.length),
      date: new Date(row.date + "T00:00" + row.timezone),
      datetime: new Date(row.datetime)
    }));

    commits = d3.groups(locData, (d) => d.commit).map(([commit, lines]) => {
      const first = lines[0];
      const { author, date, time, timezone, datetime } = first;
      return {
        id: commit,
        url: COMMIT_URL_BASE + commit,
        author,
        date,
        time,
        timezone,
        datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines
      };
    });

    commits = d3.sort(commits, (d) => -d.totalLines);
    loaded = true;
  });

  $: allLangs =
    locData.length > 0
      ? Array.from(new Set(locData.map((d) => d.type)))
      : [];

  $: barData =
    locData.length === 0
      ? []
      : (() => {
          const lines =
            clickedCommits.length > 0
              ? clickedCommits.flatMap((c) => c.lines)
              : locData;
          const counts = d3.rollup(
            lines,
            (v) => v.length,
            (d) => d.type
          );
          return allLangs.map((label) => ({
            label,
            value: counts.get(label) ?? 0
          }));
        })();

  $: barChartTitle =
    clickedCommits.length === 0
      ? "Website language breakdown (all lines)"
      : `Language breakdown (${clickedCommits.length} selected commit${
          clickedCommits.length === 1 ? "" : "s"
        })`;
</script>

<svelte:head>
  <title>Meta — Code Stats</title>
</svelte:head>

<div class="page-inner">
  <h1>Codebase Meta Stats</h1>
  <p>
    This page shows commits over time (scatterplot) and a breakdown of lines
    of code per language, based on <code>loc.csv</code> from
    <code>elocuent</code>.
  </p>

  {#if !loaded}
    <p>Loading code statistics…</p>
  {:else}
    <section class="meta-section">
      <h2>Commits by date &amp; time</h2>
      <p class="section-lead">
        Circle size encodes lines edited (area ∝ lines). Click to select
        commits; selected points stay highlighted and filter the bar chart
        below.
      </p>
      <CommitScatter bind:clickedCommits {commits} />
    </section>

    <section class="meta-section">
      <h2>Lines of code by language</h2>
      <BarHorizontal data={barData} title={barChartTitle} />
    </section>
  {/if}
</div>

<style>
  .meta-section {
    margin-bottom: 2rem;
  }

  .section-lead {
    max-width: 52ch;
    margin-bottom: 0.75rem;
    opacity: 0.92;
  }

  .meta-section h2 {
    font-size: 1.15rem;
    margin-bottom: 0.35rem;
  }
</style>
