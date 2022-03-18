<script lang="ts">
  import { updateDoc } from "firebase/firestore";
  import { mt } from "./Global.svelte";
  import MembersView from "./MembersView.svelte";
  import UnverifiedsView from "./UnverifiedsView.svelte";

  function editGoal() {
    let goalInput = parseInt(prompt("Enter a goal:"));
    if (goalInput) {
      $mt.private.data.goal = isNaN(goalInput) ? $mt.private.data.goal : goalInput;
      updateDoc($mt.private.document, {
        goal: $mt.private.data.goal,
      }).catch(console.error);
    }
  }

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText($mt.team.data.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", $mt.team.data.id);
    }
  }

  function leaveTeam() {
    if (!confirm(`Are you sure you want to leave ${$mt.team.data.name}?`)) return;
    $mt.user.data.teamId = "";
    $mt.team.data = null;
    $mt.team.document = null;
    $mt.member = {
      collection: null,
      data: null,
      document: null,
    };
    $mt.private = {
      data: null,
      document: null,
    };
    $mt.unverified = {
      collection: null,
      data: null,
      document: null,
    };
    updateDoc($mt.user.document, {
      teamId: $mt.user.data.teamId,
    }).catch(console.error);
  }
</script>

<h2>Team {$mt.team.data.number}</h2>
<p>{$mt.team.data.name}</p>
{#if $mt.private.data && $mt.member.data}
  <p>Goal: {$mt.private.data.goal} hours</p>
  <p>
    <button on:click={copyTeamId}>Copy id</button>
    {#if $mt.user.data.id == $mt.team.data.ownerId}
      <button on:click={editGoal}>Edit goal</button>
    {/if}
    |
    <button on:click={leaveTeam}>Leave team</button>
  </p>
  <MembersView />
  {#if $mt.user.data.id == $mt.team.data.ownerId}
    <UnverifiedsView />
  {/if}
{:else}
  <p>UNVERIFIED</p>
  <button on:click={leaveTeam}>Leave team</button>
{/if}
