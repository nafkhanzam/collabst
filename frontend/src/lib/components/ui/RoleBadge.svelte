<script lang="ts">
  import type { CollaboratorRole, ProjectRole } from "$lib/types";

  const VALID_ROLES = new Set(["owner", "admin", "writer", "commentor", "reader"]);

  let {
    role,
    uppercase = false,
    size = "md",
  } = $props<{
    role: CollaboratorRole | ProjectRole | string;
    uppercase?: boolean;
    size?: "sm" | "md";
  }>();

  const normalizedRole = $derived((role ?? "owner").toLowerCase());
  const safeRole = $derived(VALID_ROLES.has(normalizedRole) ? normalizedRole : "owner");
  const displayLabel = $derived(uppercase ? safeRole.toUpperCase() : safeRole);
</script>

<span class="role-badge role-{safeRole} role-{size}">
  {displayLabel}
</span>

<style></style>
