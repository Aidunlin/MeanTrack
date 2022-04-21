<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { getLogType, logOut, mt } from "./Global.svelte";
  import Dialog from "./Dialog.svelte";

  let showShareIdDialog = false;
  let showEditDialog = false;
  let showAddLogTypeDialog = false;
  let showLeaveDialog = false;
  let newTypeNameValue = "";
  let cutoffBeginEditValue = cutoffToString(getLogType($mt).cutoffBegin?.toDate());
  let cutoffEndEditValue = cutoffToString(getLogType($mt).cutoffEnd?.toDate());
  let goalEditValue = getLogType($mt).goal;
  let nameEditValue = $mt.team.data.name;

  function cutoffToString(day: Date) {
    if (!day) return "";
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
    getLogType($mt).cutoffBegin = stringToCutoff(cutoffBeginEditValue);
    getLogType($mt).cutoffEnd = stringToCutoff(cutoffEndEditValue);
    getLogType($mt).goal = goalEditValue;
    $mt.team = $mt.team.update({
      logTypes: $mt.team.data.logTypes,
      name: nameEditValue,
    });
  }

  function addLogType() {
    $mt.team.data.logTypes.push({
      cutoffBegin: null,
      cutoffEnd: null,
      goal: 0,
      name: newTypeNameValue,
    });
    $mt.team = $mt.team.update({
      logTypes: $mt.team.data.logTypes,
    });
    newTypeNameValue = "";
    showAddLogTypeDialog = false;
  }

  function dialogHasChanges() {
    let cutoffBeginChanged = cutoffBeginEditValue != cutoffToString(getLogType($mt).cutoffBegin?.toDate());
    let cutoffEndChanged = cutoffEndEditValue != cutoffToString(getLogType($mt).cutoffEnd?.toDate());
    let goalChanged = goalEditValue != getLogType($mt).goal;
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
  <label>Share team id<input type="text" value={$mt.team.id} readonly /></label>
</Dialog>

<Dialog bind:open={showEditDialog}>
  <p>Edit {$mt.team.data.name}</p>
  {#if $mt.chosenLogType}
    <label>
      Log type
      <select bind:value={$mt.chosenLogType}>
        {#each $mt.team.data.logTypes as logType (logType.name)}
          <option value={logType.name}>{logType.name}</option>
        {/each}
      </select>
    </label>
    <label>Cutoff begin<input type="date" bind:value={cutoffBeginEditValue} /></label>
    <label>Cutoff end<input type="date" bind:value={cutoffEndEditValue} /></label>
    <label>Goal<input type="number" pattern="[0-9]*" bind:value={goalEditValue} /></label>
  {/if}
  <label>Name<input type="text" bind:value={nameEditValue} /></label>
  <span slot="dialog-button">
    <button
      on:click={editTeam}
      disabled={!(nameEditValue && dialogHasChanges())}
    >
      Apply
    </button>
    <button on:click={() => (showAddLogTypeDialog = true)}>Add log type</button>
  </span>
</Dialog>

<Dialog bind:open={showAddLogTypeDialog}>
  <p>Edit {$mt.team.data.name}</p>
  <label>New log type<input type="text" bind:value={newTypeNameValue} /></label>
  <button
    slot="dialog-button"
    on:click={addLogType}
    disabled={!newTypeNameValue || $mt.team.data.logTypes.some((type) => type.name == newTypeNameValue)}
  >
    Add
  </button>
</Dialog>

<Dialog bind:open={showLeaveDialog}>
  <p>Are you sure you want to leave?</p>
  <button slot="dialog-button" class="red" on:click={leave}>Confirm</button>
</Dialog>

<details open>
  <summary>{$mt.team.data.name}</summary>
  {#if $mt.member?.data}
    <label>
      Log type
      <select bind:value={$mt.chosenLogType}>
        {#each $mt.team.data.logTypes as logType (logType.name)}
          <option value={logType.name}>{logType.name}</option>
        {/each}
      </select>
    </label>
    {#if getLogType($mt).cutoffBegin && getLogType($mt).cutoffEnd}
      <p>
        Cutoff: {getLogType($mt).cutoffBegin.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
        - {getLogType($mt).cutoffEnd.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
      </p>
    {/if}
    {#if getLogType($mt).goal}
      <p>Goal: {getLogType($mt).goal} hours</p>
    {/if}
    <div class="buttons">
      <button on:click={() => (showShareIdDialog = true)}>Share id</button>
      {#if $mt.user.id == $mt.team.data.ownerId}
        <button on:click={() => (showEditDialog = true)}>Edit...</button>
      {/if}
      <button on:click={() => (showLeaveDialog = true)}>Leave</button>
    </div>
  {:else}
    {#if $mt.unverified?.data}
      <p>You are unverified</p>
    {:else}
      <p>You've been removed</p>
    {/if}
    <div class="buttons">
      <button on:click={() => (showLeaveDialog = true)}>Leave</button>
      <button on:click={logOut}>Log out</button>
    </div>
  {/if}
</details>
