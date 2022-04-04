<script lang="ts">
  import { doc, Timestamp } from "firebase/firestore/lite";
  import { ListFS, mt, SingleFS } from "./Global.svelte";

  let nameInput = $mt.auth.currentUser.displayName;
  let teamIdInput = "";
  let teamNameInput = "";

  async function join() {
    $mt.loaded = false;
    $mt.team = new SingleFS($mt.firestore, "teams", teamIdInput);
    if (await $mt.team.getData()) {
      $mt.unverified = new ListFS($mt.team.document, "unverifieds");
      $mt.user = $mt.user.set({ teamId: $mt.team.id });
      if ($mt.user.id == $mt.team.data.ownerId) {
        $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
        await $mt.member.getData();
      } else {
        $mt.unverified.document = doc($mt.unverified.collection, $mt.user.id);
        $mt.unverified = $mt.unverified.set({ name: nameInput });
      }
    } else {
      $mt.team = null;
      alert("Team not found!");
    }
    $mt.loaded = true;
  }

  async function create() {
    $mt.loaded = false;
    $mt.team = new SingleFS($mt.firestore, "teams", ".");
    $mt.team = $mt.team.set({ goal: 0, name: teamNameInput, ownerId: $mt.user.id });
    $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
    $mt.member = $mt.member.set({ lastAction: Timestamp.now(), logs: [], name: nameInput, tracking: false });
    $mt.unverified = new ListFS($mt.team.document, "unverifieds");
    $mt.user = $mt.user.set({ teamId: $mt.team.id });
    $mt.loaded = true;
  }
</script>

<p><label>Your name<br /><input bind:value={nameInput} /></label></p>
<details open>
  <summary>Join a team</summary>
  <p><label>Team id<br /><input bind:value={teamIdInput} /></label></p>
  <p><button disabled={teamIdInput.length == 0 || nameInput.length == 0} on:click={join}>Join</button></p>
</details>
<details>
  <summary>Create a team</summary>
  <p><label>Team name<br /><input bind:value={teamNameInput} /></label></p>
  <p><button disabled={teamNameInput.length == 0 || nameInput.length == 0} on:click={create}>Create</button></p>
</details>
