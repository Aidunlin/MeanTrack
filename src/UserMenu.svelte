<script lang="ts">
  import { Timestamp, updateDoc } from "firebase/firestore";
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

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      let currentLog = getCurrentLog();
      let difference = Timestamp.now().toMillis() - currentLog.start.toMillis();
      currentLog.hours = difference / 1000 / 3600;
    } else {
      $mt.member.data.logs = [...$mt.member.data.logs, {
        hours: 0,
        start: Timestamp.now(),
      }];
    }
    $mt.member.data.tracking = !$mt.member.data.tracking;
    updateDoc($mt.member.document, {
      logs: $mt.member.data.logs,
      tracking: $mt.member.data.tracking,
    }).catch(console.error);
  }

  function getTotalHours(): number {
    let totalHours = 0;
    $mt.member.data.logs.forEach((log) => (totalHours += log.hours));
    return totalHours;
  }

  $: {
    if ($mt.member.data) {
      hoursDisplay = getTotalHours().toFixed(1);
      lastTimestamp = getCurrentLog().start;
      newHoursDisplay = (Timestamp.now().toMillis() - lastTimestamp.toMillis()) / 1000 / 3600;
    }
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