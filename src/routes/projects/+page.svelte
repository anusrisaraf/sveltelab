<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  const projectsForPage = projects.map((p) => ({
    ...p,
    image: "../" + p.image
  }));

  let years = projects.map((proj) => proj.year);
  let range = Math.max(...years) - Math.min(...years);
</script>

<svelte:head>
  <title>Projects — Sri Saraf</title>
</svelte:head>
<div class="page-inner">
  <h1>{projects.length} Projects over {range} Years</h1>
  <p>Scroll down to see a timeline of my projects and how they have contributed to my growth.</p>

  <ProjectNarrative />

  <p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>

  <div class="projects">
    {#each projectsForPage as p}
      <Project data={p} />
    {/each}
  </div>
</div>