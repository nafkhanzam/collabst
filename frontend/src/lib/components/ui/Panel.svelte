<script lang="ts">
  import type { Snippet } from 'svelte'
  
  interface PanelProps {
    title?: string
    subtitle?: string
    padding?: boolean
    class?: string
    children?: Snippet
    actions?: Snippet
  }
  
  let {
    title = '',
    subtitle = '',
    padding = true,
    class: className = '',
    children,
    actions
  }: PanelProps = $props()
</script>

<div class="panel {className}">
  {#if title || actions}
    <div class="panel-header">
      <div class="panel-header-content">
        {#if title}
          <h3 class="panel-title">{title}</h3>
        {/if}
        {#if subtitle}
          <p class="panel-subtitle">{subtitle}</p>
        {/if}
      </div>
      {#if actions}
        <div class="panel-actions">
          {@render actions()}
        </div>
      {/if}
    </div>
  {/if}
  
  <div class="panel-body {padding ? 'panel-padding' : ''}">
    {@render children?.()}
  </div>
</div>

<style></style>
