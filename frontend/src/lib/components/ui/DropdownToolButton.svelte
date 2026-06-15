<script lang="ts">
  import type { Component } from "svelte";

  interface DropdownItem {
    label: string;
    icon?: Component;
    onclick: () => void;
    separator?: boolean; // Show separator AFTER this item
  }

  interface DropdownToolButtonProps {
    icon?: Component | string; // Can be Component or text like "50%"
    items: DropdownItem[];
    disabled?: boolean;
    active?: boolean;
    class?: string;
    position?: "standalone" | "first" | "middle" | "last";
    strokeWidth?: number;
    buttonWidth?: string; // Custom button width
    buttonBackground?: string; // Custom button background color
    allowIconOverflow?: boolean; // Allow icon to overflow button boundaries
    stick?: "left" | "middle" | "right"; // Stick the dropdown menu to the edge of the button
  }

  let {
    icon,
    items,
    disabled = false,
    active = false,
    class: className = "",
    position = "standalone",
    strokeWidth = 2,
    buttonWidth = "30px",
    buttonBackground = undefined,
    allowIconOverflow = true,
    stick = "right",
  }: DropdownToolButtonProps = $props();

  let isOpen = $state(false);
  let buttonRef: HTMLButtonElement | undefined = $state();

  const iconIsComponent = $derived(
    typeof icon !== "string" && icon !== undefined,
  );
  const iconIsText = $derived(typeof icon === "string");
  const Icon = $derived(iconIsComponent ? (icon as Component) : null);

  function toggleDropdown() {
    isOpen = !isOpen;
  }

  function closeDropdown() {
    isOpen = false;
  }

  function handleItemClick(item: DropdownItem) {
    item.onclick();
    closeDropdown();
  }

  // Close dropdown when clicking outside
  function handleClickOutside(event: MouseEvent) {
    if (buttonRef && !buttonRef.contains(event.target as Node)) {
      closeDropdown();
    }
  }

  $effect(() => {
    if (isOpen) {
      document.addEventListener("click", handleClickOutside);
      return () => {
        document.removeEventListener("click", handleClickOutside);
      };
    }
  });
</script>

<div class="dropdown-container">
  <button
    bind:this={buttonRef}
    type="button"
    {disabled}
    class="tool-btn tool-btn-{position} {active
      ? 'tool-btn-active'
      : ''} {allowIconOverflow ? '' : 'no-icon-overflow'} {className}"
    onclick={toggleDropdown}
    style="{buttonBackground
      ? `background: ${buttonBackground};`
      : ''} {buttonWidth ? `width: ${buttonWidth};` : ''}"
  >
    {#if iconIsComponent && Icon}
      <Icon size={16} {strokeWidth} />
    {:else if iconIsText}
      <span class="button-text">{icon}</span>
    {/if}
  </button>

  {#if isOpen}
    <div
      class="dropdown-menu dropdown-menu-stick-{stick}"
      role="menu"
      tabindex="-1"
    >
      {#each items as item, index}
        <button
          type="button"
          class="dropdown-item"
          onclick={() => handleItemClick(item)}
        >
          {#if item.icon}
            {@const ItemIcon = item.icon}
            <span class="dropdown-item-icon">
              <ItemIcon size={14} strokeWidth={2} />
            </span>
          {/if}
          <span class="dropdown-item-label">{item.label}</span>
        </button>
        {#if item.separator}
          <div class="dropdown-separator"></div>
        {/if}
      {/each}
    </div>
  {/if}
</div>

<style></style>
