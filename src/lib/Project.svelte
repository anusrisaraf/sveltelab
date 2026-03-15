<script>
  import { base } from "$app/paths";
  export let data = {};
</script>

<article>
  <h2>
    {#if data.url}
      <a href={data.url}>{data.title}</a>
    {:else}
      {data.title}
    {/if}
  </h2>
  {#if data.year}
    <p class="project-year">{data.year}</p>
  {/if}
  <img src={base + "/" + data.image} alt="" />
  <p>{data.description}</p>
</article>

<style>
  article {
    grid-row: span 4;
    display: grid;
    grid-template-rows: subgrid;
  }

  h2 {
    margin: 0;
  }

  .project-year {
    margin: 0.15rem 0 0.35rem;
    font-size: 0.9rem;
    color: oklch(60% 0.05 250);
  }
</style>

