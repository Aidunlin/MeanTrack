<script lang="ts">
  import { deleteDoc, doc, getDocs, orderBy, query, setDoc } from "firebase/firestore";
  import { MemberData, mt, memberManagement } from "./Global.svelte";

  function verifyMembers() {
    if (!confirm("Are you sure?")) return;
    let selectedMembers = $memberManagement.unverifiedMembers
      .filter((member) => $memberManagement.selectedUnverifiedIds.includes(member.id))
      .map((unverified): MemberData => {
        return {
          id: unverified.id,
          logs: [],
          name: unverified.name,
          tracking: false,
        };
      });
    $memberManagement.verifiedMembers.push(...selectedMembers);
    $memberManagement.verifiedMembers = $memberManagement.verifiedMembers.sort((a, b) => (a.name > b.name ? 1 : -1));
    $memberManagement.unverifiedMembers = $memberManagement.unverifiedMembers.filter((member) => {
      return !$memberManagement.selectedUnverifiedIds.includes(member.id);
    });
    $memberManagement.selectedUnverifiedIds.forEach(async (id) => {
      await deleteDoc(doc($mt.unverified.collection, id)).catch(console.error);
      let d = doc($mt.member.collection, id);
      await setDoc(
        d,
        selectedMembers.find((member) => member.id == id)
      ).catch(console.error);
    });
    $memberManagement.selectedUnverifiedIds = [];
  }

  function refreshUnverifieds() {
    let unverifiedQuery = query($mt.unverified.collection, orderBy("name"));
    getDocs(unverifiedQuery).then((querySnapshot) => {
      $memberManagement.unverifiedMembers = [];
      $memberManagement.selectedUnverifiedIds = [];
      querySnapshot.forEach((unverifiedDoc) => {
        $memberManagement.unverifiedMembers = [...$memberManagement.unverifiedMembers, unverifiedDoc.data()];
      });
    });
  }

  refreshUnverifieds();
</script>

<h3>Unverified</h3>
<p>
  <button on:click={refreshUnverifieds}>Refresh</button>
  {#if $memberManagement.unverifiedMembers.length}
    |
    <button class="green" on:click={verifyMembers} disabled={!$memberManagement.selectedUnverifiedIds.length}>
      Verify
    </button>
  {/if}
</p>
{#each $memberManagement.unverifiedMembers as member (member.id)}
  <p>
    <label>
      <input
        type="checkbox"
        bind:group={$memberManagement.selectedUnverifiedIds}
        name="unverified-members"
        value={member.id}
      />
      {member.name}
    </label>
  </p>
{/each}
