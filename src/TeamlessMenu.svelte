<script lang="ts">
  import { doc } from "firebase/firestore/lite";
  import { FSDataSet, mt } from "./Global.svelte";

  let nameInput = $mt.auth.currentUser.displayName;
  let joinTeamId: string;
  let createTeamName: string;

  async function joinTeam() {
    if (!(joinTeamId && nameInput)) return;
    $mt.loaded = false;
    $mt.team = new FSDataSet($mt.firestore, "teams", joinTeamId);
    if (await $mt.team.refreshData()) {
      $mt.unverified = new FSDataSet($mt.team.document, "unverifieds");
      await $mt.user.updateData({
        teamId: $mt.team.document.id,
      });
      if ($mt.user.document.id == $mt.team.data.ownerId) {
        $mt.teamPrivate = new FSDataSet($mt.team.document, "private", "data");
        await $mt.teamPrivate.refreshData();
        $mt.member = new FSDataSet($mt.team.document, "members", $mt.user.document.id);
        await $mt.member.refreshData();
      } else {
        $mt.unverified.document = doc($mt.unverified.collection, $mt.user.document.id);
        $mt.unverified.data = {
          name: nameInput,
        };
      }
    } else {
      $mt.team = null;
      alert("Team not found!");
    }
    $mt.loaded = true;
  }

  async function createTeam() {
    if (!(createTeamName && nameInput)) return;
    $mt.loaded = false;
    $mt.team = new FSDataSet($mt.firestore, "teams");
    $mt.team.document = doc($mt.team.collection);
    $mt.team.data = {
      name: createTeamName,
      ownerId: $mt.user.document.id,
    };
    $mt.teamPrivate = new FSDataSet($mt.team.document, "private", "data");
    $mt.teamPrivate.data = {
      goal: 0,
    };
    $mt.member = new FSDataSet($mt.team.document, "members", $mt.user.document.id);
    $mt.member.data = {
      logs: [],
      name: nameInput,
      tracking: false,
    };
    $mt.unverified = new FSDataSet($mt.team.document, "unverifieds");
    await $mt.user.updateData({
      teamId: $mt.team.document.id,
    });
    $mt.loaded = true;
  }
</script>

<p>
  <label>
    Your name
    <br />
    <input bind:value={nameInput} />
  </label>
</p>
<h2>Join a team</h2>
<p>
  <label>
    Team id
    <br />
    <input bind:value={joinTeamId} />
  </label>
</p>
<p><button disabled={!(joinTeamId && nameInput)} on:click={joinTeam}>Join</button></p>
<h2>Create a team</h2>
<p>
  <label>
    Team name
    <br />
    <input bind:value={createTeamName} />
  </label>
</p>
<p><button disabled={!(createTeamName && nameInput)} on:click={createTeam}>Create</button></p>
