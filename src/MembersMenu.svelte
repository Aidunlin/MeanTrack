<script lang="ts">
  import { deleteDoc, doc, Timestamp } from "firebase/firestore/lite";
  import { Log, MemberData, mt, sameDay, Week, withinCutoff } from "./Global.svelte";
  import Dialog from "./Dialog.svelte";

  let dayNames: string[] = [];
  let week = new Week();

  let editMemberData: {
    member: MemberData & { id: string };
    show: boolean;
  } = {
    member: null,
    show: false,
  };

  let adjustData: {
    dayDisplay: string;
    dayIndex: number;
    member: MemberData & { id: string };
    newHours: number;
    prevHours: number;
    show: boolean;
  } = {
    dayDisplay: "",
    dayIndex: 0,
    member: null,
    newHours: 0,
    prevHours: 0,
    show: false,
  };

  function getHours(logs: Log[]) {
    let hours = { total: 0, days: [0, 0, 0, 0, 0, 0, 0] };
    logs.forEach((log) => {
      if (withinCutoff($mt, log)) {
        hours.total += log.hours;
      }
      let logDate = log.start.toDate();
      if (week.contains(logDate)) hours.days[logDate.getDay()] += log.hours;
    });
    return hours;
  }

  async function refresh() {
    $mt.member.list = await $mt.member.getList();
  }

  function showEditMember(member: MemberData & { id: string }) {
    if ($mt.user.id != $mt.team.data.ownerId || member.id == $mt.team.data.ownerId) return;
    editMemberData = {
      member,
      show: true,
    };
  }

  function removeMember() {
    if (!confirm("Are you sure?")) return;
    $mt.member.list = $mt.member.list.filter((member) => {
      let shouldRemove = member.id == editMemberData.member.id && member.id != $mt.team.data.ownerId;
      if (shouldRemove) deleteDoc(doc($mt.member.collection, member.id));
      return !shouldRemove;
    });
    editMemberData = {
      member: null,
      show: false,
    };
  }

  function showAdjust(member: MemberData & { id: string }, prevHours: number, dayIndex: number) {
    if ($mt.user.id != $mt.team.data.ownerId && $mt.user.id != member.id) return;
    adjustData = {
      dayDisplay: week.day(dayIndex).toLocaleString(undefined, { dateStyle: "medium" }),
      dayIndex,
      member,
      newHours: prevHours,
      prevHours,
      show: true,
    };
  }

  async function adjust() {
    let day = week.day(adjustData.dayIndex);
    if (isNaN(adjustData.newHours) || !isFinite(adjustData.newHours)) return;
    let existingLog = adjustData.member.logs.find((log) => sameDay(log.start.toDate(), day));
    if (existingLog) existingLog.hours = adjustData.newHours;
    else adjustData.member.logs.push({ hours: adjustData.newHours, start: Timestamp.fromDate(day) });
    $mt.member = $mt.member.update({ logs: adjustData.member.logs }, adjustData.member.id);
  }

  $: {
    $mt.member = $mt.member;
    dayNames = [...Array(7).keys()].map((dayIndex) => {
      let day = new Date(week.sunday.getFullYear(), week.sunday.getMonth(), week.sunday.getDate() + dayIndex);
      return day.toLocaleString(undefined, { month: "numeric", day: "numeric" });
    });
  }
</script>

<Dialog bind:open={editMemberData.show}>
  <p>{editMemberData.member.name}</p>
  <p><button class="red" on:click={removeMember}>Remove</button></p>
</Dialog>

<Dialog bind:open={adjustData.show}>
  <p>{adjustData.member.name} - {adjustData.dayDisplay}</p>
  <label>Edit hours:<br /><input type="number" bind:value={adjustData.newHours} /></label>
  <button slot="buttons" on:click={adjust} disabled={adjustData.newHours == adjustData.prevHours}>Apply</button>
</Dialog>

<details open={$mt.user.id == $mt.team.data.ownerId}>
  <summary>Members</summary>
  <p>Week of {week.sunday.toLocaleString(undefined, { dateStyle: "long" })}</p>
  <p class="overflow-wide">
    <button on:click={refresh} title="Refresh">&circlearrowright;</button>
    <button on:click={() => (week = week.previous)} title="Previous week">&lt;</button>
    <button on:click={() => (week = week.next)} title="Next week">&gt;</button>
    <button on:click={() => (week = new Week())}>This week</button>
    <button on:click={() => (week = new Week($mt.team.data.cutoffBegin.toDate()))}>Cutoff begin</button>
    <button on:click={() => (week = new Week($mt.team.data.cutoffEnd.toDate()))}>Cutoff end</button>
  </p>
  <div class="overflow-wide">
    <table>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#each dayNames as day}
          <th class="day-col">{day}</th>
        {/each}
      </tr>
      {#if $mt.member?.list}
        {#each $mt.member.list as member (member.id)}
          {@const hoursData = getHours(member.logs)}
          <tr>
            <td
              class="name-col"
              class:editable={$mt.user.id == $mt.team.data.ownerId && member.id != $mt.team.data.ownerId}
              on:click={() => showEditMember(member)}
            >
              {member.name}
            </td>
            <td class="hours-col">{hoursData.total.toFixed(1)}</td>
            {#each hoursData.days as hours, i}
              <td
                class="day-col"
                class:editable={$mt.user.id == $mt.team.data.ownerId || $mt.user.id == member.id}
                on:click={() => showAdjust(member, hours, i)}
              >
                {#if hours}
                  {hours.toFixed(1)}
                {/if}
              </td>
            {/each}
          </tr>
        {/each}
      {/if}
    </table>
  </div>
</details>
