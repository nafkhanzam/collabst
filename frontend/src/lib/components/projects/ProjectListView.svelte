<script lang="ts">
  import type { Project } from "$lib/types";
  import ListActionButton from "./ListActionButton.svelte";
  import RoleBadge from "$lib/components/ui/RoleBadge.svelte";
  import File from "@lucide/svelte/icons/file";
  import Play from "@lucide/svelte/icons/play";
  import Trash2 from "@lucide/svelte/icons/trash-2";
  import UserPlus from "@lucide/svelte/icons/user-plus";
  import ChevronDown from "@lucide/svelte/icons/chevron-down";

  export let projects: Project[];
  export let sortBy: "name" | "created" | "modified";
  export let onSortByColumn: (column: "name" | "created" | "modified") => void;
  export let onInvite: (projectId: string) => void;
  export let onDelete: (projectId: string) => void;

  function formatDate(dateString: string): string {
    const date = new Date(dateString);
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));

    if (diffDays === 0) {
      return "Today";
    } else if (diffDays === 1) {
      return "Yesterday";
    } else if (diffDays < 7) {
      return `${diffDays} days ago`;
    } else {
      return date.toLocaleDateString();
    }
  }
</script>

<div class="projects-list">
  {#if projects.length === 0}
    <div class="empty">
      <h2>No projects yet</h2>
      <p>Create your first project to get started!</p>
    </div>
  {:else}
    <div class="list-header">
      <button
        class="list-header-cell project-col"
        onclick={() => onSortByColumn("name")}
      >
        Project
        {#if sortBy === "name"}
          <ChevronDown size={16} />
        {/if}
      </button>
      <div class="list-header-cell actions-col"></div>
      <button
        class="list-header-cell"
        onclick={() => onSortByColumn("created")}
      >
        Created
        {#if sortBy === "created"}
          <ChevronDown size={16} />
        {/if}
      </button>
      <button
        class="list-header-cell"
        onclick={() => onSortByColumn("modified")}
      >
        Last Modified
        {#if sortBy === "modified"}
          <ChevronDown size={16} />
        {/if}
      </button>
    </div>

    {#each projects as project, index (project.id)}
      <div class="list-row" class:odd={index % 2 === 1}>
        <a
          href="/editor/{project.id}"
          class="list-row-link"
          aria-label="Open {project.name}"
        ></a>

        <div class="list-cell project-col">
          <File size={18} class="project-file-icon" />
          <span class="project-name">{project.name}</span>
          {#if project.current_user_role}
            <RoleBadge role={project.current_user_role} />
          {/if}
        </div>

        <div class="list-cell actions-col">
          <div class="list-action-buttons">
            <ListActionButton
              action="open"
              icon={Play}
              title="Open"
              href="/editor/{project.id}"
            />

            {#if project.current_user_role === "owner" || project.current_user_role === "admin"}
              <ListActionButton
                action="invite"
                icon={UserPlus}
                title="Invite"
                onclick={(e) => {
                  e.stopPropagation();
                  onInvite(project.id);
                }}
              />
            {/if}

            {#if project.current_user_role === "owner"}
              <ListActionButton
                action="delete"
                icon={Trash2}
                title="Delete"
                onclick={(e) => {
                  e.stopPropagation();
                  onDelete(project.id);
                }}
              />
            {/if}
          </div>
        </div>

        <div class="list-cell">
          {formatDate(project.created_at)}
        </div>

        <div class="list-cell">
          {formatDate(project.updated_at)}
        </div>
      </div>
    {/each}
  {/if}
</div>

<style></style>
