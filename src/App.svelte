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
    query,
    where,
    getDocs,
    FirestoreDataConverter,
    CollectionReference,
  } from "firebase/firestore";

  const firebaseConfig: FirebaseOptions = {
    apiKey: "AIzaSyAQZgF7DJ0_ty-E436BZhZ9kFMsj8D7RLk",
    authDomain: "meantrack-97d77.firebaseapp.com",
    projectId: "meantrack-97d77",
    storageBucket: "meantrack-97d77.appspot.com",
    messagingSenderId: "267200471704",
    appId: "1:267200471704:web:8a054875c674aebcd6a6ed",
  };

  interface UserData {
    team: number;
    hours: number;
  }

  interface TeamData {
    hours: number;
    members: number;
  }

  const userDataConverter: FirestoreDataConverter<UserData> = {
    toFirestore: (data: UserData) => {
      return { team: data.team, hours: data.hours };
    },

    fromFirestore: (snapshot, options): UserData => {
      const data = snapshot.data(options);
      return { team: data.team, hours: data.hours };
    },
  };

  let firebaseApp: FirebaseApp;
  let auth: Auth;
  let firestore: Firestore;

  let userAccount: User;
  let userData: UserData = { team: 0, hours: 0 };
  let userDocument: DocumentReference<UserData>;
  let unsubUserDoc: Unsubscribe;

  let teamData: TeamData = { hours: 0, members: 0 };
  let usersCollection: CollectionReference;
  let queriedTeamData = false;

  function load() {
    firebaseApp = initializeApp(firebaseConfig);
    auth = getAuth(firebaseApp);
    firestore = getFirestore(firebaseApp);

    onAuthStateChanged(auth, (user) => {
      userAccount = user;

      if (userAccount) {
        usersCollection = collection(firestore, "users").withConverter(
          userDataConverter
        );
        userDocument = doc(usersCollection, userAccount.uid).withConverter(
          userDataConverter
        );

        unsubUserDoc = onSnapshot(
          userDocument,
          (doc) => (userData = doc.data())
        );

        getDoc(userDocument)
          .then((doc) => {
            if (!doc.exists()) {
              setDoc(userDocument, userData);
            }
          })
          .catch(console.log);
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
        queriedTeamData = false;
      })
      .catch(console.log);
  }

  function changeTeam() {
    userData.team = parseInt(prompt("Enter your team's number"));
    setDoc(userDocument, userData);
    queriedTeamData = false;
  }

  function getTeamHours() {
    queriedTeamData = false;
    teamData.hours = 0;
    teamData.members = 0;

    getDocs(query(usersCollection, where("team", "==", userData.team)))
      .then((querySnapShot) => {
        querySnapShot.forEach((doc) => {
          teamData.hours += doc.data().hours;
          teamData.members++;
        });

        queriedTeamData = true;
      })
      .catch(console.log);
  }
</script>

<svelte:window on:load={load} />

<h1>MeanTrack</h1>

{#if !userAccount}
  <button on:click={login}>Login with Google</button>
{:else}
  <button on:click={logout}>Logout</button>
  <button on:click={changeTeam}>Change Team</button>
  <p>Welcome, {userAccount.displayName}, Team {userData.team}</p>
  <p>Total hours: {userData.hours}</p>
  <button on:click={getTeamHours}>Team Stats</button>
  {#if queriedTeamData}
    <p>Total team hours: {teamData.hours}</p>
    <p>Total team members: {teamData.members}</p>
  {/if}
{/if}
