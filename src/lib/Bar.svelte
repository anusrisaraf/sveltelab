<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 420;
  let height = 260;

  let margin = { top: 40, right: 150, bottom: 80, left: 60 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;
  let selectedIndex = -1;
  let liveText = "";
  let showChart = true;

  $: xScale = d3
    .scaleBand()
    .domain(
      data
        .slice()
        .sort((a, b) => Number(a.label) - Number(b.label))
        .map((d) => d.label)
    )
    .range([0, innerWidth])
    .padding(0.2);

  $: yScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([innerHeight, 0]);

  $: colorScale = d3
    .scaleOrdinal()
    .domain(data.map((d) => d.label))
    .range(["#ccece6", "#99d8c9", "#66c2a4", "#41ae76", "#238b45"]);

  $: description = `A bar chart showing project counts by year. ${data
    .map((d) => `${d.label}: ${d.value} projects`)
    .join(", ")}.`;

  $: maxBar = d3.greatest(data, (d) => d.value);
  $: labelX = margin.left + innerWidth * 0.7;
  $: labelY = margin.top + innerHeight * 0.4;

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(
      d3
        .axisLeft(yScale)
        .tickFormat((d) => (Number.isInteger(d) ? d : ""))
        .tickValues(d3.range(0, (d3.max(data, (d) => d.value) || 0) + 1))
    );
  }

  function toggleBar(index, event) {
    if (!event.key || event.key === "Enter" || event.key === " ") {
      selectedIndex = index;
      const d = data[index];
      liveText = `${d.label}: ${d.value} projects selected.`;
    }
  }

  function toggleView() {
    showChart = !showChart;
    liveText = showChart ? "Bar chart view shown." : "Table view shown.";
  }
</script>

<button
  on:click={toggleView}
  aria-pressed={!showChart}
  aria-label="Toggle between bar chart and table view"
  class="toggle-button"
>
  {showChart ? "Show Table" : "Show Chart"}
</button>

