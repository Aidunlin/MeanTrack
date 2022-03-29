<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { mt } from "./Global.svelte";

  function getTimestampDifference(a: Timestamp, b: Timestamp) {
    return Math.abs(a.toMillis() - b.toMillis()) / 1000 / 3600;
  }

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      $mt.member.data.logs.push({
        hours: getTimestampDifference(Timestamp.now(), $mt.member.data.lastAction),
        start: $mt.member.data.lastAction,
      });
    }
    $mt.member.updateData({
      lastAction: Timestamp.now(),
      logs: $mt.member.data.logs,
      tracking: !$mt.member.data.tracking,
    });
    $mt.member = $mt.member;
  }

  function displayTotalHours() {
    let totalHours = 0;
    $mt.member.data.logs.forEach((log) => (totalHours += log.hours));
    return totalHours.toFixed(1);
  }

  function displayLastAction() {
    return $mt.member.data.lastAction.toDate().toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" });
  }
</script>

<details open>
  <summary>{$mt.member.data.name}</summary>
  {#if $mt.member.data.logs}
    <p>Hours: {displayTotalHours()}</p>
  {/if}
  {#if $mt.member.data.tracking && $mt.member.data.lastAction}
    <p>Started: {displayLastAction()}</p>
  {/if}
  <p><button on:click={toggleTracking}>{$mt.member.data.tracking ? "Stop" : "Start"} tracking</button></p>  
</details>
