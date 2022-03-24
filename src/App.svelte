<script lang="ts">
  import { initializeApp } from "firebase/app";
  import { getAuth, GoogleAuthProvider, onAuthStateChanged, signInWithPopup, signOut, User } from "firebase/auth";
  import { collection, doc, getDoc, getFirestore, setDoc } from "firebase/firestore/lite";
  import { convertData, mt } from "./Global.svelte";
  import UserMenu from "./UserMenu.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import MembersMenu from "./MembersMenu.svelte";
  import UnverifiedsMenu from "./UnverifiedsMenu.svelte";

  function login() {
    signInWithPopup(getAuth(), new GoogleAuthProvider()).catch(console.error);
  }

  function logout() {
    if (!confirm("Are you sure?")) return;
    signOut(getAuth()).catch(console.error);
  }

  async function loadTeam(id: string) {
    $mt.team.document = doc($mt.team.collection, id);
    $mt.team.data = (await getDoc($mt.team.document)).data();
    $mt.unverified.collection = collection($mt.team.document, "unverifieds").withConverter(convertData.unverified);
    $mt.unverified.document = doc($mt.unverified.collection, $mt.user.data.id);
    $mt.unverified.data = (await getDoc($mt.unverified.document)).data();
    if (!$mt.unverified.data || $mt.user.data.id == $mt.team.data.ownerId) {
      $mt.teamPrivate.document = doc($mt.team.document, "private/data").withConverter(convertData.teamPrivate);
      $mt.teamPrivate.data = (await getDoc($mt.teamPrivate.document)).data();
      $mt.member.collection = collection($mt.team.document, "members").withConverter(convertData.member);
      $mt.member.document = doc($mt.member.collection, $mt.user.data.id);
      $mt.member.data = (await getDoc($mt.member.document)).data();
    }
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user.collection = collection(getFirestore(), "users").withConverter(convertData.user);
      $mt.team.collection = collection(getFirestore(), "teams").withConverter(convertData.team);
      $mt.user.document = doc($mt.user.collection, user.uid);
      let snapshot = await getDoc($mt.user.document);
      if (snapshot.exists()) {
        $mt.user.data = snapshot.data();
        if ($mt.user.data.teamId) {
          await loadTeam($mt.user.data.teamId);
        }
      } else {
        $mt.user.data = {
          id: user.uid,
          teamId: "",
        };
        await setDoc($mt.user.document, $mt.user.data).catch(console.error);
      }
    } else {
      $mt.user = {
        data: null,
        document: null,
        collection: null,
      };
      $mt.team = {
        data: null,
        document: null,
        collection: null,
      };
      $mt.teamPrivate = {
        data: null,
        document: null,
      };
      $mt.member = {
        data: null,
        document: null,
        collection: null,
      };
      $mt.unverified = {
        data: null,
        document: null,
        collection: null,
      };
    }
    $mt.loaded = true;
  }

  initializeApp({
    apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
    authDomain: "meantrack-97d77.firebaseapp.com",
    projectId: "meantrack-97d77",
    storageBucket: "meantrack-97d77.appspot.com",
    messagingSenderId: "267200471704",
    appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
  });
  onAuthStateChanged(getAuth(), loadUser);
</script>

<h1>MeanTrack</h1>
<p>A work-in-progress FRC hour tracking web app</p>

{#if $mt.loaded}
  {#if $mt.user.data}
    <p><button on:click={logout}>Sign out</button></p>
    {#if $mt.team.data}
      {#if $mt.member.data}
        <UserMenu />
      {/if}
      <TeamMenu />
      {#if $mt.teamPrivate.data}
        <MembersMenu />
      {/if}
      {#if $mt.user.data && $mt.user.data.id == $mt.team.data.ownerId}
        <UnverifiedsMenu />
      {/if}
    {:else}
      <TeamlessMenu />
    {/if}
  {:else}
    <p><button on:click={login}>Sign in</button></p>
  {/if}
{:else}
  <p>Loading...</p>
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
