<script lang="ts">
  import { getActivitySet, getAll, newActivitySet } from "../../data/database";
  import add from "@iconify-icons/ic/baseline-add";

  import type { ActivitySet } from "../../data/schema";
  import { sortBy } from "../../util/sort";
  import Button from "../Button.svelte";
  import InvisibleButton from "../InvisibleButton.svelte";
  import Item from "./Item.svelte";

  export let activitySet: ActivitySet;
  export let label = "Activity Set";
  export let createNew: () => void = null;
  export let withEmpty = false;

  export let allActivitySets: Promise<ActivitySet[]>;
  function refreshSets() {
    allActivitySets = getAll("ActivitySet").then((ids) =>
      Promise.all(ids.map(getActivitySet))
    );
  }
  refreshSets();

  function set(as: ActivitySet) {
    activitySet = as;
  }
</script>

<div class="label">
  <span>{label}</span>
  {#if createNew}
    <Button inlabel on:click={createNew} icon={add}>Create new</Button>
  {/if}
</div>
{#await allActivitySets}
  <div>Loading activity sets…</div>
{:then activitySets}
  <ul class="container">
    {#if withEmpty}
      <li>
        <InvisibleButton on:click={() => set(newActivitySet())}>
          <Item empty />
        </InvisibleButton>
      </li>
    {/if}
    {#each sortBy('wellKnown', sortBy('name', activitySets)) as activitySetItem}
      <li>
        <InvisibleButton on:click={() => set(activitySetItem)}>
          <Item
            activitySet={activitySetItem}
            selected={activitySetItem === activitySet}
          />
        </InvisibleButton>
      </li>
    {/each}
  </ul>
{/await}

<style lang="scss">
  @import "src/styles/tokens";
  @import "src/styles/type";
  .label {
    @include type-style($type-input-label);
    margin: $block-vertical-spacing 0 $input-spacing-inner 0;
    display: flex;
    justify-content: space-between;
  }
  .container {
    border: $input-border-size solid $input-border-color;
    border-radius: $input-border-radius;
    background-color: $input-background-color;
    list-style-type: none;
    margin: 0;
    padding: 0;
    overflow: none;
  }
  li {
    padding: 0;
    border-bottom: $input-border-size solid $input-border-color;
    &:last-child {
      border-bottom: none;
    }
  }
</style>
