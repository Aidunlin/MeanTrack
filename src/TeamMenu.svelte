<script lang="ts">
  import {
    CollectionReference,
    deleteDoc,
    DocumentReference,
    getDocs,
    orderBy,
    query,
    setDoc,
    where,
  } from "firebase/firestore";
  import type { UserData, TeamData } from "./Global.svelte";

  export let userData: UserData;
  export let userDoc: DocumentReference<UserData>;
  export let usersColl: CollectionReference<UserData>;

  export let teamData: TeamData;
  export let teamDoc: DocumentReference<TeamData>;

  let teamMembers: UserData[] = [];
  let unverifiedMembers: UserData[] = [];

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText(teamData.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", teamData.id);
    }
  }

  function refreshMembers() {
    teamMembers = [];
    unverifiedMembers = [];
    let q = query(
      usersColl,
      where("teamId", "==", teamDoc.id),
      orderBy("name")
    );
    getDocs(q).then((querySnapshot) => {
      querySnapshot.forEach((memberDoc) => {
        if (teamData.members.includes(memberDoc.id)) {
          teamMembers = [...teamMembers, memberDoc.data()];
        } else {
          unverifiedMembers = [...unverifiedMembers, memberDoc.data()];
        }
      });
    });
  }

  function verifyMember(id: string) {
    if (!teamData.members.includes(id)) {
      teamData.members.push(id);
      setDoc(teamDoc, teamData).then(refreshMembers).catch(console.error);
    }
  }

  function unverifyMember(id: string) {
    teamData.members = teamData.members.filter((member) => member != id);
    setDoc(teamDoc, teamData).then(refreshMembers).catch(console.error);
  }

  function deleteTeam() {
    if (confirm("Are you sure?")) {
      deleteDoc(teamDoc)
        .then(() => {
          teamData = null;
          userData.teamId = "";
          setDoc(userDoc, userData).catch(console.error);
        })
        .catch(console.error);
    }
  }

  function leaveTeam() {
    if (confirm("Are you sure?")) {
      userData.teamId = "";
      setDoc(userDoc, userData).catch(console.error);
    }
  }
</script>

<h2>Team {teamData.number}</h2>
<p>{teamData.name}</p>
{#if userData.id == teamData.ownerId}
  <p>
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
      {#each teamMembers as member}
        <tr>
          <td>{member.name}</td>
          <td>{member.hours.toFixed(2)}</td>
          <td>
            <button on:click={() => unverifyMember(member.id)}>Unverify</button>
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
            <button on:click={() => verifyMember(member.id)}>Verify</button>
          </td>
        </tr>
      {/each}
    </table>
  </details>
  <p><button on:click={deleteTeam}>Delete team</button></p>
{:else}
  {#if !teamData.members.includes(userData.id)}
    <p>UNVERIFIED</p>
  {/if}
  <p><button on:click={leaveTeam}>Leave team</button></p>
{/if}
