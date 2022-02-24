<script lang="ts">
  import { GoogleAuthProvider, signInWithPopup, signOut } from "firebase/auth";
  import { Timestamp, updateDoc } from "firebase/firestore";
  import { mt } from "./Global.svelte";

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - $mt.userData.lastAction.toMillis();
    if ($mt.userData.tracking) {
      $mt.userData.hours += difference / 1000 / 3600;
    }
    $mt.userData.tracking = !$mt.userData.tracking;
    $mt.userData.lastAction = newAction;
    updateDoc($mt.userDocument, {
      hours: $mt.userData.hours,
      tracking: $mt.userData.tracking,
      lastAction: $mt.userData.lastAction,
    }).catch(console.error);
  }

  function logout() {
    if (confirm("Are you sure?")) {
      signOut($mt.auth).catch(console.error);
    }
  }

  function login() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }
</script>

{#if $mt.userData}
  <h2>{$mt.userData.name}</h2>
  <p>Hours: {$mt.userData.hours.toFixed(2)}</p>
  {#if $mt.userData.lastAction.toMillis()}
    <p>
      {$mt.userData.tracking ? "Started:" : "Stopped:"}
      {$mt.userData.lastAction.toDate().toLocaleString(undefined, {
        dateStyle: "short",
        timeStyle: "short",
      })}
    </p>
  {/if}
  <p>
    <button on:click={toggleTracking}>
      {$mt.userData.tracking ? "Stop" : "Start"} tracking
    </button>
    <button on:click={logout}>Sign out</button>
  </p>
{:else}
  <p><button on:click={login}>Sign in</button></p>
{/if}
