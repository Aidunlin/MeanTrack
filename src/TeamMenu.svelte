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
  let selectedVerifiedIds: string[] = [];
  let selectedUnverifiedIds: string[] = [];

  function refreshMembers() {
    verifiedMembers = [];
    unverifiedMembers = [];
    selectedVerifiedIds = [];
    selectedUnverifiedIds = [];
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

  function deleteTeam() {
    if (
      prompt(`Type "${$mt.teamData.name}" to delete your team}`) ==
      $mt.teamData.name
    ) {
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

  function unverifyMembers() {
    let selectedMembers = verifiedMembers.filter((member) =>
      selectedVerifiedIds.includes(member.id)
    );
    unverifiedMembers.push(...selectedMembers);
    unverifiedMembers = unverifiedMembers.sort((a, b) =>
      a.name > b.name ? 1 : -1
    );
    verifiedMembers = verifiedMembers.filter(
      (member) => !selectedVerifiedIds.includes(member.id)
    );
    $mt.teamData.members = $mt.teamData.members.filter(
      (memberId) => !selectedVerifiedIds.includes(memberId)
    );
    selectedVerifiedIds = [];
    updateDoc($mt.teamDocument, {
      members: $mt.teamData.members,
    }).catch(console.error);
  }

  function verifyMembers() {
    let selectedMembers = unverifiedMembers.filter((member) =>
      selectedUnverifiedIds.includes(member.id)
    );
    verifiedMembers.push(...selectedMembers);
    verifiedMembers = verifiedMembers.sort((a, b) =>
      a.name > b.name ? 1 : -1
    );
    unverifiedMembers = unverifiedMembers.filter(
      (member) => !selectedUnverifiedIds.includes(member.id)
    );
    $mt.teamData.members.push(...selectedUnverifiedIds);
    selectedUnverifiedIds = [];
    updateDoc($mt.teamDocument, {
      members: $mt.teamData.members,
    }).catch(console.error);
  }

  function leaveTeam() {
    if (confirm(`Are you sure you want to leave ${$mt.teamData.name}?`)) {
      $mt.teamData = null;
      $mt.userData.teamId = "";
      updateDoc($mt.userDocument, {
        teamId: $mt.userData.id,
      }).catch(console.error);
    }
  }

  $: refreshMembers();
</script>

<h2>Team {$mt.teamData.number}</h2>
<p>{$mt.teamData.name}</p>
<p>Goal: {$mt.teamData.goal} hours</p>
{#if $mt.userData.id == $mt.teamData.ownerId}
  <p>
    <button on:click={refreshMembers}>Refresh</button>
    <button on:click={editGoal}>Edit goal</button>
    <button on:click={copyTeamId}>Copy id</button>
    <button on:click={deleteTeam}>Delete team</button>
  </p>
  <h3>Members</h3>
  <p>
    <button on:click={unverifyMembers} disabled={!selectedVerifiedIds.length}>
      Unverify
    </button>
  </p>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Hours</th>
      </tr>
    </thead>
    <tbody>
      {#each verifiedMembers as member (member.id)}
        <tr>
          <td>
            <label>
              <input
                type="checkbox"
                bind:group={selectedVerifiedIds}
                name="verified-members"
                value={member.id}
              />
              {member.name}
            </label>
          </td>
          <td>{member.hours.toFixed(1)}</td>
        </tr>
      {/each}
    </tbody>
  </table>
  <h3>Unverified</h3>
  <p>
    <button on:click={verifyMembers} disabled={!selectedUnverifiedIds.length}>
      Verify
    </button>
  </p>
  <table>
    <thead>
      <tr>
        <th>Name</th>
      </tr>
    </thead>
    <tbody>
      {#each unverifiedMembers as member (member.id)}
        <tr>
          <td>
            <label>
              <input
                type="checkbox"
                bind:group={selectedUnverifiedIds}
                name="unverified-members"
                value={member.id}
              />
              {member.name}
            </label>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
{:else}
  {#if !$mt.teamData.members.includes($mt.userData.id)}
    <p>UNVERIFIED</p>
  {/if}
  <p><button on:click={leaveTeam}>Leave team</button></p>
{/if}
