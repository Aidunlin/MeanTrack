<script lang="ts">
  import { initializeApp, FirebaseApp, FirebaseOptions } from "firebase/app";
  import { getAuth, Auth, onAuthStateChanged, User } from "firebase/auth";
  import {
    doc,
    setDoc,
    getFirestore,
    Firestore,
    DocumentReference,
    getDoc,
    collection,
    CollectionReference,
    Timestamp,
  } from "firebase/firestore";
  import {
    UserData,
    TeamData,
    userDataConv,
    teamDataConv,
  } from "./Global.svelte";
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
  let auth: Auth;
  let firestore: Firestore;

  let userData: UserData;
  let userDoc: DocumentReference<UserData>;
  let usersColl: CollectionReference<UserData>;

  let teamData: TeamData;
  let teamDoc: DocumentReference<TeamData>;
  let teamsColl: CollectionReference<TeamData>;

  function loadTeamDoc(id: string) {
    teamDoc = doc(teamsColl, id).withConverter(teamDataConv);
    getDoc(teamDoc)
      .then((teamSnapshot) => {
        teamData = teamSnapshot.data();
      })
      .catch(console.error);
  }

  function loadUserDoc(userAccount: User) {
    getDoc(userDoc)
      .then((userSnapshot) => {
        if (userSnapshot.exists()) {
          userData = userSnapshot.data();
          if (userData.teamId) {
            loadTeamDoc(userData.teamId);
          }
        } else {
          userData = {
            id: userAccount.uid,
            hours: 0,
            lastAction: new Timestamp(0, 0),
            name: userAccount.displayName,
            teamId: "",
            tracking: false,
          };
          setDoc(userDoc, userData).catch(console.error);
        }
      })
      .catch(console.error);
  }

  firebaseApp = initializeApp(firebaseConfig);
  auth = getAuth(firebaseApp);
  firestore = getFirestore(firebaseApp);
  onAuthStateChanged(auth, (user) => {
    if (user) {
      usersColl = collection(firestore, "users").withConverter(userDataConv);
      teamsColl = collection(firestore, "teams").withConverter(teamDataConv);
      userDoc = doc(usersColl, user.uid).withConverter(userDataConv);
      loadUserDoc(user);
    } else {
      usersColl = null;
      teamsColl = null;
      userDoc = null;
      teamDoc = null;
      userData = null;
      teamData = null;
    }
  });
</script>

<header>
  <h1>MeanTrack</h1>
</header>
<UserMenu bind:auth bind:userData bind:userDoc />
{#if teamData}
  <TeamMenu
    bind:userData
    bind:userDoc
    bind:usersColl
    bind:teamData
    bind:teamDoc
  />
{:else if userData}
  <TeamlessMenu
    bind:userData
    bind:userDoc
    bind:teamData
    bind:teamDoc
    bind:teamsColl
  />
{/if}
