<script lang="ts">
  import { notifications } from '$lib/stores/notifications'
  import Notification from './Notification.svelte'
</script>

<div class="notification-container">
  {#each $notifications as notification (notification.id)}
    <Notification {notification} />
  {/each}
</div>

<style></style>
