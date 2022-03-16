<script lang="ts">
  import { getDocs, orderBy, query, updateDoc } from "firebase/firestore";
  import { memberManagement, mt } from "./Global.svelte";
  import MembersView from "./MembersView.svelte";
  import UnverifiedsView from "./UnverifiedsView.svelte";

  function refreshMembers() {
    if (!$mt.team.member.collection) return;

    let membersQuery = query($mt.team.member.collection, orderBy("name"));
    getDocs(membersQuery).then((querySnapshot) => {
      $memberManagement.verifiedMembers = [];
      $memberManagement.selectedVerifiedIds = [];
      querySnapshot.forEach((memberDoc) => {
        $memberManagement.verifiedMembers = [...$memberManagement.verifiedMembers, memberDoc.data()];
      });
    });

    let unverifiedQuery = query($mt.team.unverified.collection, orderBy("name"));
    getDocs(unverifiedQuery).then((querySnapshot) => {
      $memberManagement.unverifiedMembers = [];
      $memberManagement.selectedUnverifiedIds = [];
      querySnapshot.forEach((unverifiedDoc) => {
        $memberManagement.unverifiedMembers = [...$memberManagement.unverifiedMembers, unverifiedDoc.data()];
      });
    });
  }

  function editGoal() {
    let goalInput = parseInt(prompt("Enter a goal:"));
    if (goalInput) {
      $mt.team.private.data.goal = isNaN(goalInput) ? $mt.team.private.data.goal : goalInput;
      updateDoc($mt.team.private.document, {
        goal: $mt.team.private.data.goal,
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

    $mt.team.data = null;
    $mt.team.document = null;
    $mt.team.member = {
      collection: null,
      data: null,
      document: null,
    };
    $mt.team.private = {
      data: null,
      document: null,
    };
    $mt.team.unverified = {
      collection: null,
      data: null,
      document: null,
    };
    $mt.user.data.teamId = "";
    updateDoc($mt.user.document, {
      teamId: $mt.user.data.teamId,
    }).catch(console.error);
  }

  $: {
    refreshMembers();
  }
</script>

<h2>Team {$mt.team.data.number}</h2>
<p>{$mt.team.data.name}</p>

{#if $mt.team.private.data && $mt.team.member.data}
  <p>Goal: {$mt.team.private.data.goal} hours</p>
  <p>
    <button on:click={refreshMembers}>Refresh</button>
    |
    <button on:click={copyTeamId}>Copy id</button>
    {#if $mt.user.data.id == $mt.team.data.ownerId}
      <button on:click={editGoal}>Edit goal</button>
    {/if}
    |
    <button on:click={leaveTeam}>Leave team</button>
  </p>

  <MembersView />
  {#if $mt.user.data.id == $mt.team.data.ownerId && $memberManagement.unverifiedMembers.length}
    <UnverifiedsView />
  {/if}
{:else}
  <p>UNVERIFIED</p>
  <button on:click={leaveTeam}>Leave team</button>
{/if}
