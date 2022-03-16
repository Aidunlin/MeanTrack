<script lang="ts">
  import { FirebaseApp, FirebaseOptions, initializeApp } from "firebase/app";
  import { getAuth, onAuthStateChanged, User } from "firebase/auth";
  import { collection, doc, Firestore, getDoc, getFirestore, setDoc } from "firebase/firestore";
  import {
    convertMemberData,
    convertTeamData,
    convertTeamPrivateData,
    convertUnverifiedData,
    convertUserData,
    mt,
  } from "./Global.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import UserMenu from "./UserMenu.svelte";

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

  let loaded = false;

  async function loadTeam(id: string) {
    $mt.team.document = doc($mt.team.collection, id);
    $mt.team.data = (await getDoc($mt.team.document)).data();

    $mt.team.unverified.collection = collection($mt.team.document, "unverified").withConverter(convertUnverifiedData);
    $mt.team.unverified.document = doc($mt.team.unverified.collection, "unverified").withConverter(
      convertUnverifiedData
    );
    $mt.team.unverified.data = (await getDoc($mt.team.unverified.document)).data();

    if (!$mt.team.unverified.data || $mt.user.data.id == $mt.team.data.ownerId) {
      $mt.team.private.document = doc($mt.team.document, "private/data").withConverter(convertTeamPrivateData);
      $mt.team.private.data = (await getDoc($mt.team.private.document)).data();

      $mt.team.member.collection = collection($mt.team.document, "members").withConverter(convertMemberData);
      $mt.team.member.document = doc($mt.team.member.collection, $mt.user.data.id);
      $mt.team.member.data = (await getDoc($mt.team.member.document)).data();
    }

    loaded = true;
  }

  async function loadUser(userAccount: User) {
    let snapshot = await getDoc($mt.user.document);
    if (snapshot.exists()) {
      $mt.user.data = snapshot.data();
      if ($mt.user.data.teamId) {
        await loadTeam($mt.user.data.teamId);
      } else {
        loaded = true;
      }
    } else {
      $mt.user.data = {
        id: userAccount.uid,
        teamId: "",
      };
      loaded = true;
      await setDoc($mt.user.document, $mt.user.data).catch(console.error);
    }
  }

  firebaseApp = initializeApp(firebaseConfig);
  $mt.auth = getAuth(firebaseApp);
  firestore = getFirestore(firebaseApp);
  
  onAuthStateChanged($mt.auth, (user) => {
    if (user) {
      $mt.user.collection = collection(firestore, "users").withConverter(convertUserData);
      $mt.team.collection = collection(firestore, "teams").withConverter(convertTeamData);
      $mt.user.document = doc($mt.user.collection, user.uid).withConverter(convertUserData);
      loadUser(user);
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
        private: {
          data: null,
          document: null,
        },
        member: {
          data: null,
          document: null,
          collection: null,
        },
        unverified: {
          data: null,
          document: null,
          collection: null,
        },
      };
      loaded = true;
    }
  });
</script>

<header>
  <h1>MeanTrack</h1>
  <p>A work-in-progress FRC hour tracking web app</p>
</header>

{#if loaded}
  <UserMenu />
  {#if $mt.team.data}
    <TeamMenu />
  {:else if $mt.user.data}
    <TeamlessMenu />
  {/if}
{/if}

<footer>
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
</footer>
