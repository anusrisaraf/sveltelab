<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import Bar from "$lib/Bar.svelte";
  import * as d3 from "d3";

  const projectsForPage = projects.map((p) => ({
    ...p,
    image: "../" + p.image
  }));

  const years = projects.map((p) => p.year);
  const range = Math.max(...years) - Math.min(...years);

  $: barData = d3
    .rollups(projects, (v) => v.length, (d) => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));
</script>

<svelte:head>
  <title>Projects — Sri Saraf</title>
</svelte:head>

<div class="page-inner">
  <h1>{projects.length} Projects over {range} Years</h1>
  <p>Scroll down to see a timeline of my projects and how they have contributed to my growth.</p>

  <Bar data={barData} />

  <ProjectNarrative />

  <p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>

  <div class="projects">
    {#each projectsForPage as p}
      <Project data={p} />
    {/each}
  </div>
</div>