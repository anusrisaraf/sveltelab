<script>
  import * as d3 from "d3";

  export let data = [];
  export let title = "Lines of code by language";

  let width = 520;
  let height = 180;

  let margin = { top: 28, right: 100, bottom: 44, left: 96 };
  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;

  $: xScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([0, innerWidth]);

  $: yScale = d3
    .scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerHeight])
    .padding(0.3);

  $: colorScale = d3
    .scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  $: maxValue = d3.max(data, (d) => d.value) ?? 0;
  $: maxBars = data.filter((d) => d.value === maxValue && maxValue > 0);
  $: mainBar = maxBars[0];

  /** At most 10 ticks; fewer when max bar height is smaller (whole-number ticks). */
  $: tickCount = Math.min(Math.max(maxValue, 1), 10);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3
        .axisBottom(xScale)
        .ticks(tickCount)
        .tickFormat((d) => (Number.isInteger(d) ? d : ""))
    );
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="chart-container">
  <svg viewBox={`0 0 ${width} ${height}`}>
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2 + 4}
      text-anchor="middle"
      class="chart-title"
    >
      {title}
    </text>

    <g
      transform={`translate(${margin.left}, ${margin.top + innerHeight})`}
      bind:this={xAxis}
    />
    <g transform={`translate(${margin.left}, ${margin.top})`} bind:this={yAxis} />

    <g transform={`translate(${margin.left}, ${margin.top})`}>
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}

      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 12}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of code
      </text>

      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 22}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Language
      </text>

      {#if maxBars.length && mainBar}
        {#each maxBars as bar}
          <rect
            x={0}
            y={yScale(bar.label)}
            width={xScale(bar.value)}
            height={yScale.bandwidth()}
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          />
        {/each}
        {#key mainBar.label}
          <text
            x={xScale(mainBar.value) + 8}
            y={yScale(mainBar.label) + yScale.bandwidth() / 2}
            text-anchor="start"
            dominant-baseline="middle"
            class="annotation"
          >
            Most LOC
          </text>
        {/key}
      {/if}
    </g>
  </svg>

  <ul class="legend">
    {#each data as d}
      <li style={`--color: ${colorScale(d.label)}`}>
        <span class="swatch"></span>
        {d.label}
      </li>
    {/each}
  </ul>
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .chart-container {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
    margin: 0.5rem auto 1.5rem;
    max-width: min(960px, 100%);
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    grid-template-columns: 1fr;
    gap: 0.35rem;
    font-size: 0.72rem;
    min-width: 7rem;
  }

  .swatch {
    width: 0.65rem;
    height: 0.65rem;
    border-radius: 2px;
    background-color: var(--color);
    margin-right: 0.4rem;
    flex-shrink: 0;
  }

  .legend li {
    display: flex;
    align-items: center;
  }

  .chart-title {
    font-size: 0.82rem;
    font-weight: 600;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.68rem;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.6rem;
    font-style: italic;
    fill: currentColor;
  }
</style>
