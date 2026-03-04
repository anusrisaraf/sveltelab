<script>
  import { base } from "$app/paths";
  import { page } from "$app/stores";

  let pages = [
    { url: "/", title: "Home" },
    { url: "/projects", title: "Projects" },
    { url: "/contact", title: "Contact" },
    { url: "/resume", title: "Resume" },
    { url: "https://github.com/anusrisaraf", title: "GitHub" }
  ];

  const storage = globalThis.localStorage ?? {};
  let colorScheme = storage.colorScheme ?? "light dark";

  let root = globalThis.document?.documentElement;

  $: root?.style.setProperty("color-scheme", colorScheme);
  $: storage.colorScheme = colorScheme;
</script>

<div class="header-row">
  <nav>
  {#each pages as p}
    <a
      href={p.url.startsWith("http") ? p.url : base + p.url}
      class:current={!p.url.startsWith("http") &&
        (p.url === "/"
          ? $page.url.pathname === base + "/"
          : $page.url.pathname.startsWith(base + p.url))}
      target={p.url.startsWith("http") ? "_blank" : null}
      rel={p.url.startsWith("http") ? "noreferrer" : null}
    >
      {p.title}
    </a>
  {/each}
  </nav>
  <label class="color-scheme-switch">
    Theme:
    <select bind:value={colorScheme}>
      <option value="light dark">Automatic</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  </label>
</div>

<slot />

<style>
  .header-row {
    position: relative;
  }

  .header-row nav {
    --border-color: oklch(50% 10% 200 / 40%);
    display: flex;
    margin-bottom: 0.5em;
    padding-bottom: 0.5em;
    padding-right: 10rem; /* reserve space for theme switcher */
    border-bottom: 2px solid var(--border-color);
  }

  nav a {
    flex: 1;
    text-decoration: none;
    color: inherit;
    text-align: center;
    padding: 0.5em;
  }

  nav a.current {
    color: var(--color-accent);
    border-bottom: 0.4em solid var(--color-accent);
    padding-bottom: 0.1em;
  }

  nav a:hover {
    border-bottom: 0.4em solid var(--color-accent);
    padding-bottom: 0.1em;
  }

  .color-scheme-switch {
    position: absolute;
    top: 0.5rem;
    right: 0;
    display: inline-flex;
    gap: 0.25rem;
    align-items: center;
    font-size: 80%;
  }

  .color-scheme-switch select {
    font: inherit;
  }
</style>
