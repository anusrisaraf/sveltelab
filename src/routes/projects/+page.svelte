<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

  // Adjust image paths for this page so they work from /projects
  const projectsForPage = projects.map((p) => ({
    ...p,
    image: "../" + p.image
  }));
</script>

<nav>
    <a href="..">Home</a>
    <a class="current" href="projects">Projects</a>
    <a href="../contact">Contact</a>
    <a href="../resume">Resume</a>
    <a href="https://github.com/anusrisaraf" target="_blank">GitHub</a>
</nav>
<div class="page-inner">
  <h1>Projects ({projects.length})</h1>
  <div class="projects">
    {#each projectsForPage as p}
      <Project data={p} />
    {/each}
  </div>
</div>