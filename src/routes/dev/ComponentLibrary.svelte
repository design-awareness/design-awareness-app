<script lang="ts">
  import BackButton from "../../components/BackButton.svelte";
  import ContentFrame from "../../components/layout/ContentFrame.svelte";
  import Link from "../../components/Link.svelte";
  import Header from "../../components/type/Header.svelte";
  import ActivityToggleDemo from "./ComponentLibrary/ActivityToggleDemo.svelte";
  import ModalDemo from "./ComponentLibrary/ModalDemo.svelte";
  import PopupMenuDemo from "./ComponentLibrary/PopupMenuDemo.svelte";

  export let params: { wild: string };

  let components = {
    "activity-toggle": ActivityToggleDemo,
    "popup-menu": PopupMenuDemo,
    modal: ModalDemo,
  };
</script>

{#if !params.wild}
  <ContentFrame>
    <BackButton href="/dev/" />
    <Header>Component library</Header>
    <ul>
      {#each Object.entries(components) as [slug, component]}
        <li>
          <Link href="/dev/component-library/{slug}">
            {component.name.replace('Demo', '')}
          </Link>
        </li>
      {/each}
    </ul>
  </ContentFrame>
{:else}
  <svelte:component this={components[params.wild.replace('/', '')]} />
{/if}
