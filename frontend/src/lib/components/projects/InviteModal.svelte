<script lang="ts">
  import SendHorizontal from "@lucide/svelte/icons/send-horizontal";
  import type { CollaboratorRole } from "$lib/types";

  let { show = $bindable(false), onSubmit } = $props<{
    show?: boolean;
    onSubmit: (email: string, role: CollaboratorRole) => void;
  }>();

  let inviteEmail = $state("");
  let inviteRole = $state<CollaboratorRole>("writer");
  let inviteEmailInput = $state<HTMLInputElement | undefined>();

  // Focus input when modal opens
  $effect(() => {
    if (show && inviteEmailInput) {
      setTimeout(() => inviteEmailInput?.focus(), 0);
    }
  });

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

  function handleSubmit(e: Event) {
    e.preventDefault();
    onSubmit(inviteEmail, inviteRole);
    inviteEmail = "";
    inviteRole = "writer";
  }

  function closeModal() {
    show = false;
    inviteEmail = "";
    inviteRole = "writer";
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
      <h2>Invite Collaborator</h2>
      <form onsubmit={handleSubmit}>
        <div class="field">
          <label for="invite-email">Email Address</label>
          <input
            id="invite-email"
            bind:this={inviteEmailInput}
            type="email"
            bind:value={inviteEmail}
            required
            placeholder="collaborator@example.com"
          />
        </div>
        <div class="field">
          <label for="invite-role">Role</label>
          <select id="invite-role" bind:value={inviteRole}>
            <option value="reader">Reader - Can only view</option>
            <option value="commentor">Commentor - Can comment</option>
            <option value="writer">Writer - Can edit files</option>
            <option value="admin">Admin - Can manage sharing and members</option>
          </select>
        </div>
        <div class="modal-actions">
          <button type="button" onclick={closeModal} class="cancel-btn">
            Cancel
          </button>
          <button type="submit" class="submit-btn">
            Send Invitation
            <SendHorizontal size={16} />
          </button>
        </div>
      </form>
    </div>
  </div>
{/if}

<style></style>
