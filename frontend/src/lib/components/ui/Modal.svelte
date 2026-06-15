<script lang="ts">
  import type { Snippet } from "svelte";
  import X from "@lucide/svelte/icons/x";
  import IconButton from "./IconButton.svelte";

  interface ModalProps {
    open?: boolean;
    title?: string;
    size?: "sm" | "md" | "lg" | "xl" | "full";
    hideCloseButton?: boolean;
    onClose?: () => void;
    children?: Snippet;
    footer?: Snippet;
  }

  let {
    open = $bindable(false),
    title = "",
    size = "md",
    hideCloseButton = false,
    onClose,
    children,
    footer,
  }: ModalProps = $props();

  // Handle Escape key with window-level event listener
  $effect(() => {
    if (!open) return;

    const handleKeydown = (e: KeyboardEvent) => {
      if (e.key === "Escape") {
        handleClose();
      }
    };

    window.addEventListener("keydown", handleKeydown);
    return () => window.removeEventListener("keydown", handleKeydown);
  });

  function handleClose() {
    open = false;
    onClose?.();
  }

  function handleBackdropClick(e: MouseEvent) {
    if (e.target === e.currentTarget) {
      handleClose();
    }
  }
</script>

{#if open}
  <div class="modal-backdrop" onclick={handleBackdropClick} role="presentation">
    <div
      class="modal modal-{size}"
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      {#if title || !hideCloseButton}
        <div class="modal-header">
          {#if title}
            <h2 id="modal-title" class="modal-title">{title}</h2>
          {/if}
          {#if !hideCloseButton}
            <IconButton
              icon={X}
              onclick={handleClose}
              title="Close"
              class="modal-close"
            />
          {/if}
        </div>
      {/if}

      <div class="modal-body">
        {@render children?.()}
      </div>

      {#if footer}
        <div class="modal-footer">
          {@render footer()}
        </div>
      {/if}
    </div>
  </div>
{/if}

<style></style>
