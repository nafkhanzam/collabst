<script lang="ts">
  import { notifications, type Notification } from '$lib/stores/notifications'

  export let notification: Notification
</script>

<div 
  class="notification notification-{notification.type}" 
  class:closing={notification.closing}
>
  <span>{notification.message}</span>
  <button class="notification-close" on:click={() => notifications.hide(notification.id)}>×</button>
</div>

<style></style>
