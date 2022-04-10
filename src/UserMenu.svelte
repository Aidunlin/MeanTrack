<script lang="ts">
  import { Timestamp } from "firebase/firestore/lite";
  import { hoursBetween, mt, sameDay } from "./Global.svelte";

  enum View {
    Default,
    Edit,
  }
  let viewing = View.Default;

  let totalHours: number;
  let tempHours: number;
  let trackingDisplay: string;

  function toggleTracking() {
    if ($mt.member.data.tracking) {
      let existingLog = $mt.member.data.logs.find((log) => {
        return sameDay(log.start.toDate(), $mt.member.data.lastAction.toDate());
      });
      let newHours = hoursBetween(Timestamp.now(), $mt.member.data.lastAction);
      if (existingLog) existingLog.hours += newHours;
      else $mt.member.data.logs.push({ hours: newHours, start: $mt.member.data.lastAction });
    }
    $mt.member = $mt.member.update({
      lastAction: Timestamp.now(),
      logs: $mt.member.data.logs,
      tracking: !$mt.member.data.tracking,
    });
  }

  function getHours() {
    let hours = 0;
    $mt.member.data.logs.forEach((log) => (hours += log.hours));
    return hours;
  }

  let nameEditValue = $mt.member.data.name;
  function editName() {
    $mt.member = $mt.member.update({ name: nameEditValue });
  }

  $: {
    totalHours = getHours();
    tempHours = hoursBetween(Timestamp.now(), $mt.member.data.lastAction);
    trackingDisplay = $mt.member.data.tracking ? "Stop" : "Start";
  }
</script>

<details open>
  <summary>{$mt.member.data.name}</summary>
  {#if viewing == View.Default}
    {#if $mt.member.data.logs}
      <p>
        Hours: {totalHours.toFixed(1)}
        {#if $mt.member.data.tracking}
          + {tempHours.toFixed(1)}
        {/if}
      </p>
    {/if}
    <p>
      <button on:click={toggleTracking}>{trackingDisplay} tracking</button>
      <button on:click={() => (viewing = View.Edit)}>Edit</button>
    </p>
  {:else if viewing == View.Edit}
    <p>
      <label>Edit name:<br /><input bind:value={nameEditValue} /></label>
      <button on:click={editName} disabled={nameEditValue == $mt.member.data.name}>Update</button>
    </p>
    <p>
      <button on:click={() => (viewing = View.Default)}>Done</button>
    </p>
  {/if}
</details>
