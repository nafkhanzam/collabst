<script lang="ts">
  import type { Project } from "$lib/types";
  import GridActionButton from "./GridActionButton.svelte";
  import RoleBadge from "$lib/components/ui/RoleBadge.svelte";
  import fileIcon from "../../../assets/collabst-file.svg";
  import Play from "@lucide/svelte/icons/play";
  import Trash2 from "@lucide/svelte/icons/trash-2";
  import UserPlus from "@lucide/svelte/icons/user-plus";

  export let projects: Project[];
  export let onInvite: (projectId: string) => void;
  export let onDelete: (projectId: string) => void;
</script>

<div class="projects-grid">
  {#if projects.length === 0}
    <div class="empty">
      <h2>No projects yet</h2>
      <p>Create your first project to get started!</p>
    </div>
  {:else}
    {#each projects as project (project.id)}
      <div class="project-card">
        <a
          href="/editor/{project.id}"
          class="card-link"
          aria-label="Open {project.name}"
        ></a>

        <div class="file-icon-container">
          <img src={fileIcon} alt="Project file" class="file-icon" />

          <div class="action-buttons">
            <GridActionButton
              action="open"
              label="Open"
              icon={Play}
              href="/editor/{project.id}"
            />

            {#if project.current_user_role === "owner" || project.current_user_role === "admin"}
              <GridActionButton
                action="invite"
                label="Invite"
                icon={UserPlus}
                onclick={(e) => {
                  e.stopPropagation();
                  onInvite(project.id);
                }}
              />
            {/if}

            {#if project.current_user_role === "owner"}
              <GridActionButton
                action="delete"
                label="Delete"
                icon={Trash2}
                onclick={(e) => {
                  e.stopPropagation();
                  onDelete(project.id);
                }}
              />
            {/if}
          </div>
        </div>

        <div class="project-info">
          <h3>{project.name}</h3>
          {#if project.current_user_role}
            <RoleBadge role={project.current_user_role} />
          {/if}
        </div>
      </div>
    {/each}
  {/if}
</div>

<style></style>
