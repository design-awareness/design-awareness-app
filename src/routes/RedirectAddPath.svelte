<script lang="ts">
  import { replace } from "svelte-spa-router/Router.svelte";

  export let params: any = null;

  // noop to give this prop a use :)
  params;
  console.warn(
    `Fixing bad path ${location.hash}, should be: ${location.hash}/`
  );
  replace(location.hash + "/");
</script>
