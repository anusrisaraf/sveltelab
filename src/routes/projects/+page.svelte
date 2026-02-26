<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
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
    {#each projects as p}
      <Project data={p} />
    {/each}
  </div>
</div>