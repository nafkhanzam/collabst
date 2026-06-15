<script lang="ts">
  let { show = $bindable(false), onSubmit } = $props<{
    show?: boolean;
    onSubmit: (name: string, description: string) => void;
  }>();

  let newProjectName = $state("");
  let newProjectDescription = $state("");
  let projectNameInput = $state<HTMLInputElement | undefined>();

  // Focus input when modal opens
  $effect(() => {
    if (show && projectNameInput) {
      setTimeout(() => projectNameInput?.focus(), 0);
    }
  });

  function handleSubmit(e: Event) {
    e.preventDefault();
    onSubmit(newProjectName, newProjectDescription);
    newProjectName = "";
    newProjectDescription = "";
  }

  function closeModal() {
    show = false;
    newProjectName = "";
    newProjectDescription = "";
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
      <h2>Create New Project</h2>
      <form onsubmit={handleSubmit}>
        <div class="field">
          <label for="project-name">Project Name</label>
          <input
            id="project-name"
            bind:this={projectNameInput}
            type="text"
            bind:value={newProjectName}
            required
            placeholder="My Awesome Project"
          />
        </div>
        <div class="field">
          <label for="project-description">Description</label>
          <textarea
            id="project-description"
            bind:value={newProjectDescription}
            placeholder="A brief description of your project"
          ></textarea>
        </div>
        <div class="modal-actions">
          <button type="button" onclick={closeModal} class="cancel-btn">
            Cancel
          </button>
          <button type="submit" class="submit-btn"> Create Project </button>
        </div>
      </form>
    </div>
  </div>
{/if}

<style></style>
