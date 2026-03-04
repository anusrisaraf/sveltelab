<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

  const projectsForPage = projects.map((p) => ({
    ...p,
    image: "../" + p.image
  }));
</script>

<svelte:head>
  <title>Projects — Sri Saraf</title>
</svelte:head>
<div class="page-inner">
  <h1>Projects ({projects.length})</h1>
  <div class="projects">
    {#each projectsForPage as p}
      <Project data={p} />
    {/each}
  </div>
</div>