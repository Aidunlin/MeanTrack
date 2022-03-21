<script lang="ts">
  import { collection, doc, getDoc, setDoc, updateDoc } from "firebase/firestore";
  import { convertData, mt } from "./Global.svelte";

  let joinTeamId: string;
  let createTeamName: string;
  let createTeamNumber: number;

  async function joinTeam() {
    if (!joinTeamId) return;
    $mt.loaded = false;
    $mt.team.document = doc($mt.team.collection, joinTeamId);
    $mt.team.data = (await getDoc($mt.team.document)).data();
    if ($mt.team.data) {
      $mt.unverified.collection = collection($mt.team.document, "unverifieds").withConverter(convertData.unverified);
      $mt.user.data.teamId = $mt.team.data.id;
      await updateDoc($mt.user.document, {
        teamId: $mt.user.data.teamId,
      }).catch(console.error);
      if ($mt.user.data.id == $mt.team.data.ownerId) {
        $mt.teamPrivate.document = doc($mt.team.document, "private/data").withConverter(convertData.teamPrivate);
        $mt.teamPrivate.data = (await getDoc($mt.teamPrivate.document)).data();
        $mt.member.collection = collection($mt.team.document, "members").withConverter(convertData.member);
        $mt.member.document = doc($mt.member.collection, $mt.user.data.id);
        $mt.member.data = (await getDoc($mt.member.document)).data();
      } else {
        $mt.unverified.document = doc($mt.unverified.collection, $mt.user.data.id);
        $mt.unverified.data = {
          id: $mt.user.data.id,
          name: $mt.auth.currentUser.displayName,
        };
        await setDoc($mt.unverified.document, $mt.unverified.data).catch(console.error);
      }
    } else {
      alert("Team not found!");
      $mt.team.document = null;
      $mt.team.data = null;
    }
    $mt.loaded = true;
  }

  async function createTeam() {
    if (!(createTeamName && createTeamNumber)) return;
    $mt.loaded = false;
    $mt.team.document = doc($mt.team.collection);
    $mt.team.data = {
      id: $mt.team.document.id,
      name: createTeamName,
      number: createTeamNumber,
      ownerId: $mt.user.data.id,
    };
    await setDoc($mt.team.document, $mt.team.data).catch(console.error);
    $mt.teamPrivate.document = doc($mt.team.document, "private/data").withConverter(convertData.teamPrivate);
    $mt.teamPrivate.data = {
      goal: 0,
    };
    await setDoc($mt.teamPrivate.document, $mt.teamPrivate.data).catch(console.error);
    $mt.member.collection = collection($mt.team.document, "members").withConverter(convertData.member);
    $mt.member.document = doc($mt.member.collection, $mt.user.data.id);
    $mt.member.data = {
      id: $mt.user.data.id,
      logs: [],
      name: $mt.auth.currentUser.displayName,
      tracking: false,
    };
    await setDoc($mt.member.document, $mt.member.data).catch(console.error);
    $mt.unverified.collection = collection($mt.team.document, "unverifieds").withConverter(convertData.unverified);
    $mt.user.data.teamId = $mt.team.document.id;
    await updateDoc($mt.user.document, {
      teamId: $mt.user.data.teamId,
    }).catch(console.error);
    $mt.loaded = true;
  }
</script>

<h2>Join a team</h2>
<p>
  <label>
    Team id
    <br />
    <input bind:value={joinTeamId} />
  </label>
</p>
<p><button disabled={!joinTeamId} on:click={joinTeam}>Join</button></p>
<h2>Create a team</h2>
<p>
  <label>
    Team name
    <br />
    <input bind:value={createTeamName} />
  </label>
</p>
<p>
  <label>
    Team number
    <br />
    <input bind:value={createTeamNumber} type="number" />
  </label>
</p>
<p><button disabled={!(createTeamName && createTeamNumber)} on:click={createTeam}>Create</button></p>
