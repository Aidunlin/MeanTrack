<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query } from "firebase/firestore/lite";
  import { Log, MemberData, mt } from "./Global.svelte";

  let sunday = getThisWeekSunday();

  function getThisWeekSunday() {
    let today = new Date();
    return new Date(today.getFullYear(), today.getMonth(), today.getDate() - today.getDay());
  }

  function updateSunday(i?: number) {
    if (i == null) {
      sunday = getThisWeekSunday();
    } else {
      sunday.setDate(sunday.getDate() + i);
      sunday = sunday;
    }
    $mt.cachedMembers = $mt.cachedMembers;
  }

  function removeMembers() {
    if (!confirm("Are you sure?")) return;
    let membersToRemove = $mt.cachedMembers.filter((member) => {
      let isOwner = member.id == $mt.team.data.ownerId;
      let isSelected = $mt.selectedMembers.includes(member.id);
      return !isOwner && isSelected;
    });
    membersToRemove.forEach(async (member) => {
      await deleteDoc(doc($mt.member.collection, member.id)).catch(console.error);
    });
    $mt.cachedMembers = $mt.cachedMembers.filter((member) => {
      return !$mt.selectedMembers.includes(member.id);
    });
    $mt.selectedMembers = [];
  }

  function getTotalHours(logs: Log[]): number {
    let totalHours = 0;
    logs.forEach((log) => {
      totalHours += log.hours;
    });
    return totalHours;
  }

  function getOneDayHours(dayIndex: number, logs: Log[]): number {
    let hours = 0;
    let day = new Date(sunday);
    day.setDate(day.getDate() + dayIndex);
    logs.forEach((log) => {
      let sameDay = log.start.toDate().toDateString() == day.toDateString();
      if (sameDay) {
        hours += log.hours;
      }
    });
    return hours;
  }

  function refreshMembers() {
    if (!$mt.member.collection) return;
    let membersQuery = query($mt.member.collection, orderBy("name"));
    getDocs(membersQuery).then((querySnapshot) => {
      $mt.cachedMembers = [];
      $mt.selectedMembers = [];
      querySnapshot.forEach((memberDoc) => {
        let memberData: MemberData & { id: string } = { ...memberDoc.data(), id: memberDoc.id };
        $mt.cachedMembers = [...$mt.cachedMembers, memberData];
      });
    });
  }

  refreshMembers();
</script>

<h3>Members</h3>
<p>
  <button on:click={refreshMembers}>Refresh</button>
  <button on:click={() => updateSunday(-7)} title="Previous week">&lt;&lt;</button>
  <button on:click={() => updateSunday(7)} title="Next week">&gt;&gt;</button>
  <button on:click={() => updateSunday()} disabled={sunday.toString() == getThisWeekSunday().toString()}>
    This week
  </button>
</p>
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#each [...Array(7).keys()] as dayIndex}
          {@const date = new Date(sunday.getFullYear(), sunday.getMonth(), sunday.getDate() + dayIndex)}
          <th class="day-col">{date.toLocaleDateString(undefined, { dateStyle: "short" })}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each $mt.cachedMembers as member (member.id)}
        <tr>
          <td>
            <label>
              <input
                type="checkbox"
                bind:group={$mt.selectedMembers}
                value={member.id}
                disabled={$mt.user.document.id != $mt.team.data.ownerId}
              />
              {member.name}
            </label>
          </td>
          <td>{getTotalHours(member.logs).toFixed(1)}</td>
          {#each [...Array(7).keys()] as dayIndex}
            {@const hours = getOneDayHours(dayIndex, member.logs)}
            <td>{hours ? hours.toFixed(1) : ""}</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>
{#if $mt.user.document.id == $mt.team.data.ownerId}
  <p><button class="red" on:click={removeMembers} disabled={!$mt.selectedMembers.length}>Remove</button></p>
{/if}
