<script lang="ts">
  import { onMount } from 'svelte'
  import { invitationsApi } from '../services/api'
  import RoleBadge from './ui/RoleBadge.svelte'
  import type { Invitation } from '../types'

  let invitations: Invitation[] = []
  let loading = true

  async function loadInvitations() {
    try {
      invitations = await invitationsApi.listPending()
    } catch (error) {
      console.error('Failed to load invitations:', error)
    } finally {
      loading = false
    }
  }

  async function handleAccept(id: string) {
    try {
      await invitationsApi.accept(id)
      loadInvitations()
      window.location.reload()
    } catch (error) {
      console.error('Failed to accept invitation:', error)
      alert('Failed to accept invitation')
    }
  }

  async function handleDecline(id: string) {
    try {
      await invitationsApi.decline(id)
      loadInvitations()
    } catch (error) {
      console.error('Failed to decline invitation:', error)
    }
  }

  onMount(() => {
    loadInvitations()
  })
</script>

{#if loading}
  <div class="loading">Loading invitations...</div>
{:else if invitations.length > 0}
  <div class="container">
    <div class="header">
      <h3>Pending Invitations ({invitations.length})</h3>
    </div>
    <div class="list">
      {#each invitations as invitation (invitation.id)}
        <div class="invitation">
          <div class="info">
            <div class="role-badge-wrap">
              <RoleBadge role={invitation.role} size="sm" uppercase={true} />
            </div>
            <div class="email">From: {invitation.invitee_email}</div>
            <div class="date">
              {new Date(invitation.created_at).toLocaleDateString()}
            </div>
          </div>
          <div class="actions">
            <button
              on:click={() => handleAccept(invitation.id)}
              class="accept"
            >
              Accept
            </button>
            <button
              on:click={() => handleDecline(invitation.id)}
              class="decline"
            >
              Decline
            </button>
          </div>
        </div>
      {/each}
    </div>
  </div>
{/if}

<style></style>
