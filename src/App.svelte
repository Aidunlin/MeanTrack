<script lang="ts">
  import { initializeApp } from "firebase/app";
  import { getAuth, GoogleAuthProvider, onAuthStateChanged, signInWithPopup, User } from "firebase/auth";
  import { getFirestore } from "firebase/firestore/lite";
  import { isOwner, ListFS, mt, SingleFS } from "./Global.svelte";
  import UserMenu from "./UserMenu.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import MembersMenu from "./MembersMenu.svelte";
  import UnverifiedsMenu from "./UnverifiedsMenu.svelte";

  let message = "Loading...";

  function logIn() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }

  async function loadTeam() {
    $mt.team = new SingleFS($mt.firestore, "teams", $mt.user.data.teamId);
    if (await $mt.team.getData()) {
      $mt.chosenLogType = $mt.team.data.logTypes[0]?.name ?? "";
      $mt.unverified = new ListFS($mt.team.document, "unverifieds", $mt.user.id);
      await $mt.unverified.getList();
      $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
      if (!(await $mt.member.getList())) $mt.member = $mt.unverified = $mt.team = null;
    } else $mt.team = null;
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user = new SingleFS($mt.firestore, "users", user.uid);
      if (await $mt.user.getData()) {
        if ($mt.user.data.teamId) await loadTeam();
      } else $mt.user = $mt.user.set({ teamId: "" });
    } else $mt.user = $mt.team = $mt.member = $mt.unverified = null;
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
  } else message = "OFFLINE - MeanTrack needs an internet connection";
</script>

<header>
  <h1>MeanTrack</h1>
  <p>A work-in-progress FRC hour tracking web app</p>
</header>

<main>
  {#if $mt.loaded}
    {#if $mt.user?.data}
      {#if $mt.team?.data}
        {#if $mt.member?.data}
          <UserMenu />
        {/if}
        <TeamMenu />
        {#if $mt.member?.list}
          <MembersMenu />
        {/if}
        {#if isOwner($mt)}
          <UnverifiedsMenu />
        {/if}
      {:else}
        <TeamlessMenu />
      {/if}
    {:else}
      <details open>
        <summary>Log in</summary>
        <div class="buttons"><button on:click={logIn}>Log in with Google</button></div>
      </details>
    {/if}
  {:else}
    <p>{message}</p>
  {/if}
</main>

<footer>
  <p>
    Made with <a href="https://firebase.google.com/" target="_blank">Firebase</a>
    and <a href="https://svelte.dev/" target="_blank">Svelte</a>
  </p>
  <p>
    View <a href="https://github.com/Aidunlin/MeanTrack" target="_blank">Repo</a>
    or <a href="https://docs.google.com/document/d/1yPmfHWuSQf4gsOyfTsaVunR9jaGW08NvOWJTsm6861c" target="_blank">Doc</a>
  </p>
</footer>
