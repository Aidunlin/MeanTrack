<script lang="ts">
  import { GoogleAuthProvider, signInWithPopup, signOut } from "firebase/auth";
  import { Timestamp, updateDoc } from "firebase/firestore";
  import { Log, mt } from "./Global.svelte";

  function startTracking() {
    let newLog: Log = {
      hours: 0,
      start: Timestamp.now(),
    };
    $mt.team.member.data.logs = [...$mt.team.member.data.logs, newLog];
  }

  function getCurrentLog() {
    let currentLog: Log = {
      hours: 0,
      start: new Timestamp(0, 0),
    }

    if ($mt.team.member.data.logs.length) {
      $mt.team.member.data.logs.forEach((log) => {
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

  function toggleTracking() {
    if ($mt.team.member.data.tracking) {
      stopTracking();
    } else {
      startTracking();
    }

    $mt.team.member.data.tracking = !$mt.team.member.data.tracking;
    updateDoc($mt.team.member.document, {
      logs: $mt.team.member.data.logs,
      tracking: $mt.team.member.data.tracking,
    }).catch(console.error);
  }

  function getTotalHours(): number {
    let totalHours = 0;
    $mt.team.member.data.logs.forEach(log => {
      totalHours += log.hours;
    });
    return totalHours;
  }

  function logout() {
    if (!confirm("Are you sure?")) return;
    signOut($mt.auth).catch(console.error);
  }

  function login() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }

  let hours: string;
  let lastTimestamp: string;

  $: {
    if ($mt.team.member.data) {
      hours = getTotalHours().toFixed(1);
      lastTimestamp = getCurrentLog().start.toDate().toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" })
    }
  }
</script>

{#if $mt.team.member.data}
  <h2>{$mt.team.member.data.name}</h2>
  {#if $mt.team.member.data.logs}
    <p>Hours: {hours}</p>
    <p>
      {$mt.team.member.data.tracking ? "Started:" : "Stopped:"}
      {lastTimestamp}
    </p>
  {/if}
  <p>
    <button on:click={toggleTracking}>{$mt.team.member.data.tracking ? "Stop" : "Start"} tracking</button>
    |
    <button on:click={logout}>Sign out</button>
  </p>
{:else if $mt.user.data}
  <p><button on:click={logout}>Sign out</button></p>
{:else}
  <p><button on:click={login}>Sign in</button></p>
{/if}
