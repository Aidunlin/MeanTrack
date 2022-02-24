<script lang="ts">
  import {
    CollectionReference,
    doc,
    DocumentReference,
    getDoc,
    setDoc,
  } from "firebase/firestore";
  import { UserData, TeamData, teamDataConv } from "./Global.svelte";

  export let userData: UserData;
  export let userDoc: DocumentReference<UserData>;

  export let teamData: TeamData;
  export let teamDoc: DocumentReference<TeamData>;
  export let teamsColl: CollectionReference<TeamData>;

  let joinTeamId: string;
  let createTeamName: string;
  let createTeamNumber: number;

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
</script>

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
