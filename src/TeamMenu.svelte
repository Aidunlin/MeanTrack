<script lang="ts">
  import { Timestamp, updateDoc } from "firebase/firestore";
  import { Log, mt } from "./Global.svelte";

  function editGoal() {
    let goalPrompt = prompt("Enter a goal:");
    if (goalPrompt) {
      $mt.teamPrivate.data.goal = parseInt(goalPrompt);
      updateDoc($mt.teamPrivate.document, {
        goal: $mt.teamPrivate.data.goal,
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
    $mt.loaded = false;
    $mt.user.data.teamId = "";
    $mt.team.data = null;
    $mt.team.document = null;
    $mt.member = {
      collection: null,
      data: null,
      document: null,
    };
    $mt.teamPrivate = {
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
    $mt.loaded = true;
  }
</script>

<h2>Team {$mt.team.data.number}</h2>
<p>{$mt.team.data.name}</p>
{#if $mt.teamPrivate.data && $mt.member.data}
  <p>Goal: {$mt.teamPrivate.data.goal} hours</p>
  <p>
    <button on:click={copyTeamId}>Copy id</button>
    {#if $mt.user.data.id == $mt.team.data.ownerId}
      <button on:click={editGoal}>Edit goal</button>
    {/if}
  </p>
{:else}
  <p>UNVERIFIED</p>
{/if}
<p><button on:click={leaveTeam}>Leave team</button></p>
