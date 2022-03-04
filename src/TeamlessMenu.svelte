<script lang="ts">
  import {
    collection,
    doc,
    getDoc,
    setDoc,
    Timestamp,
    updateDoc,
  } from "firebase/firestore";
  import {
    convertMemberData,
    convertTeamPrivateData,
    convertUnverifiedData,
    mt,
  } from "./Global.svelte";

  let joinTeamId: string;
  let createTeamName: string;
  let createTeamNumber: number;

  async function joinTeam() {
    $mt.team.document = doc($mt.team.collection, joinTeamId);
    $mt.team.data = (await getDoc($mt.team.document)).data();

    if ($mt.team.data) {
      $mt.team.unverified.collection = collection(
        $mt.team.document,
        "unverified"
      ).withConverter(convertUnverifiedData);
      $mt.team.unverified.document = doc(
        $mt.team.unverified.collection,
        $mt.user.data.id
      );
      $mt.team.unverified.data = {
        id: $mt.user.data.id,
        name: $mt.auth.currentUser.displayName,
      };
      await setDoc(
        $mt.team.unverified.document,
        $mt.team.unverified.data
      ).catch(console.error);

      $mt.user.data.teamId = $mt.team.data.id;
      await updateDoc($mt.user.document, {
        teamId: $mt.user.data.teamId,
      }).catch(console.error);
    } else {
      console.error("Team data not found!");
      $mt.team.document = null;
      $mt.team.data = null;
    }
  }

  async function createTeam() {
    if (createTeamName && createTeamNumber) {
      $mt.team.document = doc($mt.team.collection);
      $mt.team.data = {
        id: $mt.team.document.id,
        name: createTeamName,
        number: createTeamNumber,
        ownerId: $mt.user.data.id,
      };
      await setDoc($mt.team.document, $mt.team.data).catch(console.error);

      $mt.team.private.document = doc(
        $mt.team.document,
        "private/data"
      ).withConverter(convertTeamPrivateData);
      $mt.team.private.data = {
        goal: 0,
      };
      await setDoc($mt.team.private.document, $mt.team.private.data).catch(
        console.error
      );

      $mt.team.member.collection = collection(
        $mt.team.document,
        "members"
      ).withConverter(convertMemberData);
      $mt.team.member.document = doc(
        $mt.team.member.collection,
        $mt.user.data.id
      );
      $mt.team.member.data = {
        id: $mt.user.data.id,
        hours: 0,
        lastAction: Timestamp.now(),
        name: $mt.auth.currentUser.displayName,
        tracking: false,
      };
      await setDoc($mt.team.member.document, $mt.team.member.data).catch(
        console.error
      );

      $mt.team.unverified.collection = collection(
        $mt.team.document,
        "unverified"
      ).withConverter(convertUnverifiedData);

      $mt.user.data.teamId = $mt.team.document.id;
      await updateDoc($mt.user.document, {
        teamId: $mt.user.data.teamId,
      }).catch(console.error);
    }
  }
</script>

<h2>Join a team</h2>
<label>
  Team id
  <br />
  <input bind:value={joinTeamId} />
</label>
<p><button on:click={joinTeam}>Join</button></p>
<h2>Create a team</h2>
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
