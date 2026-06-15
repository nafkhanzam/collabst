<script lang="ts">
  import Hand from "@lucide/svelte/icons/hand";
  import X from "@lucide/svelte/icons/x";
  import ListX from "@lucide/svelte/icons/list-x";

  interface UploadAssetModalProps {
    show: boolean;
    onClose: () => void;
    onUpload: (files: File[]) => Promise<void> | void;
  }

  let { show, onClose, onUpload }: UploadAssetModalProps = $props();

  let fileInput: HTMLInputElement | undefined = $state();
  let stagedFiles: File[] = $state([]);
  let isDragging = $state(false);
  let isUploading = $state(false);

  function handleClose() {
    if (isUploading) return;
    onClose();
  }

  // Handle Escape key when modal is open
  $effect(() => {
    if (!show) return;

    const handleKeydown = (e: KeyboardEvent) => {
      if (e.key === "Escape") {
        handleClose();
      }
    };

    window.addEventListener("keydown", handleKeydown);
    return () => window.removeEventListener("keydown", handleKeydown);
  });

  function fileSignature(file: File): string {
    return `${file.name}:${file.size}:${file.lastModified}`;
  }

  function addFiles(files: FileList | File[]) {
    const incoming = Array.from(files);
    if (incoming.length === 0) return;

    const existing = new Set(stagedFiles.map(fileSignature));
    const uniqueIncoming = incoming.filter((file) => {
      const signature = fileSignature(file);
      if (existing.has(signature)) {
        return false;
      }
      existing.add(signature);
      return true;
    });

    if (uniqueIncoming.length > 0) {
      stagedFiles = [...stagedFiles, ...uniqueIncoming];
    }
  }

  function handleFileSelect(e: Event) {
    const target = e.target as HTMLInputElement;
    if (target.files) {
      addFiles(target.files);
    }
    target.value = "";
  }

  function handleDrop(e: DragEvent) {
    e.preventDefault();
    isDragging = false;
    const files = e.dataTransfer?.files;
    if (files) {
      addFiles(files);
    }
  }

  function handleDragOver(e: DragEvent) {
    e.preventDefault();
    isDragging = true;
  }

  function handleDragLeave(e: DragEvent) {
    e.preventDefault();
    isDragging = false;
  }

  async function handleSubmit(e: Event) {
    e.preventDefault();
    if (stagedFiles.length === 0 || isUploading) return;

    isUploading = true;
    try {
      await onUpload(stagedFiles);
      stagedFiles = [];
      if (fileInput) fileInput.value = "";
    } finally {
      isUploading = false;
    }
  }

  function removeStagedFile(index: number) {
    stagedFiles = stagedFiles.filter((_, i) => i !== index);
  }

  function clearStagedFiles() {
    stagedFiles = [];
    if (fileInput) fileInput.value = "";
  }

  function formatFileSize(bytes: number): string {
    if (bytes < 1024) return `${bytes} B`;
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
    return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
  }

  function handleBackdropClick(e: MouseEvent) {
    if (e.target === e.currentTarget) {
      handleClose();
    }
  }

  function handleDropZoneKeydown(e: KeyboardEvent) {
    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      openFilePicker();
    }
  }

  function openFilePicker() {
    fileInput?.click();
  }
</script>

{#if show}
  <div
    class="modal"
    onclick={handleBackdropClick}
    role="presentation"
  >
    <div class="modal-content" role="dialog" aria-modal="true" tabindex="-1">
      <h2>Upload Files</h2>
      <form onsubmit={handleSubmit}>
        <input
          bind:this={fileInput}
          type="file"
          multiple
          onchange={handleFileSelect}
          style="display: none;"
        />

        <div
          class="drop-zone"
          class:dragging={isDragging}
          onclick={openFilePicker}
          ondrop={handleDrop}
          ondragover={handleDragOver}
          ondragleave={handleDragLeave}
          onkeydown={handleDropZoneKeydown}
          role="button"
          tabindex="0"
        >
          <Hand size={64} strokeWidth={1.5} />
          <p class="drop-text">
            Click or drag and drop files here to stage them
          </p>
        </div>

        {#if stagedFiles.length > 0}
          <div class="file-list-header">
            <p class="file-info">{stagedFiles.length} file(s) ready to upload</p>
            <button
              type="button"
              class="clear-btn"
              onclick={clearStagedFiles}
              disabled={isUploading}
            >
              <ListX size={16} />
              Clear List
            </button>
          </div>

          <ul class="file-list" aria-label="Staged upload files">
            {#each stagedFiles as stagedFile, index}
              <li class="file-row">
                <div class="file-meta">
                  <span class="file-name" title={stagedFile.name}>{stagedFile.name}</span>
                  <span class="file-size">{formatFileSize(stagedFile.size)}</span>
                </div>
                <button
                  type="button"
                  class="remove-btn"
                  aria-label={`Remove ${stagedFile.name}`}
                  title="Remove from upload batch"
                  onclick={() => removeStagedFile(index)}
                  disabled={isUploading}
                >
                  <X size={16} />
                </button>
              </li>
            {/each}
          </ul>
        {/if}

        <div class="modal-actions">
          <button type="button" onclick={handleClose} class="cancel-btn" disabled={isUploading}>
            Cancel
          </button>
          <button
            type="submit"
            class="submit-btn"
            disabled={stagedFiles.length === 0 || isUploading}
          >
            {isUploading ? "Uploading..." : "Upload"}
          </button>
        </div>
      </form>
    </div>
  </div>
{/if}

<style></style>