{#if showChart}
  <div class="chart-container">
    <svg viewBox={`0 0 ${width} ${height}`} role="img" aria-labelledby="bar-title bar-desc">
      <title id="bar-title">Projects by Year</title>
      <desc id="bar-desc">{description}</desc>
      <text
        x={margin.left + innerWidth / 2}
        y={margin.top / 2}
        text-anchor="middle"
        class="chart-title"
      >
        Projects per Year
      </text>

      <g
        transform={`translate(${margin.left}, ${margin.top + innerHeight})`}
        bind:this={xAxis}
      />
      <g transform={`translate(${margin.left}, ${margin.top})`} bind:this={yAxis} />

      <g transform={`translate(${margin.left}, ${margin.top})`} class="container">
        {#each data as d, index}
          <rect
            x={xScale(d.label)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill={colorScale(d.label)}
            stroke="black"
            class:selected-bar={selectedIndex === index}
            opacity={selectedIndex === -1 || selectedIndex === index ? 1 : 0.45}
            tabindex="0"
            role="button"
            aria-label={`${d.label}: ${d.value} projects`}
            on:click={(e) => toggleBar(index, e)}
            on:keyup={(e) => toggleBar(index, e)}
          />
        {/each}

        <text
          x={innerWidth / 2}
          y={innerHeight + margin.bottom - 10}
          text-anchor="middle"
          class="axis-label"
        >
          Year
        </text>

        <text
          x={-(innerHeight / 2)}
          y={-margin.left + 20}
          text-anchor="middle"
          transform="rotate(-90)"
          class="axis-label"
        >
          Number of Projects
        </text>

        {#if maxBar}
          <rect
            x={xScale(maxBar.label)}
            y={yScale(maxBar.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(maxBar.value)}
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          />
          <line
            x1={xScale(maxBar.label) + xScale.bandwidth()}
            y1={yScale(maxBar.value) +
              (innerHeight - yScale(maxBar.value)) / 2}
            x2={labelX}
            y2={labelY}
            stroke="currentColor"
            stroke-width="1"
          />
          <text
            x={labelX + 5}
            y={labelY}
            text-anchor="start"
            dominant-baseline="middle"
            class="annotation"
          >
            My most prolific year
          </text>
        {/if}
      </g>
    </svg>

    <div class="legend-wrap" aria-label="Legend for projects by year">
    <p class="legend-title">Legend:</p>
    <ul class="legend">
      {#each data as d}
        <li style={`--color: ${colorScale(d.label)}`}>
          <span class="swatch"></span>
          {d.label} ({d.value})
        </li>
      {/each}
    </ul>
    </div>
  </div>
{:else}
  <table aria-label="Table showing project counts by year" class="data-table">
    <caption>Projects by Year</caption>
    <thead>
      <tr>
        <th id="year-header" scope="col">Year</th>
        <th id="projects-header" scope="col">Projects</th>
      </tr>
    </thead>
    <tbody>
      {#each data as d, i}
        <tr>
          <th id={`row-${i}`} scope="row">{d.label}</th>
          <td aria-labelledby={`row-${i} projects-header`}>{d.value}</td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

<p aria-live="polite" class="sr-only">{liveText}</p>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .chart-container {
    display: flex;
    gap: 1.5rem;
    align-items: flex-start;
    margin: 1.5rem auto 2rem;
    max-width: 640px;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    grid-template-columns: 1fr;
    gap: 0.5rem;
    font-size: 0.85rem;
  }

  .legend-wrap {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
  }

  .legend-title {
    margin: 0;
    font-size: 0.85rem;
    font-weight: 700;
    color: #111;
  }

  .swatch {
    width: 0.9rem;
    height: 0.9rem;
    border-radius: 2px;
    background-color: var(--color);
    margin-right: 0.5rem;
    flex-shrink: 0;
  }

  .legend li {
    display: flex;
    align-items: center;
    color: #111;
  }

  .chart-title {
    font-size: 1rem;
    font-weight: 600;
    fill: #111;
  }

  .axis-label {
    font-size: 0.8rem;
    fill: #111;
  }

  .annotation {
    font-size: 0.4rem;
    font-style: italic;
    fill: #111;
  }

  :global(.tick text),
  :global(.tick line),
  :global(.domain) {
    stroke: #111;
    fill: #111;
    color: #111;
  }

  .toggle-button {
    margin: 0 auto 0.75rem;
    display: block;
    border: 1px solid #444;
    background: #6a6a6a;
    color: #fff;
    padding: 0.45rem 0.75rem;
    border-radius: 3px;
    cursor: pointer;
    font-weight: 600;
  }

  rect {
    transition: 300ms;
    outline: none;
    stroke: black;
    stroke-width: 1;
  }

  svg:hover rect:not(:hover),
  .container:focus-within rect:not(:focus-visible) {
    opacity: 50%;
  }

  rect:focus-visible {
    stroke: white;
    stroke-width: 2px;
    stroke-dasharray: 4;
  }

  rect:hover {
    stroke: #ff7a00;
    stroke-width: 2.5px;
    stroke-dasharray: none;
  }

  rect.selected-bar {
    stroke: #ff7a00;
    stroke-width: 3px;
    stroke-dasharray: none;
  }

  .sr-only {
    position: absolute;
    left: -9999px;
    width: 1px;
    height: 1px;
    overflow: hidden;
  }

  .data-table {
    margin-top: 1rem;
    margin-bottom: 1rem;
    border-collapse: collapse;
    width: 100%;
    max-width: 30em;
  }

  .data-table caption {
    font-weight: bold;
    margin-bottom: 0.5em;
    text-align: left;
  }

  .data-table th,
  .data-table td {
    border: 1px solid #ccc;
    padding: 0.5em;
    text-align: left;
  }

  .data-table th {
    background-color: #f0f0f0;
  }

  .data-table tbody tr:hover {
    background: #fff7e8;
  }

  .data-table tbody tr:hover th,
  .data-table tbody tr:hover td {
    background: #fff7e8;
  }

  @media (prefers-color-scheme: dark) {
    .chart-title,
    .axis-label,
    .annotation,
    .legend,
    .legend-title,
    .legend li {
      fill: #f2f2f2;
      color: #f2f2f2;
    }

    :global(.tick text),
    :global(.tick line),
    :global(.domain) {
      stroke: #f2f2f2;
      fill: #f2f2f2;
      color: #f2f2f2;
    }

    .toggle-button {
      background: #6a6a6a;
      color: #fff;
      border-color: #8a8a8a;
    }

    .data-table {
      color: #f5f5f5;
      border-color: #646464;
    }

    .data-table caption {
      background: #3c3c3f;
      color: #f2f2f2;
      padding: 0.65rem 0.8rem;
      margin-bottom: 0.35rem;
    }

    .data-table th,
    .data-table td {
      border-color: #646464;
      background-color: #d3d3d3;
      color: #111;
    }

    .data-table th {
      background-color: #cfcfd2;
    }

    .data-table tbody tr:hover th,
    .data-table tbody tr:hover td {
      background: #f0e6d8;
    }
  }
</style>

