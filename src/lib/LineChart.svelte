<script>
  import * as d3 from "d3";

  export let data = [];

  const width = 1000;
  const height = 300;
  const margin = { top: 36, right: 28, bottom: 48, left: 56 };

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
    width: width - margin.left - margin.right,
    height: height - margin.top - margin.bottom
  };

  $: minDate = data.length ? d3.min(data, (d) => d.date) : new Date();
  $: maxDate = data.length
    ? d3.timeDay.offset(d3.max(data, (d) => d.date), 1)
    : new Date();

  $: xScale = d3
    .scaleTime()
    .domain([minDate, maxDate])
    .range([usableArea.left, usableArea.right]);

  $: yScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.count) || 1])
    .nice()
    .range([usableArea.bottom, usableArea.top]);

  $: line = d3
    .line()
    .x((d) => xScale(d.date))
    .y((d) => yScale(d.count))
    .curve(d3.curveBumpX);

  let xAxis;
  let yAxis;

  let hoveredDay = null; // e.g. "Monday"

  $: chartTitle = hoveredDay ? `Lines edited on ${hoveredDay}` : "Lines edited by day";

  $: dayRegions = (() => {
    if (!data.length) return [];
    return data.map((d, i, arr) => {
      const prev = arr[i - 1];
      const next = arr[i + 1];
      const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
      const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;
      return {
        date: d.date,
        weekday: d.date.toLocaleString("en", { weekday: "long" }),
        x: xScale(left),
        width: xScale(right) - xScale(left)
      };
    });
  })();

  $: {
    if (xAxis) d3.select(xAxis).call(d3.axisBottom(xScale));
    if (yAxis) d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="wrap">
  <h3 class="title">{chartTitle}</h3>
  <svg
    class="svg"
    viewBox="0 0 {width} {height}"
    role="img"
    aria-label="Lines edited by day"
    on:mouseleave={() => (hoveredDay = null)}
  >
    <g transform="translate(0,{usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left},0)" bind:this={yAxis} />

    <!-- invisible hover regions -->
    {#each dayRegions as region}
      <rect
        role="button"
        tabindex="0"
        aria-label="Highlight {region.weekday}"
        x={region.x}
        y={usableArea.top}
        width={region.width}
        height={usableArea.bottom - usableArea.top}
        fill="transparent"
        on:mouseenter={() => (hoveredDay = region.weekday)}
        on:keydown={(e) => {
          if (e.key === "Enter" || e.key === " ") {
            e.preventDefault();
            hoveredDay = region.weekday;
          }
        }}
      />
    {/each}

    <!-- highlight bands -->
    {#if hoveredDay}
      {#each dayRegions as region}
        {#if region.weekday === hoveredDay}
          <rect
            x={region.x}
            y={usableArea.top}
            width={region.width}
            height={usableArea.bottom - usableArea.top}
            fill="var(--color-accent, #c45c26)"
            opacity="0.18"
          />
        {/if}
      {/each}
    {/if}

    <path d={line(data)} fill="none" stroke="steelblue" stroke-width="2" />

    {#each data as d}
      {@const isHighlighted =
        hoveredDay && d.date.toLocaleString("en", { weekday: "long" }) === hoveredDay}

      <circle
        cx={xScale(d.date)}
        cy={yScale(d.count)}
        r={isHighlighted ? 4 : 3}
        fill={isHighlighted ? "var(--color-accent, #c45c26)" : "steelblue"}
      />

      {#if isHighlighted}
        <text
          x={xScale(d.date)}
          y={usableArea.top + 14}
          text-anchor="middle"
          font-size="12"
          fill="var(--color-accent, #c45c26)"
        >
          {Math.round(d.count)}
        </text>
      {/if}
    {/each}

    <!-- x-axis label -->
    <text
      x={usableArea.left + usableArea.width / 2}
      y={height - 6}
      text-anchor="middle"
      class="axis-label"
    >
      Date
    </text>

    <!-- y-axis label -->
    <text
      x={-(usableArea.top + usableArea.height / 2)}
      y={12}
      text-anchor="middle"
      transform="rotate(-90)"
      class="axis-label"
    >
      Lines edited
    </text>
  </svg>
</div>

<style>
  .wrap {
    max-width: 100%;
    margin: 0.5rem auto 0;
  }

  .title {
    margin: 0.25rem 0 0.4rem;
    text-align: center;
    font-size: 1rem;
  }

  .svg {
    display: block;
    width: 100%;
    height: auto;
    max-height: min(40vh, 320px);
    overflow: visible;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
    opacity: 0.85;
  }
</style>

