<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<nav>
    <a class="current" href=".">Home</a>
    <a href="projects">Projects</a>
    <a href="contact">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/anusrisaraf" target="_blank">GitHub</a>
</nav>

<div class="home-layout">
  <div class="home-main">
    <h1>Sri Saraf</h1>
    <p>Enterprising MIT student fueled by a deep passion for computer science, cybersecurity, and a relentless pursuit of knowledge. Currently pursuing majors in Computer Science & Artificial Intelligence and Business Analytics, while exploring other fields with minors in Mathematics and Music.</p>

    <img src="images/mt.png" alt="fall foliage and the ocean in Acadia National Park">
  </div>

  <aside class="home-reading">
    <h2>What I’m Reading</h2>
    <div class="reading">
      {#each reading as item}
        <ReadingItem data={item} />
      {/each}
    </div>
    <section>
      <h2>Latest projects</h2>
      <div class="projects">
        {#each projects.slice(0, 3) as p}
          <Project data={p} />
        {/each}
      </div>
    </section>

    
  </aside>
</div>