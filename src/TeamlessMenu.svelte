<script lang="ts">
  import { doc, Timestamp } from "firebase/firestore/lite";
  import { isOwner, ListFS, logOut, mt, SingleFS } from "./Global.svelte";

  let nameInput = $mt.auth.currentUser.displayName;
  let teamIdInput = "";
  let teamNameInput = "";

  async function join() {
    $mt.loaded = false;
    $mt.team = new SingleFS($mt.firestore, "teams", teamIdInput);
    if (await $mt.team.getData()) {
      $mt.unverified = new ListFS($mt.team.document, "unverifieds");
      $mt.user = $mt.user.set({ teamId: $mt.team.id });
      if (isOwner($mt)) {
        $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
        await $mt.member.getList();
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
    $mt.team = $mt.team.set({
      cutoffBegin: new Timestamp(0, 0),
      cutoffEnd: Timestamp.now(),
      goal: 0,
      name: teamNameInput,
      ownerId: $mt.user.id,
    });
    $mt.member = new ListFS($mt.team.document, "members", $mt.user.id);
    $mt.member = $mt.member.set({ lastAction: Timestamp.now(), logs: [], name: nameInput, tracking: false });
    $mt.unverified = new ListFS($mt.team.document, "unverifieds");
    $mt.user = $mt.user.set({ teamId: $mt.team.id });
    $mt.loaded = true;
  }
</script>

<label>Your name<input type="text" bind:value={nameInput} /></label>
<details open>
  <summary>Join a team</summary>
  <label>Team id<input type="text" bind:value={teamIdInput} /></label>
  <button disabled={!teamIdInput.length || !nameInput.length} on:click={join}>Join</button>
</details>
<details>
  <summary>Create a team</summary>
  <label>Team name<input type="text" bind:value={teamNameInput} /></label>
  <button disabled={!teamNameInput.length || !nameInput.length} on:click={create}>Create</button>
</details>
<p><button on:click={logOut}>Log out</button></p>
