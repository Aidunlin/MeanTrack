<script lang="ts">
  import { FirebaseApp, FirebaseOptions, initializeApp } from "firebase/app";
  import { getAuth, onAuthStateChanged, User } from "firebase/auth";
  import {
    collection,
    doc,
    Firestore,
    getDoc,
    getFirestore,
    setDoc,
    Timestamp,
  } from "firebase/firestore";
  import { convertTeamData, convertUserData, mt } from "./Global.svelte";
  import TeamMenu from "./TeamMenu.svelte";
  import TeamlessMenu from "./TeamlessMenu.svelte";
  import UserMenu from "./UserMenu.svelte";

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

  function loadTeamDocument(id: string) {
    $mt.teamDocument = doc($mt.teamCollection, id).withConverter(
      convertTeamData
    );
    getDoc($mt.teamDocument)
      .then((teamSnapshot) => {
        $mt.teamData = teamSnapshot.data();
        loaded = true;
      })
      .catch(console.error);
  }

  function loadUserDocument(userAccount: User) {
    getDoc($mt.userDocument)
      .then((userSnapshot) => {
        if (userSnapshot.exists()) {
          $mt.userData = userSnapshot.data();
          if ($mt.userData.teamId) {
            loadTeamDocument($mt.userData.teamId);
          } else {
            loaded = true;
          }
        } else {
          $mt.userData = {
            id: userAccount.uid,
            hours: 0,
            lastAction: new Timestamp(0, 0),
            name: userAccount.displayName,
            teamId: "",
            tracking: false,
          };
          loaded = true;
          setDoc($mt.userDocument, $mt.userData).catch(console.error);
        }
      })
      .catch(console.error);
  }

  firebaseApp = initializeApp(firebaseConfig);
  $mt.auth = getAuth(firebaseApp);
  firestore = getFirestore(firebaseApp);
  onAuthStateChanged($mt.auth, (user) => {
    if (user) {
      $mt.userCollection = collection(firestore, "users").withConverter(
        convertUserData
      );
      $mt.teamCollection = collection(firestore, "teams").withConverter(
        convertTeamData
      );
      $mt.userDocument = doc($mt.userCollection, user.uid).withConverter(
        convertUserData
      );
      loadUserDocument(user);
    } else {
      $mt.userCollection = null;
      $mt.teamCollection = null;
      $mt.userDocument = null;
      $mt.teamDocument = null;
      $mt.userData = null;
      $mt.teamData = null;
      loaded = true;
    }
  });
</script>

<header>
  <h1>MeanTrack</h1>
</header>
{#if loaded}
  <UserMenu />
  {#if $mt.teamData}
    <TeamMenu />
  {:else if $mt.userData}
    <TeamlessMenu />
  {/if}
{/if}
