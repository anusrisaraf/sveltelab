<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 480;
  let height = 320;

  let margin = { top: 40, right: 40, bottom: 60, left: 120 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

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
    .padding(0.2);

  $: colorScale = d3
    .scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  $: maxValue = d3.max(data, (d) => d.value) ?? 0;
  $: maxBars = data.filter((d) => d.value === maxValue);
  $: mainBar = maxBars[0];

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3
        .axisBottom(xScale)
        .ticks(5)
        .tickFormat((d) => (Number.isInteger(d) ? d : ""))
    );
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="chart-container">
  <svg viewBox={`0 0 ${width} ${height}`}>
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title"
    >
      Lines of Code by Language
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
        y={innerHeight + margin.bottom - 15}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of code
      </text>

      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 30}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Language
      </text>

      {#if maxBars.length}
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
        {#if mainBar}
          {#key mainBar.label}
            <line
              x1={xScale(mainBar.value)}
              y1={yScale(mainBar.label) + yScale.bandwidth() / 2}
              x2={xScale(mainBar.value) + 40}
              y2={yScale(mainBar.label) + yScale.bandwidth() / 2}
              stroke="currentColor"
              stroke-width="1"
            />
            <text
              x={xScale(mainBar.value) + 45}
              y={yScale(mainBar.label) + yScale.bandwidth() / 2}
              text-anchor="start"
              dominant-baseline="middle"
              class="annotation"
            >
              Most LOC
            </text>
          {/key}
        {/if}
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
    gap: 1.5rem;
    align-items: flex-start;
    margin-block: 2rem;
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
  }

  .chart-title {
    font-size: 1rem;
    font-weight: 600;
  }

  .axis-label {
    font-size: 0.8rem;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.7rem;
    font-style: italic;
     fill: currentColor;
  }
</style>

