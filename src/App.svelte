<script lang="ts">
  import { initializeApp } from "firebase/app";
  import {
    getAuth,
    GoogleAuthProvider,
    onAuthStateChanged,
    signInWithPopup,
    signOut,
    User,
  } from "firebase/auth";
  import { getFirestore } from "firebase/firestore/lite";
  import { FSDataSet, mt } from "./Global.svelte";
  import UserMenu from "./UserMenu.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import MembersMenu from "./MembersMenu.svelte";
  import UnverifiedsMenu from "./UnverifiedsMenu.svelte";

  let loadMessage = "Loading...";

  function logInWithGoogle() {
    signInWithPopup($mt.firebase.auth, new GoogleAuthProvider()).catch(console.error);
  }

  function logOut() {
    if (!confirm("Are you sure?")) return;
    signOut($mt.firebase.auth).catch(console.error);
  }

  async function loadTeam() {
    $mt.team = new FSDataSet($mt.firebase.firestore, "teams", $mt.user.data.teamId);
    await $mt.team.refreshData();
    $mt.teamPrivate = new FSDataSet($mt.team.document, "private", "data");
    await $mt.teamPrivate.refreshData();
    $mt.member = new FSDataSet($mt.team.document, "members", $mt.user.document.id);
    await $mt.member.refreshData();
    $mt.unverified = new FSDataSet($mt.team.document, "unverifieds", $mt.user.document.id);
    await $mt.unverified.refreshData();
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user = new FSDataSet($mt.firebase.firestore, "users", user.uid);
      if (await $mt.user.refreshData()) {
        $mt.user.data.teamId && (await loadTeam());
      } else {
        $mt.user.data = {
          teamId: "",
        };
      }
    } else {
      $mt.user = null;
      $mt.team = null;
      $mt.teamPrivate = null;
      $mt.member = null;
      $mt.unverified = null;
    }
    $mt.loaded = true;
  }

  if (navigator.onLine) {
    $mt.firebase.app = initializeApp({
      apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
      authDomain: "meantrack-97d77.firebaseapp.com",
      projectId: "meantrack-97d77",
      storageBucket: "meantrack-97d77.appspot.com",
      messagingSenderId: "267200471704",
      appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
    });
    $mt.firebase.auth = getAuth($mt.firebase.app);
    $mt.firebase.firestore = getFirestore($mt.firebase.app);
    onAuthStateChanged($mt.firebase.auth, loadUser);
  } else {
    loadMessage = "OFFLINE - MeanTrack needs an internet connection";
  }
</script>

<h1>MeanTrack</h1>
<p>A work-in-progress FRC hour tracking web app</p>

{#if $mt.loaded}
  {#if $mt.user?.data}
    <p><button on:click={logOut}>Sign out</button></p>
    {#if $mt.team?.data}
      {#if $mt.member?.data}
        <UserMenu />
      {/if}
      <TeamMenu />
      {#if $mt.teamPrivate?.data}
        <MembersMenu />
      {/if}
      {#if $mt.user.document.id == $mt.team.data.ownerId}
        <UnverifiedsMenu />
      {/if}
    {:else}
      <TeamlessMenu />
    {/if}
  {:else}
    <p><button on:click={logInWithGoogle}>Sign in with Google</button></p>
  {/if}
{:else}
  <p>{loadMessage}</p>
{/if}

<h2>About</h2>
<p>
  View
  <a href="https://github.com/Aidunlin/MeanTrack" target="_blank">Repo</a>
  |
  <a href="https://docs.google.com/document/d/1yPmfHWuSQf4gsOyfTsaVunR9jaGW08NvOWJTsm6861c/" target="_blank">Doc</a>
</p>
<p>
  Made with
  <a href="https://firebase.google.com/" target="_blank">Firebase</a>
  |
  <a href="https://svelte.dev/" target="_blank">Svelte</a>
</p>
