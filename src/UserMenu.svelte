<script lang="ts">
  import { signOut } from "firebase/auth";
  import { Timestamp, updateDoc } from "firebase/firestore";
  import { Log, mt } from "./Global.svelte";

  let hours: string;
  let lastTimestamp: Timestamp;
  let newHours: number;

  function getCurrentLog() {
    let currentLog: Log = {
      hours: 0,
      start: new Timestamp(0, 0),
    };
    if ($mt.member.data.logs.length) {
      $mt.member.data.logs.forEach((log) => {
        if (log.start.toMillis() > currentLog.start.toMillis()) {
          currentLog = log;
        }
      });
    }
    return currentLog;
  }

  function stopTracking() {
    let currentLog = getCurrentLog();
    let difference = Timestamp.now().toMillis() - currentLog.start.toMillis();
    currentLog.hours = difference / 1000 / 3600;
  }

  function startTracking() {
    let newLog: Log = {
      hours: 0,
      start: Timestamp.now(),
    };
    $mt.member.data.logs = [...$mt.member.data.logs, newLog];
  }

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      stopTracking();
    } else {
      startTracking();
    }
    $mt.member.data.tracking = !$mt.member.data.tracking;
    updateDoc($mt.member.document, {
      logs: $mt.member.data.logs,
      tracking: $mt.member.data.tracking,
    }).catch(console.error);
  }

  function getTotalHours(): number {
    let totalHours = 0;
    $mt.member.data.logs.forEach((log) => {
      totalHours += log.hours;
    });
    return totalHours;
  }

  function logout() {
    if (!confirm("Are you sure?")) return;
    signOut($mt.auth).catch(console.error);
  }

  $: {
    if ($mt.member.data) {
      hours = getTotalHours().toFixed(1);
      lastTimestamp = getCurrentLog().start;
      newHours = (Timestamp.now().toMillis() - lastTimestamp.toMillis()) / 1000 / 3600;
    }
  }
</script>

<h2>{$mt.member.data.name}</h2>
{#if $mt.member.data.logs}
  <p>
    Hours:
    {hours}
    {#if $mt.member.data.tracking}
      (+ {newHours.toFixed(1)})
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
  |
  <button on:click={logout}>Sign out</button>
</p>
