<script>
  import * as d3 from "d3";
  import { computePosition, autoPlacement, offset } from "@floating-ui/dom";

  /** @type {Array<Record<string, unknown>>} */
  export let commits = [];

  /** Selected commit objects (same references as in `commits`) */
  export let clickedCommits = [];
  export let brushedCommits = [];
  export let selectedCommits = [];

  const width = 900;
  const height = 380;

  const margin = { top: 36, right: 28, bottom: 48, left: 56 };

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
    width: width - margin.left - margin.right,
    height: height - margin.top - margin.bottom
  };

  /** Largest dots drawn first so smaller dots stay on top */
  $: sortedCommits = d3.sort(commits, (d) => -d.totalLines);

  $: minDate =
    commits.length > 0 ? d3.min(commits, (d) => d.datetime) : new Date();
  $: maxDate =
    commits.length > 0
      ? d3.timeDay.offset(d3.max(commits, (d) => d.datetime), 1)
      : new Date();

  $: xScale = d3
    .scaleTime()
    .domain([minDate, maxDate])
    .range([usableArea.left, usableArea.right])
    .nice();

  $: yScale = d3
    .scaleLinear()
    .domain([0, 24])
    .range([usableArea.bottom, usableArea.top]);

  $: lineExtent = d3.extent(commits, (d) => d.totalLines);
  $: rScale = d3
    .scaleSqrt()
    .domain(lineExtent[0] === undefined ? [0, 1] : lineExtent)
    .range([5, 30]);

  let xAxis;
  let yAxis;
  let yAxisGridlines;

  let svgEl;
  let brushSelection = null;

  let hoveredIndex = -1;
  $: hoveredCommit = sortedCommits[hoveredIndex] ?? {};

  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };

  $: {
    selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));

    if (xAxis) {
      d3.select(xAxis).call(d3.axisBottom(xScale));
    }
    if (yAxis) {
      d3.select(yAxis).call(
        d3
          .axisLeft(yScale)
          .tickFormat((d) => String(d % 24).padStart(2, "0") + ":00")
      );
    }
    if (yAxisGridlines) {
      d3.select(yAxisGridlines).call(
        d3
          .axisLeft(yScale)
          .tickFormat("")
          .tickSize(-usableArea.width)
      );
    }
  }

  function isCommitBrushed(commit) {
    if (!brushSelection) return false;
    const [[x0, y0], [x1, y1]] = brushSelection;
    const cx = xScale(commit.datetime);
    const cy = yScale(commit.hourFrac);
    return x0 <= cx && cx <= x1 && y0 <= cy && cy <= y1;
  }

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  $: brushedCommits = brushSelection ? sortedCommits.filter(isCommitBrushed) : [];

  $: {
    if (svgEl) {
      d3.select(svgEl).call(
        d3
          .brush()
          .extent([
            [usableArea.left, usableArea.top],
            [usableArea.right, usableArea.bottom]
          ])
          .on("start brush end", brushed)
      );

      // Keep dots/tooltips interactive above the brush overlay.
      d3.select(svgEl).selectAll(".dots, .overlay ~ *").raise();
    }
  }

  function toggleCommitAtIndex(index) {
    const commit = sortedCommits[index];
    if (!clickedCommits.includes(commit)) {
      clickedCommits = [...clickedCommits, commit];
    } else {
      clickedCommits = clickedCommits.filter((c) => c !== commit);
    }
  }

  async function dotInteraction(index, evt) {
    const hoveredDot = evt.target;
    if (evt.type === "mouseenter") {
      hoveredIndex = index;
      if (commitTooltip) {
        tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
          strategy: "fixed",
          middleware: [offset(5), autoPlacement()]
        });
      }
    } else if (evt.type === "mouseleave") {
      hoveredIndex = -1;
    } else if (evt.type === "click") {
      toggleCommitAtIndex(index);
    }
  }
</script>

