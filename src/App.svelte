<script lang="ts">
  import { initializeApp } from "firebase/app";
  import {
    getAuth,
    GoogleAuthProvider,
    onAuthStateChanged,
    signInAnonymously,
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

  let message = "Loading...";

  function logIn() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }

  function logInAnon() {
    signInAnonymously($mt.auth).catch(console.error);
  }

  function logOut() {
    confirm("Are you sure?") && signOut($mt.auth).catch(console.error);
  }

  async function loadTeam() {
    $mt.team = new FSDataSet($mt.firestore, "teams", $mt.user.data.teamId);
    $mt.teamPrivate = new FSDataSet($mt.team.document, "private", "data");
    $mt.member = new FSDataSet($mt.team.document, "members", $mt.user.document.id);
    $mt.unverified = new FSDataSet($mt.team.document, "unverifieds", $mt.user.document.id);
    await $mt.team.refreshData();
    if (!(await $mt.unverified.refreshData())) {
      await Promise.all([$mt.teamPrivate.refreshData(), $mt.member.refreshData()]).catch(console.error);
    }
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user = new FSDataSet($mt.firestore, "users", user.uid);
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
    initializeApp({
      apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
      authDomain: "meantrack-97d77.firebaseapp.com",
      projectId: "meantrack-97d77",
      storageBucket: "meantrack-97d77.appspot.com",
      messagingSenderId: "267200471704",
      appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
    });
    $mt.auth = getAuth();
    $mt.firestore = getFirestore();
    onAuthStateChanged($mt.auth, loadUser);
  } else {
    message = "OFFLINE - MeanTrack needs an internet connection";
  }
</script>

<header>
  <h1>MeanTrack</h1>
  <p>A work-in-progress FRC hour tracking web app</p>
</header>

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
    <p><button on:click={logIn}>Sign in with Google</button></p>
    {#if location.hostname.includes("localhost")}
      <p><button on:click={logInAnon}>Sign in anonymously</button></p>
    {/if}
  {/if}
{:else}
  <p>{message}</p>
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
  |
  <a href="https://newcss.net/" target="_blank">new.css</a>
</p>
