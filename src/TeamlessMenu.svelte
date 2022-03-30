<script lang="ts">
  import { doc, Timestamp } from "firebase/firestore/lite";
  import { FSDataSet, FSListSet, mt } from "./Global.svelte";

  let nameInput = $mt.auth.currentUser.displayName;
  let joinTeamId = "";
  let createTeamName = "";

  async function joinTeam() {
    $mt.loaded = false;
    $mt.team = new FSDataSet($mt.firestore, "teams", joinTeamId);
    if (await $mt.team.getData()) {
      $mt.unverified = new FSListSet($mt.team.document, "unverifieds");
      $mt.user = $mt.user.set({ teamId: $mt.team.document.id });
      if ($mt.user.document.id == $mt.team.data.ownerId) {
        $mt.member = new FSListSet($mt.team.document, "members", $mt.user.document.id);
        await $mt.member.getData();
      } else {
        $mt.unverified.document = doc($mt.unverified.collection, $mt.user.document.id);
        $mt.unverified = $mt.unverified.set({ name: nameInput });
      }
    } else {
      $mt.team = null;
      alert("Team not found!");
    }
    $mt.loaded = true;
  }

  async function createTeam() {
    $mt.loaded = false;
    $mt.team = new FSDataSet($mt.firestore, "teams", ".");
    $mt.team = $mt.team.set({ goal: 0, name: createTeamName, ownerId: $mt.user.document.id });
    $mt.member = new FSListSet($mt.team.document, "members", $mt.user.document.id);
    $mt.member = $mt.member.set({ lastAction: Timestamp.now(), logs: [], name: nameInput, tracking: false });
    $mt.unverified = new FSListSet($mt.team.document, "unverifieds");
    $mt.user = $mt.user.set({ teamId: $mt.team.document.id });
    $mt.loaded = true;
  }
</script>

<p><label>Your name<br /><input bind:value={nameInput} /></label></p>
<details open>
  <summary>Join a team</summary>
  <p><label>Team id<br /><input bind:value={joinTeamId} /></label></p>
  <p><button disabled={joinTeamId.length == 0 || nameInput.length == 0} on:click={joinTeam}>Join</button></p>
</details>
<details>
  <summary>Create a team</summary>
  <p><label>Team name<br /><input bind:value={createTeamName} /></label></p>
  <p><button disabled={createTeamName.length == 0 || nameInput.length == 0} on:click={createTeam}>Create</button></p>
</details>
