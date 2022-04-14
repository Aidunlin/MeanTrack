<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { logOut, mt } from "./Global.svelte";

  enum View {
    Default,
    ShareId,
    Edit,
    Leave,
  }

  let viewing = View.Default;
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

  function editCutoffBegin() {
    $mt.team = $mt.team.update({ cutoffBegin: stringToCutoff(cutoffBeginEditValue) });
  }

  function editCutoffEnd() {
    $mt.team = $mt.team.update({ cutoffEnd: stringToCutoff(cutoffEndEditValue) });
  }

  function editGoal() {
    $mt.team = $mt.team.update({ goal: goalEditValue });
  }

  function editName() {
    $mt.team = $mt.team.update({ name: nameEditValue });
  }

  function leave() {
    $mt.user = $mt.user.set({ teamId: "" });
    $mt.team = $mt.member = $mt.unverified = null;
    viewing = View.Default;
  }
</script>

<details open>
  <summary>{$mt.team.data.name}</summary>
  {#if viewing == View.Default}
    {#if $mt.member?.data}
      {#if $mt.team.data.cutoffBegin && $mt.team.data.cutoffEnd}
        <p>
          Cutoff: {$mt.team.data.cutoffBegin.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
          - {$mt.team.data.cutoffEnd.toDate().toLocaleString(undefined, { dateStyle: "medium" })}
        </p>
      {/if}
      <p>Goal: {$mt.team.data.goal} hours</p>
      <p class="overflow-wide">
        <button on:click={() => (viewing = View.ShareId)}>Share id</button>
        {#if $mt.user.id == $mt.team.data.ownerId}
          <button on:click={() => (viewing = View.Edit)}>Edit</button>
        {/if}
        <button on:click={() => (viewing = View.Leave)}>Leave</button>
      </p>
    {:else}
      {#if $mt.unverified?.data}
        <p>You are unverified</p>
      {:else}
        <p>You've been removed</p>
      {/if}
      <p>
        <button on:click={() => (viewing = View.Leave)}>Leave</button>
        <button on:click={logOut}>Log out</button>
      </p>
    {/if}
  {:else if viewing == View.ShareId}
    <p>Id: <code>{$mt.team.id}</code></p>
    <p><button on:click={() => (viewing = View.Default)}>Done</button></p>
  {:else if viewing == View.Edit}
    <p>
      <label>Cutoff begin:<br /><input type="date" bind:value={cutoffBeginEditValue} /></label>
      <button
        on:click={editCutoffBegin}
        disabled={cutoffBeginEditValue == cutoffToString($mt.team.data.cutoffBegin?.toDate())}
      >
        Update
      </button>
    </p>
    <p>
      <label>Cutoff end:<br /><input type="date" bind:value={cutoffEndEditValue} /></label>
      <button
        on:click={editCutoffEnd}
        disabled={cutoffEndEditValue == cutoffToString($mt.team.data.cutoffEnd?.toDate())}
      >
        Update
      </button>
    </p>
    <p>
      <label>Goal:<br /><input type="number" pattern="[0-9]*" bind:value={goalEditValue} /></label>
      <button on:click={editGoal} disabled={goalEditValue == $mt.team.data.goal}>Update</button>
    </p>
    <p>
      <label>Name:<br /><input type="text" bind:value={nameEditValue} /></label>
      <button on:click={editName} disabled={nameEditValue == $mt.team.data.name}>Update</button>
    </p>
    <p>
      <button on:click={() => (viewing = View.Default)}>Done</button>
    </p>
  {:else if viewing == View.Leave}
    <p>Are you sure you want to leave?</p>
    <p class="overflow-wide">
      <button class="red" on:click={leave}>Confirm</button>
      <button on:click={() => (viewing = View.Default)}>Cancel</button>
    </p>
  {/if}
</details>
