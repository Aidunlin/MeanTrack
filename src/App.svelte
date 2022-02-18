<script lang="ts">
  import { initializeApp, FirebaseApp, FirebaseOptions } from "firebase/app";
  import {
    getAuth,
    Auth,
    onAuthStateChanged,
    GoogleAuthProvider,
    signInWithPopup,
    signOut,
    User,
    Unsubscribe,
  } from "firebase/auth";
  import {
    doc,
    setDoc,
    getFirestore,
    Firestore,
    DocumentReference,
    getDoc,
    onSnapshot,
    collection,
    FirestoreDataConverter,
    CollectionReference,
    Timestamp,
    query,
    where,
    DocumentSnapshot,
  } from "firebase/firestore";
  import Icon from "./Icon.svelte";
  import IconButton from "./IconButton.svelte";

  const firebaseConfig: FirebaseOptions = {
    apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
    authDomain: "meantrack-97d77.firebaseapp.com",
    projectId: "meantrack-97d77",
    storageBucket: "meantrack-97d77.appspot.com",
    messagingSenderId: "267200471704",
    appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
  };

  interface UserData {
    hours: number;
    lastAction: Timestamp;
    teamId: string;
    tracking: boolean;
  }

  interface TeamData {
    members: string[];
    name: string;
    number: number;
    ownerId: string;
  }

  const userDataConv: FirestoreDataConverter<UserData> = {
    toFirestore: (data: UserData) => {
      return { ...data };
    },

    fromFirestore: (snapshot, options) => {
      const user = snapshot.data(options);
      return {
        hours: user.hours,
        lastAction: user.lastAction,
        teamId: user.teamId,
        tracking: user.tracking,
      };
    },
  };

  const teamDataConv: FirestoreDataConverter<TeamData> = {
    toFirestore: (data: TeamData) => {
      return { ...data };
    },

    fromFirestore: (snapshot, options) => {
      const team = snapshot.data(options);
      return {
        members: team.members,
        name: team.name,
        number: team.number,
        ownerId: team.ownerId,
      };
    },
  };

  let firebaseApp: FirebaseApp;
  let auth: Auth;
  let firestore: Firestore;
  let userAccount: User;

  let userData: UserData = {
    hours: 0,
    lastAction: new Timestamp(0, 0),
    teamId: "",
    tracking: false,
  };

  let userDoc: DocumentReference<UserData>;
  let unsubUserDoc: Unsubscribe;
  let usersColl: CollectionReference;

  let teamData: TeamData = {
    members: [],
    name: "",
    number: 0,
    ownerId: "",
  };

  let teamDoc: DocumentReference<TeamData>;
  let unsubTeamDoc: Unsubscribe;
  let teamsColl: CollectionReference;

  let isTeamOwner = false;
  let isVerified = false;

  async function loadUser() {
    usersColl = collection(firestore, "users").withConverter(userDataConv);
    userDoc = doc(usersColl, userAccount.uid).withConverter(userDataConv);

    teamsColl = collection(firestore, "teams").withConverter(teamDataConv);

    unsubUserDoc = onSnapshot(userDoc, (userSnapshot) => {
      if (userSnapshot.exists()) {
        if (userData.teamId != userSnapshot.data().teamId) {
          teamDoc = doc(teamsColl, userSnapshot.data().teamId).withConverter(
            teamDataConv
          );
          unsubTeamDoc = onSnapshot(
            teamDoc,
            (teamSnapshot) => (teamData = teamSnapshot.data())
          );
        }

        userData = userSnapshot.data();
      } else {
        setDoc(userDoc, userData).catch(console.log);
      }
    });
  }

  function load() {
    firebaseApp = initializeApp(firebaseConfig);
    auth = getAuth(firebaseApp);
    firestore = getFirestore(firebaseApp);

    onAuthStateChanged(auth, (user) => {
      userAccount = user;

      if (userAccount) {
        loadUser();
      }
    });
  }

  function login() {
    signInWithPopup(auth, new GoogleAuthProvider()).catch(console.log);
  }

  function logout() {
    signOut(auth)
      .then(() => {
        unsubUserDoc();
        unsubTeamDoc();
      })
      .catch(console.log);
  }

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - userData.lastAction.toMillis();

    if (userData.tracking) {
      userData.hours += difference / 1000 / 3600;
    }

    userData.tracking = !userData.tracking;
    userData.lastAction = newAction;
    setDoc(userDoc, userData).catch(console.log);
  }

  $: {
    if (userAccount) {
      isTeamOwner = teamData.ownerId == userAccount.uid;
      isVerified = teamData.members.includes(userAccount.uid) || isTeamOwner;
    }
  }
</script>

<svelte:window on:load={load} />

<div class="flex spaced center bg">
  <span class="icon-button padding">
    <Icon name="hourglass" />MeanTrack
  </span>
</div>

<div class="flex spaced center">
  {#if userAccount}
    <p>{userAccount.displayName}</p>

    {#if teamData.number}
      <p>{teamData.name} - {teamData.number}</p>
    {/if}

    {#if isTeamOwner}
      <p>(You own this team)</p>
    {:else if !isVerified}
      <p>(Unverified)</p>
    {/if}

    <button class="big" on:click={toggleTracking}>
      <Icon name={userData.tracking ? "pause" : "play"} />
    </button>
    <p>Hours: {userData.hours.toFixed(2)}</p>

    {#if userData.lastAction.toMillis()}
      <p>
        {userData.tracking ? "Started" : "Stopped"}:
        {userData.lastAction.toDate().toLocaleString(undefined, {
          dateStyle: "short",
          timeStyle: "short",
        })}
      </p>
    {/if}
  {:else}
    <IconButton icon="sign-in" text="Login with Google" on:click={login} />
  {/if}
</div>

<div class="flex spaced space-between bg">
  {#if userAccount}
    <IconButton icon="sign-out" text="Logout" on:click={logout} />
  {/if}
</div>
