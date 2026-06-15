<script lang="ts">
  import type { Component } from "svelte";

  interface ToolButtonProps {
    icon: Component;
    onclick?: () => void;
    disabled?: boolean;
    active?: boolean;
    class?: string;
    position?: "standalone" | "first" | "middle" | "last";
    strokeWidth?: number;
  }

  let {
    icon,
    onclick,
    disabled = false,
    active = false,
    class: className = "",
    position = "standalone",
    strokeWidth = 2,
  }: ToolButtonProps = $props();

  const Icon = $derived(icon);
</script>

<button
  type="button"
  {disabled}
  class="tool-btn tool-btn-{position} {active
    ? 'tool-btn-active'
    : ''} {className}"
  {onclick}
>
  <Icon size={16} {strokeWidth} />
</button>

<style></style>
