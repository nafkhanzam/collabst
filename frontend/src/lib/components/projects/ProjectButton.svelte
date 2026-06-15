<script lang="ts">
  import Plus from "@lucide/svelte/icons/plus";

  export let onclick: () => void;
</script>

<button {onclick} class="create-btn">
  <span
    class="create-btn-content"
    style="display: flex; align-items: center; gap: 0.3rem;"
  >
    <Plus size={18} strokeWidth={3} style="vertical-align: middle;" />
    New Project
  </span>
</button>

<style></style>
