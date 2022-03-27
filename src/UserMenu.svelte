<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { Log, mt } from "./Global.svelte";

  let hoursDisplay: string;
  let lastTimestamp: Timestamp;
  let newHoursDisplay: number;

  function getCurrentLog(): Log {
    if ($mt.member.data.logs.length) {
      let currentLog = $mt.member.data.logs[0];
      $mt.member.data.logs.forEach((log) => {
        if (log.start.toMillis() > currentLog.start.toMillis()) {
          currentLog = log;
        }
      });
      return currentLog;
    } else {
      return {
        hours: 0,
        start: Timestamp.now(),
      };
    }
  }

  function getTimestampDifference(a: Timestamp, b: Timestamp) {
    return Math.abs(a.toMillis() - b.toMillis()) / 1000 / 3600;
  }

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      let currentLog = getCurrentLog();
      currentLog.hours = getTimestampDifference(Timestamp.now(), currentLog.start);
    } else {
      $mt.member.data.logs = [
        ...$mt.member.data.logs,
        {
          hours: 0,
          start: Timestamp.now(),
        },
      ];
    }
    $mt.member.updateData({
      tracking: !$mt.member.data.tracking,
      logs: $mt.member.data.logs,
    });
  }

  function getTotalHours(): number {
    let totalHours = 0;
    $mt.member.data.logs.forEach((log) => (totalHours += log.hours));
    return totalHours;
  }

  $: {
    hoursDisplay = getTotalHours().toFixed(1);
    lastTimestamp = getCurrentLog().start;
    newHoursDisplay = getTimestampDifference(Timestamp.now(), lastTimestamp);
  }
</script>

<h2>{$mt.member.data.name}</h2>
{#if $mt.member.data.logs}
  <p>
    Hours:
    {hoursDisplay}
    {#if $mt.member.data.tracking}
      (+ {newHoursDisplay.toFixed(1)})
    {/if}
  </p>
  {#if $mt.member.data.tracking}
    <p>
      Started:
      {lastTimestamp.toDate().toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" })}
    </p>
  {/if}
{/if}
<p>
  <button on:click={toggleTracking}>{$mt.member.data.tracking ? "Stop" : "Start"} tracking</button>
</p>
