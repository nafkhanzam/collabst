<script lang="ts">
  import Search from "@lucide/svelte/icons/search";
  import X from "@lucide/svelte/icons/x";

  let { value = $bindable("") } = $props();
</script>

<label class="search-bar">
  <Search size={16} class="search-icon" />
  <input
    type="text"
    bind:value
    placeholder="Search projects..."
    class="search-input"
  />
  {#if value}
    <button
      onclick={() => (value = "")}
      class="search-clear"
      aria-label="Clear search"
    >
      <X size={16} />
    </button>
  {/if}
</label>

<style></style>
