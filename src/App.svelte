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
  import { ListFS, mt, SingleFS } from "./Global.svelte";
  import UserMenu from "./UserMenu.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import MembersMenu from "./MembersMenu.svelte";
  import UnverifiedsMenu from "./UnverifiedsMenu.svelte";

  let message = "Loading...";

  const logIn = () => signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  const logInAnon = () => signInAnonymously($mt.auth).catch(console.error);
  const logOut = () => signOut($mt.auth).catch(console.error);

  async function loadTeam() {
    $mt.team = new SingleFS($mt.firestore, "teams", $mt.user.data.teamId);
    if (await $mt.team.getData()) {
      $mt.unverified = new ListFS($mt.team.document, "unverifieds", $mt.user.id);
      await $mt.unverified.getList();
      $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
      if (!(await $mt.member.getList())) {
        $mt.member = null;
        $mt.unverified = null;
        $mt.team = null;
      }
    } else {
      $mt.team = null;
    }
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user = new SingleFS($mt.firestore, "users", user.uid);
      if (await $mt.user.getData()) {
        if ($mt.user.data.teamId) await loadTeam();
      } else {
        $mt.user = $mt.user.set({ teamId: "" });
      }
    } else {
      $mt.user = null;
      $mt.team = null;
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

<main>
  {#if $mt.loaded}
    {#if $mt.user?.data}
      <p><button on:click={logOut}>Log out</button></p>
      {#if $mt.team?.data}
        {#if $mt.member?.data}
          <UserMenu />
        {/if}
        <TeamMenu />
        {#if $mt.member?.data}
          <MembersMenu />
        {/if}
        {#if $mt.user.id == $mt.team.data.ownerId}
          <UnverifiedsMenu />
        {/if}
      {:else}
        <TeamlessMenu />
      {/if}
    {:else}
      <p><button on:click={logIn}>Log in with Google</button></p>
      {#if location.hostname.includes("localhost")}
        <p><button on:click={logInAnon}>Log in anonymously</button></p>
      {/if}
    {/if}
  {:else}
    <p>{message}</p>
  {/if}
</main>

<footer>
  <p>
    View <a href="https://github.com/Aidunlin/MeanTrack" target="_blank">Repo</a>
    | <a href="https://docs.google.com/document/d/1yPmfHWuSQf4gsOyfTsaVunR9jaGW08NvOWJTsm6861c" target="_blank">Doc</a>
  </p>
  <p>
    Made with <a href="https://firebase.google.com/" target="_blank">Firebase</a>
    | <a href="https://svelte.dev/" target="_blank">Svelte</a>
    | <a href="https://newcss.net/" target="_blank">new.css</a>
  </p>
</footer>
