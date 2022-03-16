<script lang="ts">
  import { deleteDoc, doc } from "firebase/firestore";
  import { days, Log, mt, memberManagement } from "./Global.svelte";

  let sunday = getThisWeekSunday();
  let sundayPrint = updateSundayPrint();

  function getThisWeekSunday() {
    let today = new Date();
    let currentYear = today.getFullYear();
    let currentMonth = today.getMonth();
    let currentDate = today.getDate();
    let currentDay = today.getDay();
    return new Date(currentYear, currentMonth, currentDate - currentDay);
  }

  function updateSundayPrint() {
    $memberManagement.verifiedMembers = $memberManagement.verifiedMembers;
    sunday = sunday;
    return sunday.toLocaleDateString(undefined, { dateStyle: "long" });
  }

  function incrementSunday() {
    sunday.setDate(sunday.getDate() + 7);
    sundayPrint = updateSundayPrint();
  }

  function decrementSunday() {
    sunday.setDate(sunday.getDate() - 7);
    sundayPrint = updateSundayPrint();
  }

  function removeMembers() {
    if (!confirm("Are you sure?")) return;

    let membersToRemove = $memberManagement.verifiedMembers.filter((member) => {
      let isOwner = member.id == $mt.team.data.ownerId;
      let isSelected = $memberManagement.selectedVerifiedIds.includes(member.id);
      return !isOwner && isSelected;
    });
    membersToRemove.forEach(async (member) => {
      await deleteDoc(doc($mt.team.member.collection, member.id)).catch(console.error);
    });
    $memberManagement.verifiedMembers = $memberManagement.verifiedMembers.filter((member) => {
      return !$memberManagement.selectedVerifiedIds.includes(member.id);
    });
    $memberManagement.selectedVerifiedIds = [];
  }

  function getMemberHours(logs: Log[]): number {
    let totalHours = 0;
    logs.forEach((log) => {
      totalHours += log.hours;
    });
    return totalHours;
  }

  function getLogHours(dayIndex: number, logs: Log[]): number {
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
</script>

<h3>Members</h3>
<p>
  <button on:click={decrementSunday}>&lt;&lt;</button>
  <button on:click={incrementSunday}>&gt;&gt;</button>
  |
  {#if $mt.user.data.id == $mt.team.data.ownerId}
    <button class="red" on:click={removeMembers} disabled={!$memberManagement.selectedVerifiedIds.length}>
      Remove
    </button>
  {/if}
</p>
<p>Week of {sundayPrint}</p>
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th style="border-right-width: 4px">Hours</th>
        {#each days as _day, i}
          <th>
            {new Date(sunday.getFullYear(), sunday.getMonth(), sunday.getDate() + i).toLocaleDateString(undefined, {
              dateStyle: "short",
            })}
          </th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each $memberManagement.verifiedMembers as member (member.id)}
        <tr>
          <td>
            <label>
              <input
                type="checkbox"
                bind:group={$memberManagement.selectedVerifiedIds}
                name="verified-members"
                value={member.id}
                disabled={$mt.user.data.id != $mt.team.data.ownerId}
              />
              {member.name}
              {#if member.name == $mt.team.member.data.name}
                (you)
              {/if}
            </label>
          </td>
          <td style="border-right-width: 4px;">{getMemberHours(member.logs).toFixed(1)}</td>
          {#each days as _day, i}
            {@const hours = getLogHours(i, member.logs)}
            <td>{hours ? hours.toFixed(1) : ""}</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>
