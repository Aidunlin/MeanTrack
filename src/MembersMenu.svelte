<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query, Timestamp } from "firebase/firestore/lite";
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
    let hours = {
      total: 0,
      days: [0, 0, 0, 0, 0, 0, 0],
    };
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
    $mt.cachedMembers = $mt.cachedMembers.filter((member) => {
      let shouldRemove = selectedMembers.includes(member.id) && member.id != $mt.team.data.ownerId;
      if (shouldRemove) {
        deleteDoc(doc($mt.member.collection, member.id)).catch(console.error);
      }
      return !shouldRemove;
    });
    selectedMembers = [];
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

  $: {
    $mt.cachedMembers = $mt.cachedMembers;
    sunday = sunday;
    nextSunday = nextSunday;
    weekDayNames = getWeekDays(sunday);
  }
</script>

<h3>Members</h3>
<p>
  <button on:click={refreshMembers} title="Refresh">↻</button>
  <button on:click={getThisWeek}>Today</button>
  <button on:click={getPreviousWeek} title="Previous week">&lt;&lt;</button>
  <button on:click={getNextWeek} title="Next week">&gt;&gt;</button>
</p>
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#each weekDayNames as day}
          <th class="day-col">{day}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each $mt.cachedMembers as member (member.id)}
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
    </tbody>
  </table>
</div>
{#if $mt.user.document.id == $mt.team.data.ownerId}
  <p><button class="red" on:click={removeMembers} disabled={!selectedMembers.length}>Remove</button></p>
{/if}
