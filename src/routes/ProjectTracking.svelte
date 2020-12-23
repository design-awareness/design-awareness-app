<script lang="ts">
  import { onDestroy, beforeUpdate, afterUpdate } from "svelte";
  import Button from "../components/Button.svelte";
  import ContentFrame from "../components/layout/ContentFrame.svelte";
  import Link from "../components/Link.svelte";
  import Modal from "../components/Modal.svelte";
  import TrackingTimer from "../components/TrackingTimer.svelte";
  import { getEntityOrFail, getProject } from "../data/database";
  import { goUpSafe } from "../util/history";
  import useInterval from "../util/interval";

  export let params: { id: string; wild: string };

  const TICK_INTERVAL = 250;

  let thenProject = getProject(params.id);

  let projectTime = 761000; // 12:41, for testing - get from project!

  let activeSubsession = false;
  let pastSessionTime = 0;
  let thisSubsessionStart = 0;
  let subsessionTime = 0;

  function startTracking() {
    activeSubsession = true;
    thisSubsessionStart = Date.now();
    subsessionTime = 0;
  }

  function tick() {
    let tickTime = Date.now();
    subsessionTime = tickTime - thisSubsessionStart;
  }

  function stopTracking() {
    if (activeSubsession) {
      tick();
      pastSessionTime += subsessionTime;
      subsessionTime = 0;
      activeSubsession = false;
    }
  }

  const [enableInterval, destroyInterval] = useInterval(
    { start: startTracking, tick, stop: stopTracking },
    TICK_INTERVAL
  );

  // tracking is enabled when no modal is open
  $: enableInterval(!params.wild);

  // bind this to a Modal's `visible` to enable closing it via tap on scrim
  // an inner Link[up] or closeModal() will also work!
  let bindModalOpen = true;
  let goingUp = false;

  function closeModal() {
    goingUp = true;
    goUpSafe(`/projects/${params.id}/track/`).then(() => {
      bindModalOpen = true;
      goingUp = false;
    });
  }

  $: if (params.wild && !bindModalOpen && !goingUp) closeModal();

  onDestroy(async () => {
    destroyInterval();
    thenProject.then((project) => project && project.save());
    console.log("destroy!");
  });

  function beforeUnload(evt: BeforeUnloadEvent) {
    if (localStorage["dev__suppressBeforeUnload"] !== "1") {
      evt.preventDefault();
      evt.returnValue = "You are currently tracking a project!";
      return "You are currently tracking a project!";
    }
  }
</script>

<style lang="scss">
  @import "src/styles/tokens";
  @import "src/styles/type";

  .device-frame {
    min-height: 100%;
    background-color: $tracking-background-color;
  }

  .top-bar {
    display: flex;
    align-items: center;
    .flex-spacer {
      flex-grow: 1;
    }
  }
</style>

<svelte:window on:beforeunload={beforeUnload} />
<main class="device-frame page">
  <ContentFrame>
    {#await getEntityOrFail(thenProject)}
      Project loading…
    {:then project}
      <div class="top-bar">
        <TrackingTimer
          sessionTime={subsessionTime + pastSessionTime}
          {projectTime} />
        <div class="flex-spacer" />
        <Button small>Add note</Button>
      </div>
      <p>{project.name}</p>

      {#if !params.wild}
        <Link href="/projects/{params.id}/track/x">open modal</Link>
      {:else}
        <Modal bind:visible={bindModalOpen}>
          <Link up href="/projects/{params.id}/track">close modal</Link>
        </Modal>
      {/if}
    {:catch}
      This project does not exist.
      <Link href="/" up>Go home</Link>
    {/await}
  </ContentFrame>
</main>