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
  } from "firebase/auth";
  import {
    doc,
    setDoc,
    getFirestore,
    Firestore,
    DocumentReference,
    getDoc,
    collection,
    FirestoreDataConverter,
    CollectionReference,
    Timestamp,
    deleteDoc,
    query,
    where,
    getDocs,
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
    id: string;
    hours: number;
    lastAction: Timestamp;
    name: string;
    teamId: string;
    tracking: boolean;
  }

  interface TeamData {
    id: string;
    members: string[];
    name: string;
    number: number;
    ownerId: string;
  }

  const userDataConv: FirestoreDataConverter<UserData> = {
    toFirestore: (data: UserData) => {
      return {
        hours: data.hours,
        lastAction: data.lastAction,
        name: data.name,
        teamId: data.teamId,
        tracking: data.tracking,
      };
    },
    fromFirestore: (snapshot, options) => {
      const user = snapshot.data(options);
      return {
        id: snapshot.id,
        hours: user.hours,
        lastAction: user.lastAction,
        name: user.name,
        teamId: user.teamId,
        tracking: user.tracking,
      };
    },
  };

  const teamDataConv: FirestoreDataConverter<TeamData> = {
    toFirestore: (data: TeamData) => {
      return {
        members: data.members,
        name: data.name,
        number: data.number,
        ownerId: data.ownerId,
      };
    },
    fromFirestore: (snapshot, options) => {
      const team = snapshot.data(options);
      return {
        id: snapshot.id,
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

  let userData: UserData;
  let userDoc: DocumentReference<UserData>;
  let usersColl: CollectionReference<UserData>;

  let teamData: TeamData;
  let teamDoc: DocumentReference<TeamData>;
  let teamsColl: CollectionReference<TeamData>;

  let joinTeamId: string;
  let createTeamName: string;
  let createTeamNumber: number;

  function loadTeamDoc(id: string) {
    teamDoc = doc(teamsColl, id).withConverter(teamDataConv);
    getDoc(teamDoc)
      .then((teamSnapshot) => (teamData = teamSnapshot.data()))
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

  function login() {
    signInWithPopup(auth, new GoogleAuthProvider()).catch(console.error);
  }

  function logout() {
    signOut(auth).catch(console.error);
  }

  function toggleTracking() {
    let newAction = Timestamp.now();
    let difference = newAction.toMillis() - userData.lastAction.toMillis();
    if (userData.tracking) {
      userData.hours += difference / 1000 / 3600;
    }
    userData.tracking = !userData.tracking;
    userData.lastAction = newAction;
    setDoc(userDoc, userData).catch(console.error);
  }

  function joinTeam() {
    teamDoc = doc(teamsColl, joinTeamId).withConverter(teamDataConv);
    getDoc(teamDoc)
      .then((teamSnapshot) => {
        if (teamSnapshot.exists()) {
          teamData = teamSnapshot.data();
          userData.teamId = teamData.id;
          setDoc(userDoc, userData).catch(console.error);
        }
      })
      .catch(console.error);
  }

  function createTeam() {
    if (createTeamName && createTeamNumber) {
      teamDoc = doc(teamsColl).withConverter(teamDataConv);
      teamData = {
        id: teamDoc.id,
        name: createTeamName,
        number: createTeamNumber,
        members: [userData.id],
        ownerId: userData.id,
      };
      setDoc(teamDoc, teamData)
        .then(() => {
          userData.teamId = teamDoc.id;
          setDoc(userDoc, userData).catch(console.error);
        })
        .catch(console.error);
    }
  }

  let teamMembers: UserData[] = [];

  function getTeamMembers() {
    teamMembers = [];
    const q = query(usersColl, where("teamId", "==", teamDoc.id));

    getDocs(q).then((query) => {
      query.forEach((memberDoc) => {
        if (teamData.members.includes(memberDoc.id)) {
          teamMembers = [...teamMembers, memberDoc.data()];
        }
      });
    });
  }

  let unverifiedMembers: UserData[] = [];

  function getUnverifiedMembers() {
    unverifiedMembers = [];
    const q = query(usersColl, where("teamId", "==", teamDoc.id));

    getDocs(q).then((query) => {
      query.forEach((memberDoc) => {
        if (!teamData.members.includes(memberDoc.id)) {
          unverifiedMembers = [...unverifiedMembers, memberDoc.data()];
        }
      });
    });
  }

  function verifyMember(id: string) {
    if (!teamData.members.includes(id)) {
      teamData.members.push(id);
      setDoc(teamDoc, teamData)
        .then(() => {
          getTeamMembers();
          getUnverifiedMembers();
        })
        .catch(console.error);
    }
  }

  function deleteTeam() {
    if (confirm("Are you sure?")) {
      deleteDoc(teamDoc)
        .then(() => {
          teamData = null;
          userData.teamId = "";
          setDoc(userDoc, userData).catch(console.error);
        })
        .catch(console.error);
    }
  }

  function leaveTeam() {}

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
  {#if userData}
    <button on:click={logout}>Sign out</button>
  {:else}
    <button on:click={login}>Sign in</button>
  {/if}
</header>
{#if userData}
  <h2>You</h2>
  <p>{userData.name}</p>
  <p>Hours: <code>{userData.hours.toFixed(2)}</code></p>
  {#if userData.lastAction.toMillis()}
    <p>
      <span>{userData.tracking ? "Started:" : "Stopped:"}</span>
      <code>
        {userData.lastAction.toDate().toLocaleString(undefined, {
          dateStyle: "short",
          timeStyle: "short",
        })}
      </code>
    </p>
  {/if}
  <p>
    <button on:click={toggleTracking}>
      {userData.tracking ? "Stop" : "Start"} tracking
    </button>
  </p>
  <h2>Team</h2>
  {#if teamData}
    <p>{teamData.name}: <code>{teamData.number}</code></p>
    {#if userData.id == teamData.ownerId}
      <p>You own this team.</p>
      <p><button on:click={deleteTeam}>Delete team</button></p>
      <h3>Members</h3>
      <p><button on:click={getTeamMembers}>Refresh</button></p>
      {#if teamMembers.length}
        <ul>
          {#each teamMembers as member}
            <li>
              {member.name}
              <br />
              Hours: <code>{member.hours.toFixed(2)}</code>
            </li>
          {/each}
        </ul>
      {:else}
        <p>Team members will be listed here.</p>
      {/if}
      <h3>Unverified</h3>
      <p><button on:click={getUnverifiedMembers}>Refresh</button></p>
      {#if unverifiedMembers.length}
        <ul>
          {#each unverifiedMembers as member}
            <li>
              {member.name}
              <br />
              <button on:click={() => verifyMember(member.id)}>Verify</button>
            </li>
          {/each}
        </ul>
      {:else}
        <p>Unverified members will be listed here.</p>
      {/if}
    {:else}
      {#if !teamData.members.includes(userData.id)}
        <p>UNVERIFIED</p>
      {/if}
      <p><button on:click={leaveTeam}>Leave team</button></p>
    {/if}
  {:else}
    <details>
      <summary>Join a team</summary>
      <label>
        Team id
        <br />
        <input bind:value={joinTeamId} />
      </label>
      <p><button on:click={joinTeam}>Join</button></p>
    </details>
    <details>
      <summary>Create a team</summary>
      <label>
        Team name
        <br />
        <input bind:value={createTeamName} />
      </label>
      <br />
      <label>
        Team number
        <br />
        <input bind:value={createTeamNumber} type="number" />
      </label>
      <p><button on:click={createTeam}>Create</button></p>
    </details>
  {/if}
{/if}
