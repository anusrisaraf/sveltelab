<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 400;
  let height = 300;

  let margin = { top: 40, right: 150, bottom: 80, left: 60 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;

  $: xScale = d3
    .scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerWidth])
    .padding(0.2);

  $: yScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([innerHeight, 0]);

  $: colorScale = d3
    .scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  $: maxValue = d3.max(data, (d) => d.value) ?? 0;
  $: maxBars = data.filter((d) => d.value === maxValue);
  $: labelX = margin.left + innerWidth - 40;
  $: labelY = margin.top + innerHeight * 0.35;

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(
      d3
        .axisLeft(yScale)
        .tickFormat((d) => (Number.isInteger(d) ? d : ""))
        .tickValues(d3.range(0, (d3.max(data, (d) => d.value) || 0) + 1))
    );
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
      Projects per Year
    </text>

    <g
      transform={`translate(${margin.left}, ${margin.top + innerHeight})`}
      bind:this={xAxis}
    />
    <g transform={`translate(${margin.left}, ${margin.top})`} bind:this={yAxis} />

    <g transform={`translate(${margin.left}, ${margin.top})`}>
      {#each data as d}
        <rect
          x={xScale(d.label)}
          y={yScale(d.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(d.value)}
          fill={colorScale(d.label)}
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

      {#if maxBars.length}
        {#each maxBars as bar}
          <rect
            x={xScale(bar.label)}
            y={yScale(bar.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(bar.value)}
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          />
          <line
            x1={xScale(bar.label) + xScale.bandwidth()}
            y1={yScale(bar.value) +
              (innerHeight - yScale(bar.value)) / 2}
            x2={labelX}
            y2={labelY}
            stroke="currentColor"
            stroke-width="1"
          />
        {/each}
        <text
          x={labelX + 5}
          y={labelY}
          text-anchor="start"
          dominant-baseline="middle"
          class="annotation"
        >
          Year(s) with most projects
        </text>
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
    margin-bottom: 2rem;
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
    fill: currentColor;
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

