<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query, setDoc } from "firebase/firestore";
  import { MemberData, mt } from "./Global.svelte";

  function verifyMembers() {
    if (!confirm("Are you sure?")) return;
    let selectedUnverifieds = $mt.cachedUnverifieds
      .filter((member) => $mt.selectedUnverifieds.includes(member.id))
      .map((unverified): MemberData => {
        return {
          id: unverified.id,
          logs: [],
          name: unverified.name,
          tracking: false,
        };
      });
    $mt.cachedMembers.push(...selectedUnverifieds);
    $mt.cachedMembers = $mt.cachedMembers.sort((a, b) => (a.name > b.name ? 1 : -1));
    $mt.cachedUnverifieds = $mt.cachedUnverifieds.filter((member) => {
      return !$mt.selectedUnverifieds.includes(member.id);
    });
    $mt.selectedUnverifieds.forEach(async (id) => {
      await deleteDoc(doc($mt.unverified.collection, id)).catch(console.error);
      let d = doc($mt.member.collection, id);
      await setDoc(
        d,
        selectedUnverifieds.find((member) => member.id == id)
      ).catch(console.error);
    });
    $mt.selectedUnverifieds = [];
  }

  function refreshUnverifieds() {
    if (!$mt.unverified.collection) return;
    let unverifiedQuery = query($mt.unverified.collection, orderBy("name"));
    getDocs(unverifiedQuery).then((querySnapshot) => {
      $mt.cachedUnverifieds = [];
      $mt.selectedUnverifieds = [];
      querySnapshot.forEach((unverifiedDoc) => {
        $mt.cachedUnverifieds = [...$mt.cachedUnverifieds, unverifiedDoc.data()];
      });
    });
  }

  refreshUnverifieds();
</script>

<h3>Unverified</h3>
<p>
  <button on:click={refreshUnverifieds}>Refresh</button>
  {#if $mt.cachedUnverifieds.length}
    |
    <button class="green" on:click={verifyMembers} disabled={!$mt.selectedUnverifieds.length}>Verify</button>
  {/if}
</p>
{#each $mt.cachedUnverifieds as member (member.id)}
  <p>
    <label>
      <input type="checkbox" bind:group={$mt.selectedUnverifieds} name="unverifieds" value={member.id} />
      {member.name}
    </label>
  </p>
{/each}
