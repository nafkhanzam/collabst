<script lang="ts">
  import type { File as ProjectFile, Asset, FileTreeNode } from "$lib/types";
  import { Tooltip } from "$lib/components/ui";
  import File from "@lucide/svelte/icons/file";
  import FileText from "@lucide/svelte/icons/file-text";
  import Image from "@lucide/svelte/icons/image";
  import BookOpen from "@lucide/svelte/icons/book-open";
  import Video from "@lucide/svelte/icons/video";
  import Music from "@lucide/svelte/icons/music";
  import Paperclip from "@lucide/svelte/icons/paperclip";
  import Eye from "@lucide/svelte/icons/eye";
  import EyeOff from "@lucide/svelte/icons/eye-closed";
  import Folder from "@lucide/svelte/icons/folder";
  import FolderOpen from "@lucide/svelte/icons/folder-open";
  import ChevronRight from "@lucide/svelte/icons/chevron-right";
  import ChevronDown from "@lucide/svelte/icons/chevron-down";

  export let item: (ProjectFile & FileTreeNode) | Asset;
  export let isSelected: boolean = false;
  export let isPreview: boolean = false;
  export let onSelect: () => void;
  export let onSetPreview: (() => void) | undefined = undefined;
  export let onRename: ((newName: string) => Promise<void>) | null = null;
  export let onToggleFolder: (() => void) | undefined = undefined;
  export let onDragStart: (() => void) | undefined = undefined;
  export let onDragEnd: (() => void) | undefined = undefined;
  export let usersViewing: { name: string; color: string }[] = [];

  let isEditing = false;
  let editingName = "";
  let inputElement: HTMLInputElement;
  let isSubmitting = false;

  function handleDragStart(e: DragEvent) {
    if (!e.dataTransfer) return;

    // Store the item being dragged
    e.dataTransfer.effectAllowed = "move";
    e.dataTransfer.setData(
      "application/json",
      JSON.stringify({
        id: item.id,
        isAsset: isAsset(item),
        isFolder: "is_folder" in item && item.is_folder,
      }),
    );

    // Add dragging class
    if (e.target instanceof HTMLElement) {
      e.target.classList.add("dragging");
    }

    // Notify parent
    if (onDragStart) {
      onDragStart();
    }
  }

  function handleDragEnd(e: DragEvent) {
    if (e.target instanceof HTMLElement) {
      e.target.classList.remove("dragging");
    }

    // Notify parent
    if (onDragEnd) {
      onDragEnd();
    }
  }

  function getFileIconComponent(item: (ProjectFile & FileTreeNode) | Asset) {
    // Check if it's an asset
    if ("mime_type" in item) {
      if (item.mime_type.startsWith("image/")) return Image;
      if (item.mime_type.startsWith("video/")) return Video;
      if (item.mime_type.startsWith("audio/")) return Music;
      if (item.mime_type === "application/pdf") return FileText;
      return Paperclip;
    }

    // Check if it's a folder
    if ("is_folder" in item && item.is_folder) {
      return "isExpanded" in item && item.isExpanded ? FolderOpen : Folder;
    }

    // It's a file - check extension
    const name = item.name.toLowerCase();
    if (name.endsWith(".bib")) return BookOpen;
    if (name.endsWith(".pdf")) return FileText;
    if (
      name.endsWith(".svg") ||
      name.endsWith(".png") ||
      name.endsWith(".jpg") ||
      name.endsWith(".jpeg") ||
      name.endsWith(".gif") ||
      name.endsWith(".webp")
    )
      return Image;
    return File;
  }

  function getFileName(item: ProjectFile | Asset): string {
    return "filename" in item ? item.filename : item.name;
  }

  function getFileSize(item: ProjectFile | Asset): string | null {
    if ("size" in item) {
      const bytes = item.size;
      if (bytes < 1024) return `${bytes} B`;
      if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
      return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
    }
    return null;
  }

  function isAsset(item: ProjectFile | Asset): item is Asset {
    return "mime_type" in item;
  }

  function isTypstFile(item: ProjectFile | Asset): boolean {
    if (isAsset(item)) return false;
    const name = item.name.toLowerCase();
    return name.endsWith(".typ");
  }

  function handleDoubleClick() {
    if (onRename) {
      isEditing = true;
      editingName = getFileName(item);
      setTimeout(() => {
        if (inputElement) {
          inputElement.focus();
          // Select only the filename without extension
          const lastDotIndex = editingName.lastIndexOf(".");
          // Handle dotfiles: if dot at position 0 (.gitignore), skip the dot
          if (lastDotIndex === 0) {
            // Leading dot (dotfile): select from 1 to end
            inputElement.setSelectionRange(1, editingName.length);
          } else if (lastDotIndex > 0) {
            // Normal file with extension: select from start to before extension
            inputElement.setSelectionRange(0, lastDotIndex);
          } else {
            // No extension: select all
            inputElement.select();
          }
        }
      }, 0);
    }
  }

  async function handleRenameSubmit() {
    if (isSubmitting) return;

    const trimmedName = editingName.trim();
    const currentName = getFileName(item);

    // If no changes, just close the editor
    if (!trimmedName || trimmedName === currentName) {
      isEditing = false;
      isSubmitting = false;
      return;
    }

    isSubmitting = true;

    try {
      if (onRename) {
        await onRename(trimmedName);
      }
      isEditing = false;
    } catch (error) {
      console.error("Failed to rename:", error);
      // Keep editing mode open on error so user can try again
    } finally {
      isSubmitting = false;
    }
  }

  function handleRenameCancel() {
    isEditing = false;
    editingName = "";
    isSubmitting = false;
  }

  function handleRenameKeydown(e: KeyboardEvent) {
    if (e.key === "Enter") {
      e.preventDefault();
      handleRenameSubmit();
    } else if (e.key === "Escape") {
      e.preventDefault();
      handleRenameCancel();
    } else if (e.key === "Delete" || e.key === "Backspace") {
      // Allow Delete and Backspace to edit text in rename input
      // Prevent bubbling to parent keyboard handler that would delete the file
      e.stopPropagation();
    }
  }
