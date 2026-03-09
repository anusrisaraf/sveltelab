<script>
  import { onMount } from "svelte";
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";

  let githubData = null;
  let loading = true;
  let error = null;

  onMount(async () => {
    try {
      const response = await fetch("https://api.github.com/users/anusrisaraf");
      if (!response.ok) {
        throw new Error("Failed to load GitHub data");
      }
      githubData = await response.json();
    } catch (err) {
      error = err;
    }
    loading = false;
  });
</script>

<svelte:head>
  <title>Sri Saraf</title>
</svelte:head>

<div class="home-layout">
  <div class="home-main">
    <h1>Sri Saraf</h1>
    <p>Enterprising MIT student fueled by a deep passion for computer science, cybersecurity, and a relentless pursuit of knowledge. Currently pursuing majors in Computer Science & Artificial Intelligence and Business Analytics, while exploring other fields with minors in Mathematics and Music.</p>

    {#if loading}
      <section class="github-stats">
        <h2>My GitHub Stats</h2>
        <p>Loading...</p>
      </section>
    {:else if error}
      <section class="github-stats">
        <h2>My GitHub Stats</h2>
        <p>Something went wrong: {error.message}</p>
      </section>
    {:else}
      <section class="github-stats">
        <h2>My GitHub Stats</h2>
        <dl>
          <dt>Followers</dt>
          <dd>{githubData.followers}</dd>
          <dt>Following</dt>
          <dd>{githubData.following}</dd>
          <dt>Public Repos</dt>
          <dd>{githubData.public_repos}</dd>
          <dt>Public Gists</dt>
          <dd>{githubData.public_gists}</dd>
        </dl>
      </section>
    {/if}

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