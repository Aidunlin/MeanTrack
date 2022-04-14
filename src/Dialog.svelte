<script lang="ts">
  export let open = false;
</script>

{#if open}
  <div class="dialog" on:mousedown={() => (open = false)}>
    <div class="dialog-content" on:mousedown={(e) => e.stopPropagation()}>
      <slot />
      <p>
        <slot name="buttons" />
        <button on:click={() => (open = false)}>Close</button>
      </p>
    </div>
  </div>
{/if}
