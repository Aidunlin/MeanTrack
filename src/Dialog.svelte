<script lang="ts">
  export let open = false;
</script>

{#if open}
  <div class="dialog" on:mousedown={() => (open = false)}>
    <div class="dialog-content" on:mousedown={(e) => e.stopPropagation()}>
      <slot />
      <div class="buttons">
        <slot name="dialog-button" />
        <button on:click={() => (open = false)}>Close</button>
      </div>
    </div>
  </div>
{/if}
