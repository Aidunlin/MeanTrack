<script lang="ts">
  import { mt } from "./Global.svelte";

  function editGoal() {
    let goalPrompt = prompt("Enter a goal:");
    if (goalPrompt) {
      $mt.team.updateData({
        goal: parseInt(goalPrompt),
      });
      $mt.team = $mt.team;
    }
  }

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText($mt.user.data.teamId);
      alert("Copied!");
    } else {
      prompt("Copy the id:", $mt.user.data.teamId);
    }
  }

  function leaveTeam() {
    if (!confirm(`Are you sure you want to leave ${$mt.team.data.name}?`)) return;
    $mt.loaded = false;
    $mt.user.data = {
      teamId: "",
    };
    $mt.team = null;
    $mt.member = null;
    $mt.unverified = null;
    $mt.loaded = true;
  }
</script>

<h2>
  {#if !$mt.team.data.name.toLowerCase().includes("team")}
    Team
  {/if}
  {$mt.team.data.name}
</h2>
{#if !$mt.unverified?.data && $mt.member?.data}
  <p>Goal: {$mt.team.data.goal} hours</p>
  <p>
    <button on:click={copyTeamId}>Copy id</button>
    {#if $mt.user.document.id == $mt.team.data.ownerId}
      <button on:click={editGoal}>Edit goal</button>
    {/if}
  </p>
{:else}
  <p>UNVERIFIED</p>
{/if}
{#if $mt.user.document.id != $mt.team.data.ownerId}
  <p><button on:click={leaveTeam}>Leave team</button></p>
{/if}
