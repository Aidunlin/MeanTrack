<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query } from "firebase/firestore/lite";
  import { Log, mt } from "./Global.svelte";

  let selectedMembers: string[] = [];
  let sunday = thisSunday();

  function thisSunday() {
    let today = new Date();
    return new Date(today.getFullYear(), today.getMonth(), today.getDate() - today.getDay());
  }

  function updateSunday(i?: number) {
    if (i == null) {
      sunday = thisSunday();
    } else {
      sunday.setDate(sunday.getDate() + i);
      sunday = sunday;
    }
    $mt.cachedMembers = $mt.cachedMembers;
  }

  function removeMembers() {
    if (!confirm("Are you sure?")) return;
    $mt.cachedMembers = $mt.cachedMembers.filter((member) => {
      let shouldRemove = selectedMembers.includes(member.id) && member.id != $mt.team.data.ownerId;
      if (shouldRemove) {
        deleteDoc(doc($mt.member.collection, member.id)).catch(console.error);
      }
      return !shouldRemove;
    });
    selectedMembers = [];
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
      if (log.start.toDate().toDateString() == day.toDateString()) {
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
      selectedMembers = [];
      querySnapshot.forEach((memberDoc) => {
        $mt.cachedMembers = [...$mt.cachedMembers, { ...memberDoc.data(), id: memberDoc.id }];
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
  <button on:click={() => updateSunday()} disabled={sunday.toString() == thisSunday().toString()}>This week</button>
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
          <td class="name-col">
            <label>
              <input
                type="checkbox"
                bind:group={selectedMembers}
                value={member.id}
                disabled={$mt.user.document.id != $mt.team.data.ownerId || member.id == $mt.team.data.ownerId}
              />
              {member.name}
            </label>
          </td>
          <td class="hours-col">{getTotalHours(member.logs).toFixed(1)}</td>
          {#each [...Array(7).keys()] as dayIndex}
            {@const hours = getOneDayHours(dayIndex, member.logs)}
            <td class="day-col">{hours ? hours.toFixed(1) : ""}</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>
{#if $mt.user.document.id == $mt.team.data.ownerId}
  <p><button class="red" on:click={removeMembers} disabled={!selectedMembers.length}>Remove</button></p>
{/if}
