<script>
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
  <img src={data.image} alt="" />
  <p>{data.description}</p>
</article>

<style>
  article {
    grid-row: span 3;
    display: grid;
    grid-template-rows: subgrid;
  }

  h2 {
    margin: 0;
  }
</style>

