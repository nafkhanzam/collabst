<script lang="ts">
  import type { Diagnostic } from "$lib/types";

  export let diagnostics: Diagnostic[] = [];
  export let gotoDiagnostic: (diagnostic: Diagnostic) => void;

  function handleKeydown(event: KeyboardEvent, diagnostic: Diagnostic) {
    if (event.key === "Enter" || event.key === " ") {
      event.preventDefault();
      gotoDiagnostic(diagnostic);
    }
  }

  function severityValue(severity: string): number {
    switch (severity) {
      case "error":
        return 1;
      case "warning":
        return 2;
      case "info":
        return 3;
      case "hint":
        return 4;
      default:
        return 5;
    }
  }

  let sortedDiagnostics: Diagnostic[] = [];
  $: sortedDiagnostics = diagnostics.slice().sort((a, b) => {
    return severityValue(a.severity) - severityValue(b.severity);
  });
</script>

<div class="issues-panel">
  <div class="panel-header">
    <h3>Issues & Suggestions</h3>
  </div>
  <div class="panel-content">
    {#if diagnostics.length === 0}
      <p>No issues or suggestions found.</p>
    {:else}
      {#each sortedDiagnostics as diagnostic}
        <div
          class="issue-item issue-severity-{diagnostic.severity}"
          role="button"
          tabindex="0"
          on:click={() => gotoDiagnostic(diagnostic)}
          on:keydown={(e) => handleKeydown(e, diagnostic)}
        >
          <strong>{diagnostic.severity}: {diagnostic.message}</strong>
          {#if diagnostic.range}
            <p class="location-text">in {diagnostic.path}</p>
            <p class="location-text">
              at
              {diagnostic.range.start.line + 1}:{diagnostic.range.start
                .character + 1}
              -
              {diagnostic.range.end.line + 1}:{diagnostic.range.end.character +
                1}
            </p>
          {/if}
        </div>
      {/each}
    {/if}
  </div>
</div>

<style></style>
