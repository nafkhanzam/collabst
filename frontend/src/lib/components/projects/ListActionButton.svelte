<script lang="ts">
  export let action: "open" | "invite" | "delete";
  export let icon: any;
  export let title: string;
  export let href: string | undefined = undefined;
  export let onclick: ((e: MouseEvent) => void) | undefined = undefined;
</script>

{#if href}
  <a {href} class="action-btn {action}-action" {title}>
    <svelte:component this={icon} size={19} />
  </a>
{:else}
  <button {onclick} class="action-btn {action}-action" {title}>
    <svelte:component this={icon} size={19} />
  </button>
{/if}

<style></style>
