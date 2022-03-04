<script lang="ts">
  import {
    deleteDoc,
    doc,
    getDocs,
    orderBy,
    query,
    setDoc,
    Timestamp,
    updateDoc,
  } from "firebase/firestore";
  import { MemberData, mt, UnverifiedData } from "./Global.svelte";

  let verifiedMembers: MemberData[] = [];
  let selectedVerifiedIds: string[] = [];

  let unverifiedMembers: UnverifiedData[] = [];
  let selectedUnverifiedIds: string[] = [];

  function refreshMembers() {
    if (!$mt.team.member.collection) return;

    verifiedMembers = [];
    selectedVerifiedIds = [];
    let membersQuery = query($mt.team.member.collection, orderBy("name"));
    getDocs(membersQuery).then((querySnapshot) => {
      querySnapshot.forEach((memberDoc) => {
        verifiedMembers = [...verifiedMembers, memberDoc.data()];
      });
    });

    unverifiedMembers = [];
    selectedUnverifiedIds = [];
    let unverifiedQuery = query(
      $mt.team.unverified.collection,
      orderBy("name")
    );
    getDocs(unverifiedQuery).then((querySnapshot) => {
      querySnapshot.forEach((unverifiedDoc) => {
        unverifiedMembers = [...unverifiedMembers, unverifiedDoc.data()];
      });
    });
  }

  function editGoal() {
    $mt.team.private.data.goal = parseInt(prompt("Enter a goal:"));
    updateDoc($mt.team.private.document, {
      goal: $mt.team.private.data.goal,
    }).catch(console.error);
  }

  function copyTeamId() {
    if ("clipboard" in navigator) {
      navigator.clipboard.writeText($mt.team.data.id);
      alert("Copied!");
    } else {
      prompt("Copy the id:", $mt.team.data.id);
    }
  }

  function unverifyMembers() {
    if (confirm("Are you sure?")) {
      let selectedMembers = verifiedMembers
        .filter((member) => selectedVerifiedIds.includes(member.id))
        .map((verified): UnverifiedData => {
          return {
            id: verified.id,
            name: verified.name,
          };
        });
      unverifiedMembers.push(...selectedMembers);
      unverifiedMembers = unverifiedMembers.sort((a, b) =>
        a.name > b.name ? 1 : -1
      );
      verifiedMembers = verifiedMembers.filter(
        (member) => !selectedVerifiedIds.includes(member.id)
      );
      selectedVerifiedIds.forEach((id) => {
        deleteDoc(doc($mt.team.member.collection, id)).catch(console.error);
        let d = doc($mt.team.unverified.collection, id);
        setDoc(
          d,
          selectedMembers.find((member) => member.id == id)
        );
      });
      selectedVerifiedIds = [];
    }
  }

  function verifyMembers() {
    if (confirm("Are you sure?")) {
      let selectedMembers = unverifiedMembers
        .filter((member) => selectedUnverifiedIds.includes(member.id))
        .map((unverified): MemberData => {
          return {
            id: unverified.id,
            hours: 0,
            lastAction: Timestamp.now(),
            name: unverified.name,
            tracking: false,
          };
        });
      verifiedMembers.push(...selectedMembers);
      verifiedMembers = verifiedMembers.sort((a, b) =>
        a.name > b.name ? 1 : -1
      );
      unverifiedMembers = unverifiedMembers.filter(
        (member) => !selectedUnverifiedIds.includes(member.id)
      );
      selectedUnverifiedIds.forEach((id) => {
        deleteDoc(doc($mt.team.unverified.collection, id)).catch(console.error);
        let d = doc($mt.team.member.collection, id);
        setDoc(
          d,
          selectedMembers.find((member) => member.id == id)
        );
      });
      selectedUnverifiedIds = [];
    }
  }

  function leaveTeam() {
    if (confirm(`Are you sure you want to leave ${$mt.team.data.name}?`)) {
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
  }

  $: refreshMembers();
</script>

<h2>Team {$mt.team.data.number}</h2>
<p>{$mt.team.data.name}</p>
{#if $mt.team.private.data}
  <p>Goal: {$mt.team.private.data.goal} hours</p>
{:else}
  <p>UNVERIFIED</p>
{/if}
{#if $mt.user.data.id == $mt.team.data.ownerId}
  <p>
    <button on:click={refreshMembers}>Refresh</button>
    <button on:click={editGoal}>Edit goal</button>
    <button on:click={copyTeamId}>Copy id</button>
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
{/if}
<p><button on:click={leaveTeam}>Leave team</button></p>