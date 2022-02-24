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
    orderBy,
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

  let teamMembers: UserData[] = [];
  let unverifiedMembers: UserData[] = [];

  function loadTeamDoc(id: string) {
    teamDoc = doc(teamsColl, id).withConverter(teamDataConv);
    getDoc(teamDoc)
      .then((teamSnapshot) => {
        teamData = teamSnapshot.data();
        if (teamData.ownerId == userData.id) {
          refreshMembers();
        }
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

  function verifyMember(id: string) {
    if (!teamData.members.includes(id)) {
      teamData.members.push(id);
      setDoc(teamDoc, teamData).then(refreshMembers).catch(console.error);
    }
  }

  function unverifyMember(id: string) {
    teamData.members = teamData.members.filter((member) => member != id);
    setDoc(teamDoc, teamData).then(refreshMembers).catch(console.error);
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

  function leaveTeam() {
    if (confirm("Are you sure?")) {
      userData.teamId = "";
      setDoc(userDoc, userData).catch(console.error);
    }
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

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText(teamData.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", teamData.id);
    }
  }

  function refreshMembers() {
    teamMembers = [];
    unverifiedMembers = [];
    let q = query(
      usersColl,
      where("teamId", "==", teamDoc.id),
      orderBy("name")
    );
    getDocs(q).then((querySnapshot) => {
      querySnapshot.forEach((memberDoc) => {
        if (teamData.members.includes(memberDoc.id)) {
          teamMembers = [...teamMembers, memberDoc.data()];
        } else {
          unverifiedMembers = [...unverifiedMembers, memberDoc.data()];
        }
      });
    });
  }
</script>

<header>
  <h1>MeanTrack</h1>
</header>
{#if userData}
  <h2>{userData.name}</h2>
  <p>Hours: {userData.hours.toFixed(2)}</p>
  {#if userData.lastAction.toMillis()}
    <p>
      {userData.tracking ? "Started:" : "Stopped:"}
      {userData.lastAction.toDate().toLocaleString(undefined, {
        dateStyle: "short",
        timeStyle: "short",
      })}
    </p>
  {/if}
  <p>
    <button on:click={toggleTracking}>
      {userData.tracking ? "Stop" : "Start"} tracking
    </button>
    <button on:click={logout}>Sign out</button>
  </p>
  {#if teamData}
    <h2>Team {teamData.number}</h2>
    <p>{teamData.name}</p>
    {#if userData.id == teamData.ownerId}
      <p>
        <button on:click={copyTeamId}>Copy id</button>
        <button on:click={refreshMembers}>Refresh</button>
      </p>
      <details open>
        <summary>Members</summary>
        <table>
          <tr>
            <th>Name</th>
            <th>Hours</th>
            <th>Actions</th>
          </tr>
          {#each teamMembers as member}
            <tr>
              <td>{member.name}</td>
              <td>{member.hours.toFixed(2)}</td>
              <td>
                <button on:click={() => unverifyMember(member.id)}>
                  Unverify
                </button>
              </td>
            </tr>
          {/each}
        </table>
      </details>
      <details>
        <summary>Unverified</summary>
        <table>
          <tr>
            <th>Name</th>
            <th>Actions</th>
          </tr>
          {#each unverifiedMembers as member}
            <tr>
              <td>{member.name}</td>
              <td>
                <button on:click={() => verifyMember(member.id)}>
                  Verify
                </button>
              </td>
            </tr>
          {/each}
        </table>
      </details>
      <p><button on:click={deleteTeam}>Delete team</button></p>
    {:else}
      {#if !teamData.members.includes(userData.id)}
        <p>UNVERIFIED</p>
      {/if}
      <p><button on:click={leaveTeam}>Leave team</button></p>
    {/if}
  {:else}
    <details open>
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
{:else}
  <p><button on:click={login}>Sign in</button></p>
{/if}
