<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { mt } from "./Global.svelte";

  let totalHours: number;
  let newHours: number;
  let trackingDisplay: string;

  const hoursBetween = (a: Timestamp, b: Timestamp) => Math.abs(a.toMillis() - b.toMillis()) / 360000;

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      $mt.member.data.logs.push({
        hours: hoursBetween(Timestamp.now(), $mt.member.data.lastAction),
        start: $mt.member.data.lastAction,
      });
    }
    $mt.member = $mt.member.update({
      lastAction: Timestamp.now(),
      logs: $mt.member.data.logs,
      tracking: !$mt.member.data.tracking,
    });
  }

  function getHours() {
    let hours = 0;
    $mt.member.data.logs.forEach((log) => (hours += log.hours));
    return hours;
  }

  $: {
    totalHours = getHours();
    newHours = hoursBetween(Timestamp.now(), $mt.member.data.lastAction);
    trackingDisplay = $mt.member.data.tracking ? "Stop" : "Start";
  }
</script>

<details open>
  <summary>{$mt.member.data.name}</summary>
  {#if $mt.member.data.logs}
    <p>
      Hours: {totalHours.toFixed(1)}
      {#if $mt.member.data.tracking}
        + {newHours.toFixed(1)}
      {/if}
    </p>
  {/if}
  <p><button on:click={toggleTracking}>{trackingDisplay} tracking</button></p>
</details>
