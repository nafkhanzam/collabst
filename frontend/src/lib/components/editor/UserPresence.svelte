<script lang="ts">
  import type { WebsocketProvider } from 'y-websocket'
  import { getProfilePicUrl } from '$lib/utils/urls'
  import VenetianMask from '@lucide/svelte/icons/venetian-mask';

  interface UserPresenceProps {
    provider: WebsocketProvider | null
  }

  let { provider }: UserPresenceProps = $props()

  let awarenessStates: [number, any][] = $state([])
  let loadedProfilePics = $state<Record<string, boolean>>({})

  function getUserName(state: any) {
    return state?.user?.name || state?.user?.display_name || null
  }

  function updateAwareness() {
    if (provider?.awareness) {
      awarenessStates = Array.from(provider.awareness.getStates().entries())
    } else {
      awarenessStates = []
    }
  }

  $effect(() => {
    if (!provider?.awareness) {
      awarenessStates = []
      return
    }

    provider.awareness.on('change', updateAwareness)
    updateAwareness()

    return () => {
      provider.awareness.off('change', updateAwareness)
    }
  })

  // Filter out states without user info and limit to show max 10 users
  let users = $derived(
    awarenessStates.filter(([_, state]) => getUserName(state)).slice(0, 10)
  )

  let totalUsers = $derived(
    awarenessStates.filter(([_, state]) => getUserName(state)).length
  )

  function profilePicSrc(userId: string) {
    return getProfilePicUrl(userId)
  }

  function handleAvatarLoad(userId: string) {
    loadedProfilePics = { ...loadedProfilePics, [userId]: true }
  }

  function hasLoadedAvatar(userId: string) {
    return !!loadedProfilePics[userId]
  }
</script>

<div class="user-presence">
  {#if users.length > 0}
    <div class="user-avatars">
      {#each users as [clientId, state], index}
        <div
          class="avatar"
          style="background: {state.user?.color || '#999'}; z-index: {100 - index}; --avatar-glow: {state.user?.color || '#999'}70"
        >
          {#if state.user?.user_type === 'guest'}
            <VenetianMask size={30} style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: white; opacity: 0.3;" />
          {/if}
          {#if state.user?.id}
            <span class="avatar-initial" class:avatar-initial-hidden={hasLoadedAvatar(state.user.id)}>
              {(getUserName(state) || 'U')[0].toUpperCase()}
            </span>
            <img
              class="avatar-image"
              class:avatar-image-loaded={hasLoadedAvatar(state.user.id)}
              src={profilePicSrc(state.user.id)}
              alt={`${getUserName(state) || 'User'} avatar`}
              onload={() => handleAvatarLoad(state.user.id)}
              onerror={() => {}}
            />
          {:else}
            <span class="avatar-initial">
              {(getUserName(state) || 'U')[0].toUpperCase()}
            </span>
          {/if}
          <div class="tooltip">
            {#if state.user?.user_type === 'guest'}
              <VenetianMask size={14} />
            {/if}
            {getUserName(state) || `User ${clientId}`}
          </div>
        </div>
      {/each}
      {#if totalUsers > 10}
        <div class="avatar more" style="z-index: 0">
          <span class="avatar-initial">+{totalUsers - 10}</span>
          <div class="tooltip">
            +{totalUsers - 10} more
          </div>
        </div>
      {/if}
    </div>
  {/if}
</div>

<style></style>