<div class="scatter-wrap">
  <svg
    class="scatter-svg"
    viewBox="0 0 {width} {height}"
    role="img"
    aria-label="Commits by date and time of day"
    bind:this={svgEl}
  >
    <g
      class="gridlines"
      transform="translate({usableArea.left}, 0)"
      bind:this={yAxisGridlines}
    />
    <g
      transform="translate(0,{usableArea.bottom})"
      bind:this={xAxis}
    />
    <g transform="translate({usableArea.left},0)" bind:this={yAxis} />

    <g class="dots">
      {#each sortedCommits as commit, index}
        <circle
          role="button"
          tabindex="0"
          aria-pressed={selectedCommits.includes(commit)}
          aria-label="Commit {commit.id}, press Enter or Space to toggle selection"
          cx={xScale(commit.datetime)}
          cy={yScale(commit.hourFrac)}
          r={rScale(commit.totalLines)}
          class:selected={selectedCommits.includes(commit)}
          on:mouseenter={(e) => dotInteraction(index, e)}
          on:mouseleave={(e) => dotInteraction(index, e)}
          on:click={(e) => dotInteraction(index, e)}
          on:keydown={(e) => {
            if (e.key === "Enter" || e.key === " ") {
              e.preventDefault();
              toggleCommitAtIndex(index);
            }
          }}
        />
      {/each}
    </g>
  </svg>

  <dl
    class="info tooltip"
    bind:this={commitTooltip}
    hidden={hoveredIndex === -1}
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px;"
  >
    <dt>Commit</dt>
    <dd>
      <a href={hoveredCommit.url} target="_blank" rel="noreferrer"
        >{hoveredCommit.id}</a
      >
    </dd>
    <dt>Date</dt>
    <dd>
      {hoveredCommit.datetime?.toLocaleString?.("en", {
        dateStyle: "full"
      })}
    </dd>
    <dt>Time</dt>
    <dd>
      {hoveredCommit.datetime?.toLocaleTimeString?.("en", {
        timeStyle: "short"
      })}
    </dd>
    <dt>Author</dt>
    <dd>{hoveredCommit.author}</dd>
    <dt>Lines edited</dt>
    <dd>{hoveredCommit.totalLines}</dd>
  </dl>
</div>

<style>
  .scatter-wrap {
    position: relative;
    max-width: 100%;
    margin: 0 auto 1rem;
  }

  .scatter-svg {
    display: block;
    width: 100%;
    height: auto;
    max-height: min(55vh, 420px);
    overflow: visible;
  }

  .gridlines {
    stroke-opacity: 0.2;
  }

  @keyframes marching-ants {
    to {
      stroke-dashoffset: -8; /* 5 + 3 */
    }
  }

  svg :global(.selection) {
    fill: var(--color-accent, #c45c26);
    fill-opacity: 0.1;
    stroke: currentColor;
    stroke-opacity: 0.7;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }

  circle {
    fill: steelblue;
    fill-opacity: 0.65;
    transition: fill 200ms, fill-opacity 200ms;
    cursor: pointer;
  }

  circle:hover:not(.selected) {
    fill: darkgreen;
  }

  circle.selected {
    fill: var(--color-accent, #c45c26);
    fill-opacity: 0.95;
  }

  circle.selected:hover {
    fill: #fbbf24;
    fill-opacity: 1;
  }

  dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.25rem 1rem;
    margin: 0;
    font-size: 0.85rem;
  }

  dl.info dt {
    margin: 0;
    font-weight: 600;
    opacity: 0.75;
  }

  dl.info dd {
    margin: 0;
  }

  .tooltip {
    position: fixed;
    z-index: 20;
    pointer-events: none;
    padding: 0.65rem 0.85rem;
    border-radius: 8px;
    background-color: oklch(100% 0 0 / 88%);
    color: var(--color-text, #1a1a1a);
    box-shadow: 0 4px 18px oklch(0% 0 0 / 18%);
    backdrop-filter: blur(6px);
    transition-duration: 400ms;
    transition-property: opacity, visibility;
  }

  /* Apply dark tooltip only when the site theme is explicitly set to dark */
  :global(html[style*="color-scheme: dark"]) .tooltip {
    background-color: oklch(14% 0.02 255 / 90%);
    color: oklch(96% 0 0);
    box-shadow:
      0 10px 28px oklch(0% 0 0 / 55%),
      0 0 0 1px oklch(100% 0 0 / 12%),
      0 1px 0 oklch(100% 0 0 / 16%) inset;
  }

  dl.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
  }
</style>
