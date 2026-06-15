<script lang="ts">
  export interface RightClickMenuItem {
    label: string;
    icon?: any;
    onclick: () => void;
    separator?: boolean; // Add separator after this item
  }

  interface Props {
    show: boolean;
    x: number;
    y: number;
    items: RightClickMenuItem[];
    onClose: () => void;
  }

  let { show = $bindable(), x, y, items, onClose }: Props = $props();

  let menuElement = $state<HTMLDivElement | undefined>();

  function handleClickOutside(event: MouseEvent) {
    if (menuElement && !menuElement.contains(event.target as Node)) {
      onClose();
    }
  }

  function handleContextMenu(event: MouseEvent) {
    event.preventDefault();
  }

  $effect(() => {
    if (show) {
      // Add listeners when menu opens
      setTimeout(() => {
        document.addEventListener("click", handleClickOutside);
        document.addEventListener("contextmenu", handleClickOutside);
      }, 0);

      return () => {
        document.removeEventListener("click", handleClickOutside);
        document.removeEventListener("contextmenu", handleClickOutside);
      };
    }
  });

  function handleItemClick(item: RightClickMenuItem) {
    item.onclick();
    onClose();
  }
</script>

{#if show}
  <div
    bind:this={menuElement}
    class="right-click-menu"
    style="top: {y}px; left: {x}px;"
    oncontextmenu={handleContextMenu}
    role="menu"
    tabindex="-1"
  >
    {#each items as item, i}
      <button
        class="menu-item"
        onclick={() => handleItemClick(item)}
        role="menuitem"
      >
        {#if item.icon}
          {@const Icon = item.icon}
          <span class="menu-icon">
            <Icon size={16} />
          </span>
        {/if}
        <span class="menu-label">{item.label}</span>
      </button>
      {#if item.separator && i < items.length - 1}
        <div class="menu-separator"></div>
      {/if}
    {/each}
  </div>
{/if}

<style></style>
