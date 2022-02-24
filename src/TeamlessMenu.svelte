<script lang="ts">
  import { doc, getDoc, setDoc, updateDoc } from "firebase/firestore";
  import { convertTeamData, mt } from "./Global.svelte";

  let joinTeamId: string;
  let createTeamName: string;
  let createTeamNumber: number;

  function joinTeam() {
    $mt.teamDocument = doc($mt.teamCollection, joinTeamId).withConverter(
      convertTeamData
    );
    getDoc($mt.teamDocument)
      .then((teamSnapshot) => {
        if (teamSnapshot.exists()) {
          $mt.teamData = teamSnapshot.data();
          $mt.userData.teamId = $mt.teamData.id;
          updateDoc($mt.userDocument, {
            teamId: $mt.userData.teamId,
          }).catch(console.error);
        }
      })
      .catch(console.error);
  }

  function createTeam() {
    if (createTeamName && createTeamNumber) {
      $mt.teamDocument = doc($mt.teamCollection).withConverter(convertTeamData);
      $mt.teamData = {
        id: $mt.teamDocument.id,
        goal: 0,
        name: createTeamName,
        number: createTeamNumber,
        members: [$mt.userData.id],
        ownerId: $mt.userData.id,
      };
      setDoc($mt.teamDocument, $mt.teamData)
        .then(() => {
          $mt.userData.teamId = $mt.teamDocument.id;
          updateDoc($mt.userDocument, {
            teamId: $mt.userData.teamId,
          }).catch(console.error);
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
