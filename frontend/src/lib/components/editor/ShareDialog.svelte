<script lang="ts">
  import Copy from "@lucide/svelte/icons/copy";
  import Link2 from "@lucide/svelte/icons/link-2";
  import Trash2 from "@lucide/svelte/icons/trash-2";
  import SendHorizontal from "@lucide/svelte/icons/send-horizontal";
  import Users from "@lucide/svelte/icons/users";
  import X from "@lucide/svelte/icons/x";
  import Tooltip from "$lib/components/ui/Tooltip.svelte";
  import DropdownSettingsButton from "$lib/components/ui/DropdownSettingsButton.svelte";
  import IconButton from "$lib/components/ui/IconButton.svelte";
  import { notifications } from "$lib/stores/notifications";
  import type {
    CollaboratorRole,
    Project,
    ShareLinksSummary,
    SharingOverview,
  } from "$lib/types";
  import { invitationsApi, projectsApi, sharingApi } from "$lib/services/api";

  let {
    show = $bindable(false),
    project,
    onClose,
  } = $props<{
    show?: boolean;
    project: Project;
    onClose?: () => void;
  }>();

  let loading = $state(false);
  let error = $state<string | null>(null);
  let overview = $state<SharingOverview | null>(null);

  let inviteEmail = $state("");
  let inviteRole = $state<CollaboratorRole>("writer");
  let inviteLoading = $state(false);

  type PublicLinkType = "read" | "comment" | "edit";

  const canManage = $derived(
    project.current_user_role === "owner" ||
      project.current_user_role === "admin",
  );
  const publicLinks = $derived<ShareLinksSummary>(
    overview?.public_links ?? { read: null, comment: null, edit: null },
  );
  const canSeeCommentLink = $derived(project.current_user_role !== "reader");
  const canSeeEditLink = $derived(
    project.current_user_role === "owner" ||
      project.current_user_role === "admin" ||
      project.current_user_role === "writer",
  );
  const visibleLinkTypes = $derived([
    { key: "read", label: "Read-only" },
    ...(canSeeCommentLink ? [{ key: "comment", label: "Comment-only" }] : []),
    ...(canSeeEditLink ? [{ key: "edit", label: "Edit" }] : []),
  ] as { key: PublicLinkType; label: string }[]);
  const collaboratorRoleOptions = [
    { value: "reader", label: "Reader" },
    { value: "commentor", label: "Commentor" },
    { value: "writer", label: "Writer" },
    { value: "admin", label: "Admin" },
  ];

  const members = $derived.by(() => {
    if (!overview) return [];
    const ownerId = project.owner?.id;
    const ownerExists =
      !!ownerId &&
      overview.collaborators.some(
        (c) => c.user_id === ownerId || c.role === "owner",
      );

    if (ownerExists || !project.owner) {
      return overview.collaborators;
    }

    return [
      {
        id: 0,
        project_id: project.id,
        user_id: project.owner.id,
        role: "owner" as CollaboratorRole,
        created_at: project.created_at,
        updated_at: project.updated_at,
        user: {
          id: project.owner.id,
          email: project.owner.email,
          display_name: project.owner.display_name,
          is_active: true,
          is_superuser: false,
          created_at: project.created_at,
          updated_at: project.updated_at,
        },
      },
      ...overview.collaborators,
    ];
  });

  async function loadOverview() {
    loading = true;
    error = null;
    try {
      overview = await sharingApi.getOverview(project.id);
    } catch (err: any) {
      error = err?.response?.data?.detail || "Failed to load sharing settings";
    } finally {
      loading = false;
    }
  }

  $effect(() => {
    if (show) {
      loadOverview();
    }
  });

  $effect(() => {
    if (!show) return;
    const onKey = (e: KeyboardEvent) => {
      if (e.key === "Escape") close();
    };
    window.addEventListener("keydown", onKey);
    return () => window.removeEventListener("keydown", onKey);
  });

  function close() {
    show = false;
    onClose?.();
  }

  function getAbsoluteUrl(relativePath: string): string {
    if (typeof window === "undefined") return relativePath;
    return `${window.location.origin}${relativePath}`;
  }

  async function createLink(linkType: "read" | "comment" | "edit") {
    if (!canManage) return;
    await sharingApi.createPublicLink(project.id, linkType);
    await loadOverview();
  }

  async function revokeLink(linkType: "read" | "comment" | "edit") {
    if (!canManage) return;
    await sharingApi.revokePublicLink(project.id, linkType);
    await loadOverview();
  }

  async function copyLink(url: string) {
    try {
      await navigator.clipboard.writeText(getAbsoluteUrl(url));
      notifications.show("Link copied to clipboard", "info", 2000);
    } catch {
      notifications.show("Failed to copy link", "error", 2500);
    }
  }

  async function sendInvitation() {
    if (!canManage) return;
    if (!inviteEmail.trim()) return;
    inviteLoading = true;
    try {
      await invitationsApi.send(project.id, inviteEmail.trim(), inviteRole);
      inviteEmail = "";
      inviteRole = "writer";
      await loadOverview();
    } finally {
      inviteLoading = false;
    }
  }

  async function updateMemberRole(userId: string, role: CollaboratorRole) {
    if (!canManage) return;
    await projectsApi.updateCollaborator(project.id, userId, role);
    await loadOverview();
  }

  async function removeMember(userId: string) {
    if (!canManage) return;
    await projectsApi.removeCollaborator(project.id, userId);
    await loadOverview();
  }

  async function cancelInvitation(invitationId: string) {
    if (!canManage) return;
    await invitationsApi.cancel(project.id, invitationId);
    await loadOverview();
  }
