<script lang="ts">
  export let action: "open" | "invite" | "delete";
  export let label: string;
  export let icon: any;
  export let href: string | undefined = undefined;
  export let onclick: ((e: MouseEvent) => void) | undefined = undefined;
</script>

{#if href}
  <a {href} class="action-btn {action}-action" title={label}>
    <svelte:component this={icon} size={16} />
    <span class="action-label">{label}</span>
  </a>
{:else}
  <button {onclick} class="action-btn {action}-action" title={label}>
    <svelte:component this={icon} size={16} />
    <span class="action-label">{label}</span>
  </button>
{/if}

<style></style>
