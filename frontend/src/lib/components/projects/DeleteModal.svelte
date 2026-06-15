<script lang="ts">
  import Trash2 from "@lucide/svelte/icons/trash-2";

  let { show = $bindable(false), onConfirm } = $props<{
    show?: boolean;
    onConfirm: () => void;
  }>();

  // Handle Escape key
  $effect(() => {
    if (!show) return;
    const handleKeydown = (e: KeyboardEvent) => {
      if (e.key === "Escape") {
        closeModal();
      }
    };
    window.addEventListener("keydown", handleKeydown);
    return () => window.removeEventListener("keydown", handleKeydown);
  });

  function closeModal() {
    show = false;
  }

  function handleConfirm() {
    onConfirm();
  }
</script>

{#if show}
  <div
    class="modal"
    onclick={closeModal}
    onkeydown={(e) => e.key === "Escape" && closeModal()}
    role="presentation"
  >
    <div
      class="modal-content"
      onclick={(e) => e.stopPropagation()}
      onkeydown={(e) => e.stopPropagation()}
      role="dialog"
      tabindex="-1"
    >
      <h2>Delete Project</h2>
      <p class="delete-message">
        Are you sure you want to delete this project?<br />This action cannot
        be undone and all files will be permanently deleted.
      </p>
      <div class="modal-actions">
        <button type="button" onclick={closeModal} class="cancel-btn">
          Cancel
        </button>
        <button type="button" onclick={handleConfirm} class="delete-btn">
          Delete Project
          <Trash2 size={18} />
        </button>
      </div>
    </div>
  </div>
{/if}

<style></style>