</script>

{#if show}
  <div class="overlay" onclick={close} role="presentation">
    <div
      class="dialog"
      onclick={(e) => e.stopPropagation()}
      onkeydown={(e) => e.stopPropagation()}
      role="dialog"
      aria-modal="true"
      tabindex="-1"
    >
      <header class="dialog-header">
        <h2>Share project</h2>
        <IconButton
          variant="flat"
          icon={X}
          size="md"
          onclick={close}
          ariaLabel="Close share dialog"
        />
      </header>

      {#if loading}
        <div class="state">Loading sharing settings…</div>
      {:else if error}
        <div class="state error">{error}</div>
      {:else if overview}
        <section class="section">
          <h3><Link2 size={16} /> Public link</h3>
          {#each visibleLinkTypes as linkType}
            {@const link = publicLinks[linkType.key]}
            <div class="link-row">
              <div class="link-label">{linkType.label}</div>
              {#if link}
                <input readonly value={getAbsoluteUrl(link.url)} />
                <Tooltip text="Copy link" position="top">
                  <button
                    class="icon-btn"
                    type="button"
                    onclick={() => copyLink(link.url)}
                    aria-label="Copy link"
                  >
                    <Copy size={14} />
                  </button>
                </Tooltip>
                {#if canManage}
                  <button
                    class="danger-btn"
                    type="button"
                    onclick={() => revokeLink(linkType.key)}>Revoke link</button
                  >
                {/if}
              {:else if canManage}
                <button
                  class="secondary-btn"
                  type="button"
                  onclick={() => createLink(linkType.key)}>Create link</button
                >
              {:else}
                <span class="muted">No link</span>
              {/if}
            </div>
          {/each}
        </section>

        {#if canManage}
          <section class="section">
            <h3><SendHorizontal size={16} /> Add collaborators</h3>
            <div class="invite-row">
              <input
                type="email"
                bind:value={inviteEmail}
                placeholder="collaborator@example.com"
                disabled={inviteLoading}
              />
              <div
                class="invite-role-dropdown"
                class:invite-role-dropdown-disabled={inviteLoading}
              >
                <DropdownSettingsButton
                  bind:value={inviteRole}
                  options={collaboratorRoleOptions}
                />
              </div>
              <button
                type="button"
                class="primary-btn"
                disabled={inviteLoading}
                onclick={sendInvitation}
              >
                Invite
              </button>
            </div>
            {#if overview.invitations.length === 0}
              <p class="muted">No pending invitations.</p>
            {:else}
              <div class="pending-list">
                {#each overview.invitations as invitation}
                  <div class="pending-item">
                    <span>{invitation.invitee_email} · {invitation.role}</span>
                    <Tooltip text="Cancel invitation" position="top">
                      <button
                        class="icon-btn"
                        type="button"
                        onclick={() => cancelInvitation(invitation.id)}
                        aria-label="Cancel invitation"
                      >
                        <Trash2 size={14} />
                      </button>
                    </Tooltip>
                  </div>
                {/each}
              </div>
            {/if}
          </section>
        {/if}

        <section class="section">
          <h3><Users size={16} /> Project members</h3>
          {#if members.length === 0}
            <p class="muted">No collaborators yet.</p>
          {:else}
            <div class="member-list">
              {#each members as collaborator}
                {@const isOwner =
                  collaborator.role === "owner" ||
                  collaborator.user_id === project.owner_id}
                <div class="member-row">
                  <div class="member-info">
                    <strong
                      >{collaborator.user?.display_name ||
                        collaborator.user_id}</strong
                    >
                    <span>{collaborator.user?.email}</span>
                  </div>
                  {#if canManage && !isOwner}
                    <div class="member-role-dropdown">
                      <DropdownSettingsButton
                        value={collaborator.role}
                        options={collaboratorRoleOptions}
                        onchange={(value) =>
                          updateMemberRole(
                            collaborator.user_id,
                            value as CollaboratorRole,
                          )}
                      />
                    </div>
                    <button
                      class="danger-btn"
                      type="button"
                      onclick={() => removeMember(collaborator.user_id)}
                      >Remove</button
                    >
                  {:else}
                    <span class="muted role-pill"
                      >{isOwner ? "owner" : collaborator.role}</span
                    >
                  {/if}
                </div>
              {/each}
            </div>
          {/if}
        </section>
      {/if}
    </div>
  </div>
{/if}

<style></style>
