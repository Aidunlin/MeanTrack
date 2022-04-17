<script lang="ts">
  import { doc, Timestamp, writeBatch } from "firebase/firestore/lite";
  import { mt } from "./Global.svelte";

  let selectedUnverifieds: string[] = [];

  function verify() {
    let verifyBatch = writeBatch($mt.firestore);
    $mt.unverified.list = $mt.unverified.list.filter((member) => {
      let shouldVerify = selectedUnverifieds.includes(member.id);
      if (shouldVerify) {
        let newMember = {
          lastAction: Timestamp.now(),
          logs: [],
          name: member.name,
          tracking: false,
        };
        $mt.member.list.push({ ...newMember, id: member.id });
        verifyBatch.delete(doc($mt.unverified.collection, member.id));
        verifyBatch.set(doc($mt.member.collection, member.id), newMember);
      }
      return !shouldVerify;
    });
    verifyBatch.commit().catch(console.error);
    $mt.member.list = $mt.member.list.sort((a, b) => (a.name > b.name ? 1 : -1));
    selectedUnverifieds = [];
  }

  async function refresh() {
    $mt.unverified.list = await $mt.unverified.getList();
  }
</script>

<details open={!!$mt.unverified.list.length}>
  <summary>Unverified</summary>
  <p><button on:click={refresh} title="Refresh">↻</button></p>
  {#each $mt.unverified.list as unverified (unverified.id)}
    <label>
      <input type="checkbox" bind:group={selectedUnverifieds} name="unverifieds" value={unverified.id} />
      {unverified.name}
    </label>
  {/each}
  {#if $mt.unverified.list.length}
    <p><button on:click={verify} disabled={!selectedUnverifieds.length}>Verify</button></p>
  {/if}
</details>
