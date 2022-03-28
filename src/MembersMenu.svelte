<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query } from "firebase/firestore/lite";
  import { Log, mt } from "./Global.svelte";

  let selectedMembers: string[] = [];
  let sunday = thisSunday();
  let weekDayNames = getWeekDays();

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

  function getHours(logs: Log[]) {
    let hours = {
      total: 0,
      days: [0, 0, 0, 0, 0, 0, 0],
    };
    let nextSunday = thisSunday();
    nextSunday.setDate(nextSunday.getDate() + 7);
    let weekStartMillis = thisSunday().getMilliseconds();
    let weekEndMillis = nextSunday.getMilliseconds();
    logs.forEach((log) => {
      hours.total += log.hours;
      if (log.start.toMillis() >= weekStartMillis && log.start.toMillis() < weekEndMillis) {
        hours.days[log.start.toDate().getDay()] += log.hours;
      }
    });
    return hours;
  }

  function getWeekDays() {
    return [...Array(7).keys()].map((dayIndex) => {
      let day = new Date(sunday.getFullYear(), sunday.getMonth(), sunday.getDate() + dayIndex);
      return day.toLocaleDateString(undefined, { dateStyle: "short" });
    });
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
    weekDayNames = getWeekDays();
  }
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
