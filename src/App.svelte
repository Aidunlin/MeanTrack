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
    hours: number;
    lastAction: Timestamp;
    team: number;
    tracking: boolean;
  }

  const userDataConverter: FirestoreDataConverter<UserData> = {
    toFirestore: (data: UserData) => {
      return { ...data };
    },

    fromFirestore: (snapshot, options): UserData => {
      const data = snapshot.data(options);
      return {
        hours: data.hours,
        lastAction: data.lastAction,
        team: data.team,
        tracking: data.tracking,
      };
    },
  };

  let firebaseApp: FirebaseApp;
  let auth: Auth;
  let firestore: Firestore;

  let userData: UserData = {
    hours: 0,
    lastAction: new Timestamp(0, 0),
    team: 0,
    tracking: false,
  };
  let userAccount: User;
  let userDocument: DocumentReference<UserData>;
  let unsubUserDoc: Unsubscribe;

  let usersCollection: CollectionReference;

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

        unsubUserDoc = onSnapshot(userDocument, (doc) => {
          Object.entries(doc.data()).forEach(data => {
            if (userData.hasOwnProperty(data[0])) {
              if (typeof data[1] == typeof userData[data[0]]) {
                userData[data[0]] = data[1];
              }
            }
          });
        });

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
      })
      .catch(console.log);
  }

  function changeTeam() {
    userData.team = parseInt(prompt("Enter your team's number"));
    setDoc(userDocument, userData).catch(console.log);
  }

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - userData.lastAction.toMillis();

    if (userData.tracking) {
      userData.hours += difference / 1000 / 3600;
    }

    userData.tracking = !userData.tracking;
    userData.lastAction = newAction;
    setDoc(userDocument, userData).catch(console.log);
  }
</script>

<svelte:window on:load={load} />

<h1>MeanTrack</h1>

{#if !userAccount}
  <button on:click={login}>Login with Google</button>
{:else}
  <button on:click={logout}>Logout</button>
  <button on:click={changeTeam}>Change Team</button>
  <p>Welcome, {userAccount.displayName} (Team {userData.team})</p>
  <p>Hours: {userData.hours.toFixed(2)}</p>
  <p>
    {userData.tracking ? "Started" : "Stopped"} tracking on
    {userData.lastAction
      .toDate()
      .toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" })}
  </p>
  <button on:click={toggleTracking}>
    {userData.tracking ? "Stop" : "Start"} tracking
  </button>
{/if}
