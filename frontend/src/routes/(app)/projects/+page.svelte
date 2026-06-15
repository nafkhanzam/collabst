<script lang="ts">
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";
  import { auth } from "$lib/stores/auth";
  import { notifications } from "$lib/stores/notifications";
  import { projectsApi, invitationsApi } from "$lib/services/api";
  import { ThemeToggle, ProfileMenu, Tooltip } from "$lib/components/ui";
  import InvitationsPanel from "$lib/components/InvitationsPanel.svelte";
  import PlaceholderPanel from "$lib/components/editor/PlaceholderPanel.svelte";
  import ProjectsView from "$lib/components/projects/ProjectsView.svelte";
  import CreateProjectModal from "$lib/components/projects/CreateProjectModal.svelte";
  import InviteModal from "$lib/components/projects/InviteModal.svelte";
  import DeleteModal from "$lib/components/projects/DeleteModal.svelte";
  import type { CollaboratorRole, Project } from "$lib/types";
  import collabstLogo from "../../../assets/collabst-text.svg";
  import CircleHelp from "@lucide/svelte/icons/circle-help";
  import Rocket from "@lucide/svelte/icons/rocket";
  import Settings from "@lucide/svelte/icons/settings";

  let projects = $state<Project[]>([]);
  let loading = $state(true);
  let mounted = $state(false);
  let showCreateModal = $state(false);
  let showInviteModal = $state(false);
  let showDeleteModal = $state(false);
  let showSettingsPanel = $state(false);
  let selectedProjectId = $state<string | null>(null);
  let deleteProjectId = $state<string | null>(null);

  async function loadProjects() {
    try {
      projects = await projectsApi.list();
    } catch (error) {
      console.error("Failed to load projects:", error);
    } finally {
      loading = false;
    }
  }

  async function handleCreateProject(name: string, description: string) {
    try {
      const newProject = await projectsApi.create(name, description);
      showCreateModal = false;

      notifications.show("Project created successfully!", "info", 2000);
      // Navigate to the editor - it will handle creating the initial file
      goto(`/editor/${newProject.id}`);
    } catch (error: any) {
      console.error("Failed to create project:", error);
      const message =
        error?.response?.data?.detail || "Failed to create project";
      notifications.show(message, "error", 5000);
    }
  }

  async function handleDeleteProject(id: string) {
    deleteProjectId = id;
    showDeleteModal = true;
  }

  async function confirmDeleteProject() {
    if (deleteProjectId === null) return;

    try {
      await projectsApi.delete(deleteProjectId);
      loadProjects();
      notifications.show("Project deleted successfully", "info", 2000);
    } catch (error: any) {
      console.error("Failed to delete project:", error);
      const message =
        error?.response?.data?.detail || "Failed to delete project";
      notifications.show(message, "error", 5000);
    } finally {
      showDeleteModal = false;
      deleteProjectId = null;
    }
  }

  function handleOpenInviteModal(projectId: string) {
    selectedProjectId = projectId;
    showInviteModal = true;
  }

  async function handleSendInvite(email: string, role: CollaboratorRole) {
    if (!selectedProjectId) return;

    try {
      await invitationsApi.send(selectedProjectId, email, role);
      showInviteModal = false;
      notifications.show("Invitation sent successfully!", "info", 3000);
    } catch (error: any) {
      console.error("Failed to send invitation:", error);
      const message =
        error?.response?.data?.detail || "Failed to send invitation";
      notifications.show(message, "error", 5000);
    }
  }

  onMount(() => {
    if ($auth.user?.user_type === "guest") {
      goto("/login", { replaceState: true });
      return;
    }

    loadProjects();
    // Set mounted immediately to prevent layout shift
    mounted = true;
  });
</script>

<svelte:head>
  <title>Collabst</title>
</svelte:head>

{#if loading}
  <div class="loading">Loading projects...</div>
{:else}
  <div class="container" class:mounted>
    <header>
      <div class="header-left"></div>
      <div class="header-right">
        <ThemeToggle />
        <ProfileMenu />
      </div>
    </header>

    <div class="main-container">
      <!-- Activity Bar -->
      <div class="activity-bar">
        <div class="spacer"></div>
        <div class="bottom-activities">
          <Tooltip text="Settings" position="right">
            <button
              class="activity-btn"
              class:active={showSettingsPanel}
              onclick={() => (showSettingsPanel = !showSettingsPanel)}
              aria-label="Settings"
            >
              <Settings size={24} />
            </button>
          </Tooltip>
          <Tooltip text="Typst Universe" position="right">
            <a
              class="activity-btn"
              href="https://typst.app/universe"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Typst Universe"
            >
              <Rocket size={24} />
            </a>
          </Tooltip>
          <Tooltip text="Help" position="right">
            <a
              class="activity-btn"
              href="https://typst.app/docs"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Help"
            >
              <CircleHelp size={24} />
            </a>
          </Tooltip>

          <div class="logo-container">
            <img src={collabstLogo} alt="collabst" class="collabst-logo" />
          </div>
        </div>
      </div>

      <!-- Settings Panel -->
      {#if showSettingsPanel}
        <PlaceholderPanel title="Global Settings" />
      {/if}

      <!-- Main Content -->
      <div class="content-wrapper">
        <div class="invitations-section">
          <InvitationsPanel />
        </div>

        <ProjectsView
          {projects}
          onCreateProject={() => (showCreateModal = true)}
          onInvite={handleOpenInviteModal}
          onDelete={handleDeleteProject}
        />
      </div>
    </div>

    <CreateProjectModal
      bind:show={showCreateModal}
      onSubmit={handleCreateProject}
    />

    <InviteModal bind:show={showInviteModal} onSubmit={handleSendInvite} />

    <DeleteModal bind:show={showDeleteModal} onConfirm={confirmDeleteProject} />
  </div>
{/if}

<style></style>
