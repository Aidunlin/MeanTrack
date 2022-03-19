<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query, setDoc } from "firebase/firestore";
  import { MemberData, mt, memberManagement } from "./Global.svelte";

  function verifyMembers() {
    if (!confirm("Are you sure?")) return;
    let selectedUnverifieds = $memberManagement.unverifieds
      .filter((member) => $memberManagement.selectedUnverifieds.includes(member.id))
      .map((unverified): MemberData => {
        return {
          id: unverified.id,
          logs: [],
          name: unverified.name,
          tracking: false,
        };
      });
    $memberManagement.members.push(...selectedUnverifieds);
    $memberManagement.members = $memberManagement.members.sort((a, b) => (a.name > b.name ? 1 : -1));
    $memberManagement.unverifieds = $memberManagement.unverifieds.filter((member) => {
      return !$memberManagement.selectedUnverifieds.includes(member.id);
    });
    $memberManagement.selectedUnverifieds.forEach(async (id) => {
      await deleteDoc(doc($mt.unverified.collection, id)).catch(console.error);
      let d = doc($mt.member.collection, id);
      await setDoc(
        d,
        selectedUnverifieds.find((member) => member.id == id)
      ).catch(console.error);
    });
    $memberManagement.selectedUnverifieds = [];
  }

  function refreshUnverifieds() {
    let unverifiedQuery = query($mt.unverified.collection, orderBy("name"));
    getDocs(unverifiedQuery).then((querySnapshot) => {
      $memberManagement.unverifieds = [];
      $memberManagement.selectedUnverifieds = [];
      querySnapshot.forEach((unverifiedDoc) => {
        $memberManagement.unverifieds = [...$memberManagement.unverifieds, unverifiedDoc.data()];
      });
    });
  }

  refreshUnverifieds();
</script>

<h3>Unverified</h3>
<p>
  <button on:click={refreshUnverifieds}>Refresh</button>
  {#if $memberManagement.unverifieds.length}
    |
    <button class="green" on:click={verifyMembers} disabled={!$memberManagement.selectedUnverifieds.length}>
      Verify
    </button>
  {/if}
</p>
{#each $memberManagement.unverifieds as member (member.id)}
  <p>
    <label>
      <input
        type="checkbox"
        bind:group={$memberManagement.selectedUnverifieds}
        name="unverifieds"
        value={member.id}
      />
      {member.name}
    </label>
  </p>
{/each}
