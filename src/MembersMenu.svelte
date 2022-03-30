<script lang="ts">
  import { doc, Timestamp, writeBatch } from "firebase/firestore/lite";
  import { Log, mt } from "./Global.svelte";

  let selectedMembers: string[] = [];
  let sunday: Date;
  let nextSunday: Date;
  getThisWeek();
  let weekDayNames = getWeekDays(sunday);

  function thisSunday() {
    let today = new Date();
    return new Date(today.getFullYear(), today.getMonth(), today.getDate() - today.getDay());
  }

  function getThisWeek() {
    sunday = thisSunday();
    nextSunday = new Date(sunday);
    nextSunday.setDate(sunday.getDate() + 7);
  }

  function getPreviousWeek() {
    nextSunday = new Date(sunday);
    sunday.setDate(sunday.getDate() - 7);
  }

  function getNextWeek() {
    sunday = new Date(nextSunday);
    nextSunday.setDate(nextSunday.getDate() + 7);
  }

  function getWeekDays(sundayOfWeek: Date) {
    return [...Array(7).keys()].map((dayIndex) => {
      let day = new Date(sundayOfWeek.getFullYear(), sundayOfWeek.getMonth(), sundayOfWeek.getDate() + dayIndex);
      return day.toLocaleDateString(undefined, { dateStyle: "short" });
    });
  }

  function getHours(logs: Log[]) {
    let hours = { total: 0, days: [0, 0, 0, 0, 0, 0, 0] };
    let weekStartMillis = Timestamp.fromDate(sunday).toMillis();
    let weekEndMillis = Timestamp.fromDate(nextSunday).toMillis();
    logs.forEach((log) => {
      hours.total += log.hours;
      if (log.start.toMillis() >= weekStartMillis && log.start.toMillis() < weekEndMillis) {
        hours.days[log.start.toDate().getDay()] += log.hours;
      }
    });
    return hours;
  }

  function removeMembers() {
    if (!confirm("Are you sure?")) return;
    let removeBatch = writeBatch($mt.firestore);
    $mt.member.list = $mt.member.list.filter((member) => {
      let shouldRemove = selectedMembers.includes(member.id) && member.id != $mt.team.data.ownerId;
      if (shouldRemove) removeBatch.delete(doc($mt.member.collection, member.id));
      return !shouldRemove;
    });
    removeBatch.commit().catch(console.error);
    selectedMembers = [];
  }

  async function refreshMembers() {
    $mt.member.list = await $mt.member.getList();
  }

  $: {
    $mt.member = $mt.member;
    sunday = sunday;
    nextSunday = nextSunday;
    weekDayNames = getWeekDays(sunday);
  }
</script>

<details open={$mt.user.document.id == $mt.team.data.ownerId}>
  <summary>Members</summary>
  <p>Week of {sunday.toLocaleDateString(undefined, { dateStyle: "medium" })}</p>
  <p>
    <button on:click={refreshMembers} title="Refresh">↻</button>
    <button on:click={getThisWeek}>This week</button>
    <button on:click={getPreviousWeek} title="Previous week">&lt;</button>
    <button on:click={getNextWeek} title="Next week">&gt;</button>
  </p>
  <div class="table-wrap">
    <table>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#each weekDayNames as day}
          <th class="day-col">{day}</th>
        {/each}
      </tr>
      {#each $mt.member.list as member (member.id)}
        {@const hoursData = getHours(member.logs)}
        <tr>
          <td class="name-col">
            {#if $mt.user.document.id == $mt.team.data.ownerId && member.id != $mt.team.data.ownerId}
              <label>
                <input type="checkbox" bind:group={selectedMembers} value={member.id} />
                {member.name}
              </label>
            {:else}
              {member.name}
            {/if}
          </td>
          <td class="hours-col">{hoursData.total.toFixed(1)}</td>
          {#each hoursData.days as hours}
            <td class="day-col">{hours ? hours.toFixed(1) : ""}</td>
          {/each}
        </tr>
      {/each}
    </table>
  </div>
  {#if $mt.user.document.id == $mt.team.data.ownerId}
    <p><button class="red" on:click={removeMembers} disabled={!selectedMembers.length}>Remove</button></p>
  {/if}
</details>
