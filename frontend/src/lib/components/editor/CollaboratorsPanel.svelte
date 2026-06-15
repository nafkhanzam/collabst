<script lang="ts">
  import type { WebsocketProvider } from 'y-websocket'

  export let provider: WebsocketProvider | null
  export let onClose: () => void

  $: awarenessStates = provider?.awareness
    ? Array.from(provider.awareness.getStates().entries())
    : []
</script>

<aside class="collaborators-panel">
  <div class="panel-header">
    <h3>Collaborators</h3>
    <button on:click={onClose} class="close-btn">
      ×
    </button>
  </div>
  <div class="panel-content">
    <p class="info">
      Manage collaborators from the project settings
    </p>
    <div class="online-users">
      <h4>Online Now ({awarenessStates.length})</h4>
      {#if awarenessStates.length > 0}
        <div class="user-list">
          {#each awarenessStates as [clientId, state]}
            <div class="user">
              <div
                class="user-dot"
                style="background: {state.user?.color || '#999'}"
              ></div>
              <span class="user-name">{state.user?.name || `User ${clientId}`}</span>
            </div>
          {/each}
        </div>
      {:else}
        <p class="no-users">No other users online</p>
      {/if}
    </div>
  </div>
</aside>

<style></style>
