<script lang="ts">
  import { FirebaseApp, FirebaseOptions, initializeApp } from "firebase/app";
  import { getAuth, GoogleAuthProvider, onAuthStateChanged, signInWithPopup, signOut, User } from "firebase/auth";
  import { collection, doc, Firestore, getDoc, getFirestore, setDoc } from "firebase/firestore";
  import { convertData, mt } from "./Global.svelte";
  import UserMenu from "./UserMenu.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import MembersMenu from "./MembersMenu.svelte";
  import UnverifiedsMenu from "./UnverifiedsMenu.svelte";

  const links = {
    github: "https://github.com/Aidunlin/MeanTrack",
    googleDoc: "https://docs.google.com/document/d/1yPmfHWuSQf4gsOyfTsaVunR9jaGW08NvOWJTsm6861c/edit#",
    firebase: "https://firebase.google.com/",
    svelte: "https://svelte.dev/",
    newcss: "https://newcss.net/",
  };

  const firebaseConfig: FirebaseOptions = {
    apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
    authDomain: "meantrack-97d77.firebaseapp.com",
    projectId: "meantrack-97d77",
    storageBucket: "meantrack-97d77.appspot.com",
    messagingSenderId: "267200471704",
    appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
  };

  let firebaseApp: FirebaseApp;
  let firestore: Firestore;

  function login() {
    signInWithPopup($mt.auth, new GoogleAuthProvider()).catch(console.error);
  }

  function logout() {
    if (!confirm("Are you sure?")) return;
    signOut($mt.auth).catch(console.error);
  }

  async function loadMember() {
    $mt.member.collection = collection($mt.team.document, "members").withConverter(convertData.member);
    $mt.member.document = doc($mt.member.collection, $mt.user.data.id);
    $mt.member.data = (await getDoc($mt.member.document)).data();
  }

  async function loadUnverified() {
    $mt.unverified.collection = collection($mt.team.document, "unverified").withConverter(convertData.unverified);
    $mt.unverified.document = doc($mt.unverified.collection, $mt.user.data.id);
    $mt.unverified.data = (await getDoc($mt.unverified.document)).data();
  }

  async function loadTeam(id: string) {
    $mt.team.document = doc($mt.team.collection, id);
    $mt.team.data = (await getDoc($mt.team.document)).data();
    await loadUnverified();
    if (!$mt.unverified.data || $mt.user.data.id == $mt.team.data.ownerId) {
      $mt.teamPrivate.document = doc($mt.team.document, "private/data").withConverter(convertData.teamPrivate);
      $mt.teamPrivate.data = (await getDoc($mt.teamPrivate.document)).data();
      await loadMember();
    }
  }

  async function loadUser(user: User) {
    $mt.loaded = false;
    if (user) {
      $mt.user.collection = collection(firestore, "users").withConverter(convertData.user);
      $mt.team.collection = collection(firestore, "teams").withConverter(convertData.team);
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

  firebaseApp = initializeApp(firebaseConfig);
  $mt.auth = getAuth(firebaseApp);
  firestore = getFirestore(firebaseApp);
  onAuthStateChanged($mt.auth, loadUser);
</script>

<h1>MeanTrack</h1>
<p>A work-in-progress FRC hour tracking web app</p>

{#if $mt.loaded}
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
  {:else if $mt.user.data}
    <p><button on:click={logout}>Sign out</button></p>
    <TeamlessMenu />
  {:else}
    <p><button on:click={login}>Sign in</button></p>
  {/if}
{:else}
  <p>Loading...</p>
{/if}

<h2>About</h2>
<p>
  View
  <a href={links.github} target="_blank">GitHub Repo</a>
  |
  <a href={links.googleDoc} target="_blank">Google Doc</a>
</p>
<p>
  Powered by
  <a href={links.firebase} target="_blank">Firebase</a>
  |
  <a href={links.svelte} target="_blank">Svelte</a>
  |
  <a href={links.newcss} target="_blank">new.css</a>
</p>