</script>

<div
  class="file-item"
  class:active={isSelected}
  class:asset={isAsset(item)}
  class:folder={"is_folder" in item && item.is_folder}
  style="padding-left: {'level' in item
    ? `${item.level * 1.5 + 0.5}rem`
    : '0.5rem'}"
  draggable={!isEditing}
  on:dragstart={handleDragStart}
  on:dragend={handleDragEnd}
  on:click={() => {
    // Always call onSelect to track last clicked item
    onSelect();
    // Additionally toggle folder if it's a folder
    if ("is_folder" in item && item.is_folder && onToggleFolder) {
      onToggleFolder();
    }
  }}
  on:dblclick={handleDoubleClick}
  role="button"
  tabindex="0"
  on:keydown={(e) => {
    if (e.key === "Enter") {
      if ("is_folder" in item && item.is_folder && onToggleFolder) {
        onToggleFolder();
      } else {
        onSelect();
      }
    }
  }}
>
  <span class="icon">
    <svelte:component this={getFileIconComponent(item)} size={16} />
  </span>
  <div class="info">
    <div class="name-row">
      {#if isEditing}
        <input
          bind:this={inputElement}
          bind:value={editingName}
          on:blur={handleRenameCancel}
          on:keydown={handleRenameKeydown}
          class="name-input"
          type="text"
          on:click|stopPropagation
        />
      {:else}
        <span class="name">{getFileName(item)}</span>
      {/if}
      {#if usersViewing.length > 0}
        <div class="user-indicators">
          {#each usersViewing as user}
            <div
              class="user-dot"
              style="background-color: {user.color}; color: {user.color};"
              title="{user.name} is viewing"
            ></div>
          {/each}
        </div>
      {/if}
      {#if isTypstFile(item) && onSetPreview}
        <Tooltip text={isPreview ? "Preview enabled" : "Preview this file"}>
          <button class="preview-btn" on:click|stopPropagation={onSetPreview}>
            <svelte:component this={isPreview ? Eye : EyeOff} size={16} />
          </button>
        </Tooltip>
      {/if}
      {#if "is_folder" in item && item.is_folder}
        <Tooltip text={item.isExpanded ? "Collapse folder" : "Expand folder"}>
          <button
            class="chevron-btn"
            on:click|stopPropagation={() => onToggleFolder?.()}
            aria-label={item.isExpanded ? "Collapse folder" : "Expand folder"}
          >
            <svelte:component
              this={item.isExpanded ? ChevronDown : ChevronRight}
              size={16}
            />
          </button>
        </Tooltip>
      {/if}
    </div>
    <!-- {#if getFileSize(item)}
      <span class="size">{getFileSize(item)}</span>
    {/if} -->
  </div>
</div>

<style></style>
