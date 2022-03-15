<script lang="ts">
  import { GoogleAuthProvider, signInWithPopup, signOut } from "firebase/auth";
  import { Timestamp, updateDoc } from "firebase/firestore";
  import { mt } from "./Global.svelte";

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - $mt.team.member.data.lastAction.toMillis();
    if ($mt.team.member.data.tracking) {
      $mt.team.member.data.hours += difference / 1000 / 3600;
    }
    $mt.team.member.data.tracking = !$mt.team.member.data.tracking;
    $mt.team.member.data.lastAction = newAction;
    updateDoc($mt.team.member.document, {
      hours: $mt.team.member.data.hours,
      tracking: $mt.team.member.data.tracking,
      lastAction: $mt.team.member.data.lastAction,
    }).catch(console.error);
  }

  function editHours() {
    $mt.team.member.data.hours = parseInt(prompt("Enter your hours:"));
    updateDoc($mt.team.member.document, {
      hours: $mt.team.member.data.hours,
    }).catch(console.error);
  }

  function logout() {
    if (!confirm("Are you sure?")) return;
    signOut($mt.auth).catch(console.error);
  }

  function login() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }
</script>

{#if $mt.team.member.data}
  <h2>{$mt.team.member.data.name}</h2>
  <p>Hours: {$mt.team.member.data.hours.toFixed(1)}</p>
  <p>
    {$mt.team.member.data.tracking ? "Started:" : "Stopped:"}
    {$mt.team.member.data.lastAction.toDate().toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" })}
  </p>
  <p>
    <button on:click={toggleTracking}>{$mt.team.member.data.tracking ? "Stop" : "Start"} tracking</button>
    <button on:click={editHours}>Edit hours</button>
    |
    <button on:click={logout}>Sign out</button>
  </p>
{:else if $mt.user.data}
  <p><button on:click={logout}>Sign out</button></p>
{:else}
  <p><button on:click={login}>Sign in</button></p>
{/if}
