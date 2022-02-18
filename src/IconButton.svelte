<script lang="ts">
  import Icon from "./Icon.svelte";

  export let icon = "";
  export let text = "";
</script>

<button class="icon-button" on:click><Icon bind:name={icon} />{text}</button>
