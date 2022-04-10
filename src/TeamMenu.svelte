<script lang="ts">
  import { mt } from "./Global.svelte";

  enum View {
    Default,
    ShareId,
    Edit,
    Leave,
  }

  let viewing = View.Default;
  let goalEditValue = $mt.team.data.goal;
  let nameEditValue = $mt.team.data.name;

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
      <p>Goal: {$mt.team.data.goal} hours</p>
      <p>
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
      </p>
    {/if}
  {:else if viewing == View.ShareId}
    <p>Id: <code>{$mt.team.id}</code></p>
    <p><button on:click={() => (viewing = View.Default)}>Done</button></p>
  {:else if viewing == View.Edit}
    <p>
      <label>Edit goal:<br /><input type="number" pattern="[0-9]*" bind:value={goalEditValue} /></label>
      <button on:click={editGoal} disabled={goalEditValue == $mt.team.data.goal}>Update</button>
    </p>
    <p>
      <label>Edit name:<br /><input type="text" bind:value={nameEditValue} /></label>
      <button on:click={editName} disabled={nameEditValue == $mt.team.data.name}>Update</button>
    </p>
    <p>
      <button on:click={() => (viewing = View.Default)}>Done</button>
    </p>
  {:else if viewing == View.Leave}
    <p>Are you sure you want to leave?</p>
    <p>
      <button class="red" on:click={leave}>Confirm</button>
      <button on:click={() => (viewing = View.Default)}>Cancel</button>
    </p>
  {/if}
</details>
