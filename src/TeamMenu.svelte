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

  let members: UserData[] = [];
  let unverifieds: UserData[] = [];

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText(teamData.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", teamData.id);
    }
  }

  function refreshMembers() {
    members = [];
    unverifieds = [];
    let q = query(
      usersColl,
      where("teamId", "==", teamDoc.id),
      orderBy("name")
    );
    getDocs(q).then((querySnapshot) => {
      querySnapshot.forEach((memberDoc) => {
        if (teamData.members.includes(memberDoc.id)) {
          members = [...members, memberDoc.data()];
        } else {
          unverifieds = [...unverifieds, memberDoc.data()];
        }
      });
    });
  }

  function verifyMember(member: UserData) {
    if (!teamData.members.includes(member.id)) {
      members.push(member);
      members = members.sort((a, b) => (a.name > b.name ? 1 : -1));
      unverifieds = unverifieds.filter((m) => m.id != member.id);
      teamData.members.push(member.id);
      setDoc(teamDoc, teamData).catch(console.error);
    }
  }

  function unverifyMember(member: UserData) {
    unverifieds.push(member);
    unverifieds = unverifieds.sort((a, b) => (a.name > b.name ? 1 : -1));
    members = members.filter((m) => m.id != member.id);
    teamData.members = teamData.members.filter((m) => m != member.id);
    setDoc(teamDoc, teamData).catch(console.error);
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
      {#each members as member}
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
      {#each unverifieds as unverified}
        <tr>
          <td>{unverified.name}</td>
          <td>
            <button on:click={() => verifyMember(unverified)}>Verify</button>
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
