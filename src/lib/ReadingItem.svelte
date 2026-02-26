<script>
  export let data = {};
</script>

<article class="reading-item">
  <img src={data.image} alt={data.title} />
  <div>
    <h3>{data.title}</h3>
    <p>{data.author}</p>
  </div>
</article>

<style>
  .reading-item {
    display: flex;
    align-items: center;
    gap: 0.75rem;
  }

  .reading-item img {
    width: 64px;
    height: 64px;
    object-fit: cover;
    border-radius: 4px;
  }

  .reading-item h3 {
    margin: 0;
    font-size: 1.1rem;
  }

  .reading-item p {
    margin: 0;
    color: #555;
  }
</style>

