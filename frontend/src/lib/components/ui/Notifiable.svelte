<script lang="ts">
  type NotificationType = "error" | "warning" | "info" | "hint" | "comments";

  export let hasNotification: boolean = false;
  export let position:
    | "top-right"
    | "top-left"
    | "bottom-right"
    | "bottom-left" = "top-right";
  export let size: number = 20;
  export let count: number | null = null;
  export let type: NotificationType = "error";
</script>

<div class="notifiable" class:has-notification={hasNotification}>
  <slot />
  {#if hasNotification}
    <div
      class={`notification-dot type-${type}`}
      class:position
      class:has-count={count !== null}
      style="--dot-size: {size}px"
    >
      {#if count !== null}
        <span class="count-text">{count}</span>
      {/if}
    </div>
  {/if}
</div>

<style></style>
