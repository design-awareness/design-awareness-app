<script lang="ts">
  import { pop } from "svelte-spa-router/Router.svelte";
  import BackButton from "../components/BackButton.svelte";
  import ContentFrame from "../components/layout/ContentFrame.svelte";
  import Link from "../components/Link.svelte";
  import Header from "../components/type/Header.svelte";
  import { getProject } from "../data/database";
  import type { Project } from "../data/schema";

  export let params: { id: string };

  let project: Project;
  async function load() {
    project = await getProject(params.id);
  }

  $: [params, load()];

  let active = true;
  async function remove() {
    active = false;
    await project.remove();
    pop();
  }
</script>

<style lang="scss">
</style>

<main class="device-frame page">
  <ContentFrame>
    <BackButton href="/" />
    <Header>{(project && project.name) || 'No project here!'}</Header>
    {#if project}
      <input bind:value={project.name} />
      <p>{project.id}</p>
      <button on:click={remove} disabled={!active}>delete</button>
    {/if}
  </ContentFrame>
</main>