<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query, setDoc, Timestamp } from "firebase/firestore/lite";
  import { mt } from "./Global.svelte";

  let selectedUnverifiedIds: string[] = [];

  function verifyMembers() {
    if (!confirm("Are you sure?")) return;
    $mt.cachedUnverifieds = $mt.cachedUnverifieds.filter((member) => {
      let shouldVerify = selectedUnverifiedIds.includes(member.id);
      if (shouldVerify) {
        let newMember = {
          lastAction: new Timestamp(0, 0),
          logs: [],
          name: member.name,
          tracking: false,
        };
        $mt.cachedMembers.push({ ...newMember, id: member.id });
        Promise.all([
          deleteDoc(doc($mt.unverified.collection, member.id)),
          setDoc(doc($mt.member.collection, member.id), newMember),
        ]).catch(console.error);
      }
      return !shouldVerify;
    });
    $mt.cachedMembers = $mt.cachedMembers.sort((a, b) => (a.name > b.name ? 1 : -1));
    selectedUnverifiedIds = [];
  }

  function refreshUnverifieds() {
    if (!$mt.unverified.collection) return;
    getDocs(query($mt.unverified.collection, orderBy("name"))).then((querySnapshot) => {
      $mt.cachedUnverifieds = [];
      selectedUnverifiedIds = [];
      querySnapshot.forEach((unverifiedDoc) => {
        if (!$mt.cachedMembers.some((member) => member.id == unverifiedDoc.id)) {
          $mt.cachedUnverifieds = [...$mt.cachedUnverifieds, { ...unverifiedDoc.data(), id: unverifiedDoc.id }];
        }
      });
    });
  }

  refreshUnverifieds();
</script>

<details>
  <summary>Unverified</summary>
  <p><button on:click={refreshUnverifieds} title="Refresh">↻</button></p>
  {#each $mt.cachedUnverifieds as member (member.id)}
    <p>
      <label>
        <input type="checkbox" bind:group={selectedUnverifiedIds} name="unverifieds" value={member.id} />
        {member.name}
      </label>
    </p>
  {/each}
  {#if $mt.cachedUnverifieds.length}
    <p><button class="green" on:click={verifyMembers} disabled={!selectedUnverifiedIds.length}>Verify</button></p>
  {/if}
</details>
