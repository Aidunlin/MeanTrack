<script lang="ts">
  import { deleteDoc, doc, Timestamp } from "firebase/firestore/lite";
  import { isOwner, Log, MemberData, mt, sameDay, Week, withinCutoff } from "./Global.svelte";
  import Dialog from "./Dialog.svelte";

  let dayNames: string[] = [];
  let week = new Week();

  let editMemberData: {
    member: MemberData & { id: string };
    newName: string;
    show: boolean;
  } = { member: null, newName: "", show: false };

  let adjustData: {
    dayDisplay: string;
    dayIndex: number;
    member: MemberData & { id: string };
    newHours: number;
    prevHours: number;
    show: boolean;
  } = { dayDisplay: "", dayIndex: 0, member: null, newHours: 0, prevHours: 0, show: false };

  function getHours(logs: Log[]) {
    let hours = { total: 0, days: [0, 0, 0, 0, 0, 0, 0] };
    logs.forEach((log) => {
      if (withinCutoff($mt, log)) hours.total += log.hours;
      let logDate = log.start.toDate();
      if (week.contains(logDate)) hours.days[logDate.getDay()] += log.hours;
    });
    return hours;
  }

  async function refresh() {
    $mt.member.list = await $mt.member.getList();
  }

  function showEditMember(member: MemberData & { id: string }) {
    if (!isOwner($mt)) return;
    editMemberData = { member, newName: member.name, show: true };
  }

  function editMember() {
    $mt.member = $mt.member.update({ name: editMemberData.newName }, editMemberData.member.id);
  }

  function removeMember() {
    if (!confirm("Are you sure?")) return;
    $mt.member.list = $mt.member.list.filter((member) => {
      let shouldRemove = member.id == editMemberData.member.id && member.id != $mt.team.data.ownerId;
      if (shouldRemove) deleteDoc(doc($mt.member.collection, member.id));
      return !shouldRemove;
    });
    editMemberData = { member: null, newName: "", show: false };
  }

  function showAdjust(member: MemberData & { id: string }, prevHours: number, dayIndex: number) {
    if (!isOwner($mt) && $mt.user.id != member.id) return;
    let fixedPrevHours = parseFloat(prevHours.toFixed(1));
    adjustData = {
      dayDisplay: week.day(dayIndex).toLocaleString(undefined, { dateStyle: "medium" }),
      dayIndex,
      member,
      newHours: fixedPrevHours,
      prevHours: fixedPrevHours,
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
    adjustData.prevHours = adjustData.newHours;
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
  <p>Edit {editMemberData.member.name}</p>
  <label>Name<input type="text" bind:value={editMemberData.newName} /></label>
  <span slot="dialog-button">
    <button on:click={editMember} disabled={editMemberData.newName == editMemberData.member.name}>Apply</button>
    <button class="red" on:click={removeMember} disabled={editMemberData.member.id == $mt.team.data.ownerId}>
      Remove
    </button>
  </span>
</Dialog>

<Dialog bind:open={adjustData.show}>
  <p>Edit {adjustData.member.name}</p>
  <label>Hours on {adjustData.dayDisplay}<input type="number" step="0.1" bind:value={adjustData.newHours} /></label>
  <button slot="dialog-button" on:click={adjust} disabled={adjustData.newHours == adjustData.prevHours}>Apply</button>
</Dialog>

<details open={isOwner($mt)}>
  <summary>Members</summary>
  <p>Week of {week.sunday.toLocaleString(undefined, { dateStyle: "long" })}</p>
  <div class="buttons">
    <button on:click={refresh} title="Refresh">&circlearrowright;</button>
    <button on:click={() => (week = week.previous)} title="Previous week">&lt;</button>
    <button on:click={() => (week = week.next)} title="Next week">&gt;</button>
    <button on:click={() => (week = new Week())}>This week</button>
    <button on:click={() => (week = new Week($mt.team.data.cutoffBegin.toDate()))}>Cutoff begin</button>
    <button on:click={() => (week = new Week($mt.team.data.cutoffEnd.toDate()))}>Cutoff end</button>
  </div>
  <div class="table-wrap">
    <table>
      <tr>
        <th class="name-col">Name</th>
        <th class="hours-col">Hours</th>
        {#each dayNames as day}
          <th class="day-col">{day}</th>
        {/each}
      </tr>
      {#each $mt.member.list as member (member.id)}
        {@const hoursData = getHours(member.logs)}
        <tr>
          <td class="name-col" class:editable={isOwner($mt)} on:click={() => showEditMember(member)}>
            {member.name}
          </td>
          <td class="hours-col">{hoursData.total.toFixed(1)}</td>
          {#each hoursData.days as hours, i}
            <td
              class="day-col"
              class:editable={isOwner($mt) || $mt.user.id == member.id}
              on:click={() => showAdjust(member, hours, i)}
            >
              {#if hours}
                {hours.toFixed(1)}
              {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </table>
  </div>
</details>
