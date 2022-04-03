<script lang="ts">
  import { doc, Timestamp, writeBatch } from "firebase/firestore/lite";
  import { mt } from "./Global.svelte";

  let selectedUnverifiedIds: string[] = [];

  function verifyMembers() {
    if (!confirm("Are you sure?")) return;
    let verifyBatch = writeBatch($mt.firestore);
    $mt.unverified.list = $mt.unverified.list.filter((member) => {
      let shouldVerify = selectedUnverifiedIds.includes(member.id);
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
    selectedUnverifiedIds = [];
  }

  async function refreshUnverifieds() {
    $mt.unverified.list = await $mt.unverified.getList();
  }
</script>

<details>
  <summary>Unverified</summary>
  <p><button on:click={refreshUnverifieds} title="Refresh">↻</button></p>
  {#each $mt.unverified.list as unverified (unverified.id)}
    <p>
      <label>
        <input type="checkbox" bind:group={selectedUnverifiedIds} name="unverifieds" value={unverified.id} />
        {unverified.name}
      </label>
    </p>
  {/each}
  {#if $mt.unverified.list.length}
    <p><button class="green" on:click={verifyMembers} disabled={!selectedUnverifiedIds.length}>Verify</button></p>
  {/if}
</details>
