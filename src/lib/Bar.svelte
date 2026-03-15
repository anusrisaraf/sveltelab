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
    .scaleOrdinal(d3.schemeTableau10)
    .domain(data.map((d) => d.label));

  // single bar with the maximum value (for annotation)
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

      {#if maxBar}
        <!-- outline tallest bar -->
        <rect
          x={xScale(maxBar.label)}
          y={yScale(maxBar.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(maxBar.value)}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />
        <!-- leader line -->
        <line
          x1={xScale(maxBar.label) + xScale.bandwidth()}
          y1={yScale(maxBar.value) +
            (innerHeight - yScale(maxBar.value)) / 2}
          x2={labelX}
          y2={labelY}
          stroke="currentColor"
          stroke-width="1"
        />
        <!-- annotation label -->
        <text
          x={labelX + 5}
          y={labelY}
          text-anchor="start"
          dominant-baseline="middle"
          class="annotation"
        >
          Year with most projects
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

  .legend em {
    font-style: normal;
    opacity: 0.8;
    margin-left: 0.25rem;
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
    font-size: 0.4rem;
    font-style: italic;
    fill: currentColor;
  }
</style>

