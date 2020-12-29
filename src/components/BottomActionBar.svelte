<script type="ts">
  import { createEventDispatcher } from "svelte";

  import Button from "./Button.svelte";

  export let disabled = false;
  export let label: string;

  export let label2: string = null;

  const dispatch = createEventDispatcher();
  const click = (button) => (evt) => {
    if (label2) {
      dispatch("click", {
        button,
        event: evt,
      });
    } else {
      dispatch("click", evt);
    }
  };
</script>

<style type="scss">
  @import "src/styles/tokens";
  .bar {
    position: fixed;
    left: 0;
    bottom: 0;
    height: $action-bar-height;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: $action-bar-shadow;
    background-color: $action-bar-background-color;
    gap: 1rem;
    &.disabled {
      background-color: $action-bar-background-color_disabled;
    }
  }
  .spacer {
    height: $action-bar-height;
  }
</style>

<div class="spacer" role="presentation" />
<div class="bar" class:disabled>
  <Button on:click={click(1)} {disabled}>{label}</Button>
  {#if label2}
    <Button on:click={click(2)} {disabled}>{label2}</Button>
  {/if}
</div>
