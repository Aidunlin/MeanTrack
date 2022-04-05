<script lang="ts">
  import { mt } from "./Global.svelte";

  function editGoal() {
    let goalPrompt = prompt("Enter a goal:");
    if (!goalPrompt) return;
    $mt.team = $mt.team.update({ goal: parseInt(goalPrompt) });
  }

  function copyId() {
    if ("clipboard" in navigator) navigator.clipboard.writeText($mt.user.data.teamId);
    else prompt("Copy the id:", $mt.user.data.teamId);
  }

  function leave() {
    if (!confirm(`Are you sure you want to leave ${$mt.team.data.name}?`)) return;
    $mt.loaded = false;
    $mt.user = $mt.user.set({ teamId: "" });
    $mt.team = null;
    $mt.member = null;
    $mt.unverified = null;
    $mt.loaded = true;
  }
</script>

<details open>
  <summary>{$mt.team.data.name}</summary>
  {#if $mt.member?.data}
    <p>Goal: {$mt.team.data.goal} hours</p>
    <p>
      <button on:click={copyId}>Copy id</button>
      {#if $mt.user.id == $mt.team.data.ownerId}
        <button on:click={editGoal}>Edit goal</button>
      {/if}
      <button on:click={leave}>Leave</button>
    </p>
  {:else}
    {#if $mt.unverified?.data}
      <p>You are unverified</p>
    {:else}
      <p>You've been removed from {$mt.team.data.name}</p>
    {/if}
    <p><button on:click={leave}>Leave</button></p>
  {/if}
</details>
