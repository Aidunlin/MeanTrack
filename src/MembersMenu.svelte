<script lang="ts">
  import { doc, Timestamp, writeBatch } from "firebase/firestore/lite";
  import { Log, MemberData, mt, sameDay, Week } from "./Global.svelte";

  let selectedMembers: string[] = [];
  let dayNames: string[] = [];
  let week = new Week();
  let showDays = innerWidth >= 600;

  function getHours(logs: Log[]) {
    let hours = { total: 0, days: [0, 0, 0, 0, 0, 0, 0] };
    logs.forEach((log) => {
      hours.total += log.hours;
      let logDate = log.start.toDate();
      if (week.contains(logDate)) hours.days[logDate.getDay()] += log.hours;
    });
    return hours;
  }

  function remove() {
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

  async function refresh() {
    $mt.member.list = await $mt.member.getList();
  }

  async function adjust(member: MemberData & { id: string }, prevHours: number, dayIndex: number) {
    if ($mt.user.id != $mt.team.data.ownerId && $mt.user.id != member.id) return;
    let day = week.day(dayIndex);
    let dayDisplay = day.toLocaleString(undefined, { dateStyle: "long" });
    let hoursPrompt = prompt(`Edit hours for:\n${member.name} - ${dayDisplay}`, `${prevHours ? prevHours : ""}`);
    let newHours = parseInt(hoursPrompt);
    if (!hoursPrompt || isNaN(newHours) || !isFinite(newHours)) return;
    let existingLog = member.logs.find((log) => sameDay(log.start.toDate(), day));
    if (existingLog) existingLog.hours = newHours;
    else member.logs.push({ hours: newHours, start: Timestamp.fromDate(day) });
    $mt.member = $mt.member.update({ logs: member.logs }, member.id);
  }

  $: {
    $mt.member = $mt.member;
    dayNames = [...Array(7).keys()].map((dayIndex) => {
      let day = new Date(week.sunday.getFullYear(), week.sunday.getMonth(), week.sunday.getDate() + dayIndex);
      return day.toLocaleString(undefined, { month: "numeric", day: "numeric" });
    });
  }
</script>

<details open={$mt.user.id == $mt.team.data.ownerId}>
  <summary>Members</summary>
  <p><button on:click={() => (showDays = !showDays)}>{showDays ? "Hide" : "Show"} days</button></p>
  {#if showDays}
    <p>Week of {week.sunday.toLocaleString(undefined, { dateStyle: "long" })}</p>
  {/if}
  <p>
    <button on:click={refresh} title="Refresh">↻</button>
    {#if showDays}
      <button on:click={() => (week = new Week())}>This week</button>
      <button on:click={() => (week = week.previous)} title="Previous week">&lt;</button>
      <button on:click={() => (week = week.next)} title="Next week">&gt;</button>
    {/if}
  </p>
  <div class="table-wrap">
    <table>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#if showDays}
          {#each dayNames as day}
            <th class="day-col">{day}</th>
          {/each}
        {/if}
      </tr>
      {#if $mt.member?.list}
        {#each $mt.member.list as member (member.id)}
          {@const hoursData = getHours(member.logs)}
          <tr>
            <td class="name-col">
              {#if $mt.user.id == $mt.team.data.ownerId && member.id != $mt.team.data.ownerId}
                <label>
                  <input type="checkbox" bind:group={selectedMembers} value={member.id} />
                  {member.name}
                </label>
              {:else}
                {member.name}
              {/if}
            </td>
            <td class="hours-col">{hoursData.total.toFixed(1)}</td>
            {#if showDays}
              {#each hoursData.days as hours, i}
                <td
                  class="day-col"
                  class:editable={$mt.user.id == $mt.team.data.ownerId || $mt.user.id == member.id}
                  on:click={() => adjust(member, hours, i)}
                >
                  {#if hours}
                    {hours.toFixed(1)}
                  {/if}
                </td>
              {/each}
            {/if}
          </tr>
        {/each}
      {/if}
    </table>
  </div>
  {#if $mt.user.id == $mt.team.data.ownerId}
    <p><button class="red" on:click={remove} disabled={!selectedMembers.length}>Remove</button></p>
  {/if}
</details>
