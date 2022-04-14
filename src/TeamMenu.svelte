<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { logOut, mt } from "./Global.svelte";
  import Dialog from "./Dialog.svelte";

  let showShareIdDialog = false;
  let showEditDialog = false;
  let showLeaveDialog = false;
  let cutoffBeginEditValue = cutoffToString($mt.team.data.cutoffBegin?.toDate());
  let cutoffEndEditValue = cutoffToString($mt.team.data.cutoffEnd?.toDate());
  let goalEditValue = $mt.team.data.goal;
  let nameEditValue = $mt.team.data.name;

  function cutoffToString(day: Date) {
    let year = day.getFullYear().toString();
    let month = (day.getMonth() + 1).toString().padStart(2, "0");
    let date = day.getDate().toString().padStart(2, "0");
    return `${year}-${month}-${date}`;
  }

  function stringToCutoff(format: string): Timestamp {
    let splitFormat = format.split("-");
    let date = new Date(parseInt(splitFormat[0]), parseInt(splitFormat[1]) - 1, parseInt(splitFormat[2]));
    return Timestamp.fromDate(date);
  }

  function editTeam() {
    $mt.team = $mt.team.update({
      cutoffBegin: stringToCutoff(cutoffBeginEditValue),
      cutoffEnd: stringToCutoff(cutoffEndEditValue),
      goal: goalEditValue,
      name: nameEditValue,
    });
  }

  function dialogHasChanges() {
    let cutoffBeginChanged = cutoffBeginEditValue != cutoffToString($mt.team.data.cutoffBegin?.toDate());
    let cutoffEndChanged = cutoffEndEditValue != cutoffToString($mt.team.data.cutoffEnd?.toDate());
    let goalChanged = goalEditValue != $mt.team.data.goal;
    let nameChanged = nameEditValue != $mt.team.data.name;
    return cutoffBeginChanged || cutoffEndChanged || goalChanged || nameChanged;
  }

  function leave() {
    $mt.user = $mt.user.set({ teamId: "" });
    $mt.team = $mt.member = $mt.unverified = null;
    showLeaveDialog = false;
  }
</script>

<Dialog bind:open={showShareIdDialog}>
  <label>Share team id:<br /><input type="text" value={$mt.team.id} readonly /></label>
</Dialog>

<Dialog bind:open={showEditDialog}>
  <p>Edit {$mt.team.data.name}</p>
  <label>Cutoff begin:<br /><input type="date" bind:value={cutoffBeginEditValue} /></label>
  <label>Cutoff end:<br /><input type="date" bind:value={cutoffEndEditValue} /></label>
  <label>Goal:<br /><input type="number" pattern="[0-9]*" bind:value={goalEditValue} /></label>
  <label>Name:<br /><input type="text" bind:value={nameEditValue} /></label>
  <button
    slot="buttons"
    on:click={editTeam}
    disabled={!(dialogHasChanges() && cutoffBeginEditValue && cutoffEndEditValue && goalEditValue && nameEditValue)}
  >
    Apply
  </button>
</Dialog>

<Dialog bind:open={showLeaveDialog}>
  <p>Are you sure you want to leave?</p>
  <button slot="buttons" class="red" on:click={leave}>Confirm</button>
</Dialog>

<details open>
  <summary>{$mt.team.data.name}</summary>
  {#if $mt.member?.data}
    {#if $mt.team.data.cutoffBegin && $mt.team.data.cutoffEnd}
      <p>
        Cutoff: {$mt.team.data.cutoffBegin.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
        - {$mt.team.data.cutoffEnd.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
      </p>
    {/if}
    <p>Goal: {$mt.team.data.goal} hours</p>
    <p>
      <button on:click={() => (showShareIdDialog = true)}>Share id</button>
      {#if $mt.user.id == $mt.team.data.ownerId}
        <button on:click={() => (showEditDialog = true)}>Edit</button>
      {/if}
      <button on:click={() => (showLeaveDialog = true)}>Leave</button>
    </p>
  {:else}
    {#if $mt.unverified?.data}
      <p>You are unverified</p>
    {:else}
      <p>You've been removed</p>
    {/if}
    <p>
      <button on:click={() => (showLeaveDialog = true)}>Leave</button>
      <button on:click={logOut}>Log out</button>
    </p>
  {/if}
</details>
