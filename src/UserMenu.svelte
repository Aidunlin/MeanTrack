<script lang="ts">
  import {
    Auth,
    GoogleAuthProvider,
    signInWithPopup,
    signOut,
  } from "firebase/auth";
  import { DocumentReference, setDoc, Timestamp } from "firebase/firestore";
  import type { UserData } from "./Global.svelte";

  export let auth: Auth;

  export let userData: UserData;
  export let userDoc: DocumentReference<UserData>;

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - userData.lastAction.toMillis();
    if (userData.tracking) {
      userData.hours += difference / 1000 / 3600;
    }
    userData.tracking = !userData.tracking;
    userData.lastAction = newAction;
    setDoc(userDoc, userData).catch(console.error);
  }

  function logout() {
    if (confirm("Are you sure?")) {
      signOut(auth).catch(console.error);
    }
  }

  function login() {
    signInWithPopup(auth, new GoogleAuthProvider()).catch(console.error);
  }
</script>

{#if userData}
  <h2>{userData.name}</h2>
  <p>Hours: {userData.hours.toFixed(2)}</p>
  {#if userData.lastAction.toMillis()}
    <p>
      {userData.tracking ? "Started:" : "Stopped:"}
      {userData.lastAction.toDate().toLocaleString(undefined, {
        dateStyle: "short",
        timeStyle: "short",
      })}
    </p>
  {/if}
  <p>
    <button on:click={toggleTracking}>
      {userData.tracking ? "Stop" : "Start"} tracking
    </button>
    <button on:click={logout}>Sign out</button>
  </p>
{:else}
  <p><button on:click={login}>Sign in</button></p>
{/if}
