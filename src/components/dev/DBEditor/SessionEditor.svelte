<script lang="ts">
  import { onTrigger } from "../../../util/trigger";
  import type { ITriggerPassable, TriggerInvoker } from "../../../util/trigger";
  import { getSession, newSession } from "../../../data/database";
  import type { DBModelName, Session } from "../../../data/schema";
  import EditorField from "./EditorField.svelte";
  import EditorRefField from "./EditorRefField.svelte";

  export let setView: TriggerInvoker<[DBModelName, string]>;
  export let save: ITriggerPassable<void>;
  export let remove: ITriggerPassable<void>;
  export let id: string;

  let dbObj: Session;
  let loadedId: string = undefined;
  let loading = true;

  async function loadObj() {
    loading = true;
    if (id === null) {
      dbObj = newSession();
    } else {
      dbObj = await getSession(id);
    }
    loading = false;
  }
  $: if (id !== loadedId) loadObj();

  onTrigger(save, async () => {
    await dbObj.save();
    setView(["Session", dbObj.id]);
  });
  onTrigger(remove, async () => {
    await dbObj.remove();
    setView(["Session", null]);
  });
</script>

<style lang="scss">
  .editor {
    display: table;
  }
  p {
    font-weight: 600;
    line-height: 2;
  }
  .data-view {
    max-width: calc(100vw - 2rem);
  }
  pre {
    white-space: pre-wrap;
  }
</style>

<p class="detail">{(dbObj && dbObj.id) || 'New Session'}</p>
<div class="editor">
  {#if !loading && dbObj}
    <EditorField name="label" type="string" bind:value={dbObj.label} />
    <EditorField name="startTime" type="date" bind:value={dbObj.startTime} />
    <EditorField name="duration" type="number" bind:value={dbObj.duration} />
    <EditorRefField
      name="project"
      type="Project"
      bind:value={dbObj.project}
      {setView} />
  {/if}
</div>
{#if !loading && dbObj}
  <div class="data-view">
    <pre>{'data:\n' + dbObj.data
          .map(
            (pairs, i) =>
              '[' + i + '] ' + pairs.map(([a, b]) => a + ':' + b).join(', ')
          )
          .join('\n')}</pre>
  </div>
{/if}
