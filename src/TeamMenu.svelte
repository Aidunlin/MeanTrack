<script lang="ts">
  import {
    deleteDoc,
    getDocs,
    orderBy,
    query,
    updateDoc,
    where,
  } from "firebase/firestore";
  import { mt, UserData } from "./Global.svelte";

  let verifiedMembers: UserData[] = [];
  let unverifiedMembers: UserData[] = [];

  function editGoal() {
    $mt.teamData.goal = parseInt(prompt("Enter a goal:"));
    updateDoc($mt.teamDocument, {
      goal: $mt.teamData.goal,
    }).catch(console.error);
  }

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText($mt.teamData.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", $mt.teamData.id);
    }
  }

  function refreshMembers() {
    verifiedMembers = [];
    unverifiedMembers = [];
    let membersQuery = query(
      $mt.userCollection,
      where("teamId", "==", $mt.teamDocument.id),
      orderBy("name")
    );
    getDocs(membersQuery).then((querySnapshot) => {
      querySnapshot.forEach((memberDoc) => {
        if ($mt.teamData.members.includes(memberDoc.id)) {
          verifiedMembers = [...verifiedMembers, memberDoc.data()];
        } else {
          unverifiedMembers = [...unverifiedMembers, memberDoc.data()];
        }
      });
    });
  }

  function verifyMember(member: UserData) {
    if (!$mt.teamData.members.includes(member.id)) {
      verifiedMembers.push(member);
      verifiedMembers = verifiedMembers.sort((a, b) =>
        a.name > b.name ? 1 : -1
      );
      unverifiedMembers = unverifiedMembers.filter((m) => m.id != member.id);
      $mt.teamData.members.push(member.id);
      updateDoc($mt.teamDocument, {
        members: $mt.teamData.members,
      }).catch(console.error);
    }
  }

  function unverifyMember(member: UserData) {
    unverifiedMembers.push(member);
    unverifiedMembers = unverifiedMembers.sort((a, b) =>
      a.name > b.name ? 1 : -1
    );
    verifiedMembers = verifiedMembers.filter((m) => m.id != member.id);
    $mt.teamData.members = $mt.teamData.members.filter((m) => m != member.id);
    updateDoc($mt.teamDocument, {
      members: $mt.teamData.members,
    }).catch(console.error);
  }

  function deleteTeam() {
    if (confirm("Are you sure?")) {
      deleteDoc($mt.teamDocument)
        .then(() => {
          $mt.teamData = null;
          $mt.userData.teamId = "";
          updateDoc($mt.userDocument, {
            teamId: $mt.userData.id,
          }).catch(console.error);
        })
        .catch(console.error);
    }
  }

  function leaveTeam() {
    if (confirm("Are you sure?")) {
      $mt.teamData = null;
      $mt.userData.teamId = "";
      updateDoc($mt.userDocument, {
        teamId: $mt.userData.id,
      }).catch(console.error);
    }
  }
</script>

<h2>Team {$mt.teamData.number}</h2>
<p>{$mt.teamData.name}</p>
<p>Goal: {$mt.teamData.goal} hours</p>
{#if $mt.userData.id == $mt.teamData.ownerId}
  <p>
    <button on:click={editGoal}>Edit goal</button>
    <button on:click={copyTeamId}>Copy id</button>
    <button on:click={refreshMembers}>Refresh</button>
  </p>
  <details open>
    <summary>Members</summary>
    <table>
      <tr>
        <th>Name</th>
        <th>Hours</th>
        <th>Actions</th>
      </tr>
      {#each verifiedMembers as member}
        <tr>
          <td>{member.name}</td>
          <td>{member.hours.toFixed(2)}</td>
          <td>
            <button on:click={() => unverifyMember(member)}>Unverify</button>
          </td>
        </tr>
      {/each}
    </table>
  </details>
  <details>
    <summary>Unverified</summary>
    <table>
      <tr>
        <th>Name</th>
        <th>Actions</th>
      </tr>
      {#each unverifiedMembers as member}
        <tr>
          <td>{member.name}</td>
          <td>
            <button on:click={() => verifyMember(member)}>Verify</button>
          </td>
        </tr>
      {/each}
    </table>
  </details>
  <p><button on:click={deleteTeam}>Delete team</button></p>
{:else}
  {#if !$mt.teamData.members.includes($mt.userData.id)}
    <p>UNVERIFIED</p>
  {/if}
  <p><button on:click={leaveTeam}>Leave team</button></p>
{/if}
