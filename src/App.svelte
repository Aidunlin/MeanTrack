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
          Object.entries(doc.data()).forEach((data) => {
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

<div class="flex spaced bg">
  <span class="padding">MeanTrack</span>
  {#if userAccount}
    <button on:click={logout}>Logout</button>
  {:else}
    <button on:click={login}>Login with Google</button>
  {/if}
</div>

{#if userAccount}
  <div class="flex spaced">
    <span class="padding">{userAccount.displayName} - Team {userData.team}</span>
    <span class="padding">Hours: {userData.hours.toFixed(2)}</span>
    <button on:click={toggleTracking}>
      {userData.tracking ? "Stop" : "Start"} tracking
    </button>
    <span class="padding">
      {userData.tracking ? "Started" : "Stopped"}:
      {userData.lastAction
        .toDate()
        .toLocaleString(undefined, { dateStyle: "short", timeStyle: "short" })}
    </span>
  </div>

  <div class="flex spaced extend-down bg">
    <button on:click={changeTeam}>Change Team</button>
  </div>
{/if}
