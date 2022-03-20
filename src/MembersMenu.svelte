<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query } from "firebase/firestore";
  import { Log, mt } from "./Global.svelte";

  let sunday = getThisWeekSunday();

  function getThisWeekSunday() {
    let today = new Date();
    return new Date(today.getFullYear(), today.getMonth(), today.getDate() - today.getDay());
  }

  function updateSunday(i: number) {
    sunday.setDate(sunday.getDate() + i);
    $mt.cachedMembers = $mt.cachedMembers;
    sunday = sunday;
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
        $mt.cachedMembers = [...$mt.cachedMembers, memberDoc.data()];
      });
    });
  }

  refreshMembers();
</script>

<h3>Members</h3>
<p>
  <button on:click={refreshMembers}>Refresh</button>
  {#if window.innerWidth > 600}
    |
    <button on:click={() => updateSunday(-7)}>&lt;&lt;</button>
    <button on:click={() => updateSunday(7)}>&gt;&gt;</button>
  {/if}
  {#if $mt.user.data.id == $mt.team.data.ownerId}
    |
    <button class="red" on:click={removeMembers} disabled={!$mt.selectedMembers.length}>Remove</button>
  {/if}
</p>
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Hours</th>
        {#if window.innerWidth > 600}
          {#each [...Array(7).keys()] as dayIndex}
            {@const date = new Date(sunday.getFullYear(), sunday.getMonth(), sunday.getDate() + dayIndex)}
            <th>{date.toLocaleDateString(undefined, { dateStyle: "short" })}</th>
          {/each}
        {/if}
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
                name="members"
                value={member.id}
                disabled={$mt.user.data.id != $mt.team.data.ownerId}
              />
              {member.name}
            </label>
          </td>
          <td>{getTotalHours(member.logs).toFixed(1)}</td>
          {#if window.innerWidth > 600}
            {#each [...Array(7).keys()] as dayIndex}
              {@const hours = getOneDayHours(dayIndex, member.logs)}
              <td>{hours ? hours.toFixed(1) : ""}</td>
            {/each}
          {/if}
        </tr>
      {/each}
    </tbody>
  </table>
</div>
