<script lang="ts">
  import type { Comment, UserProfile } from "$lib/types";
  import { getProfilePicUrl } from "$lib/utils/urls";
  import VenetianMask from "@lucide/svelte/icons/venetian-mask";
  import Tooltip from "../ui/Tooltip.svelte";
  import Ellipsis from "@lucide/svelte/icons/ellipsis";

  interface CommentThreadProps {
    comment: Comment;
    userProfiles: Record<string, UserProfile>;
    currentUserId: string;
    canComment?: boolean;
    canDeleteComments?: boolean;
    isActive?: boolean;
    isHovered?: boolean;
    onResolve?: (commentId: string) => void;
    onReopen?: (commentId: string) => void;
    onDelete?: (commentId: string) => void;
    onReply?: (commentId: string, content: string) => void;
    onSelect?: (commentId: string) => void;
    onHover?: (commentId: string) => void;
    onHoverEnd?: (commentId: string) => void;
  }

  let {
    comment,
    userProfiles,
    currentUserId,
    canComment = true,
    canDeleteComments = false,
    isActive = false,
    onResolve,
    onReopen,
    onDelete,
    onReply,
    onSelect,
    onHover,
    onHoverEnd,
    isHovered = false,
  }: CommentThreadProps = $props();

  let replyText = $state("");
  let replyFocused = $state(false);
  let replyTextarea: HTMLTextAreaElement | undefined = $state();
  let showMenu = $state(false);
  let loadedProfilePics = $state<Record<string, boolean>>({});

  function formatDate(dateStr: string) {
    const date = new Date(dateStr);
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMs / 3600000);
    const diffDays = Math.floor(diffMs / 86400000);

    if (diffMins < 1) return "just now";
    if (diffMins < 60) return `${diffMins}m ago`;
    if (diffHours < 24) return `${diffHours}h ago`;
    if (diffDays < 7) return `${diffDays}d ago`;
    return date.toLocaleDateString();
  }

  function handleResolve() {
    onResolve?.(comment.id);
  }

  function handleReopen() {
    onReopen?.(comment.id);
  }

  function handleDelete() {
    onDelete?.(comment.id);
  }

  function handleSubmitReply() {
    if (replyText.trim()) {
      onReply?.(comment.id, replyText.trim());
      replyText = "";
      replyFocused = false;
      replyTextarea?.blur();
    }
  }

  function handleCancelReply() {
    replyText = "";
    replyFocused = false;
    replyTextarea?.blur();
  }

  function toggleMenu(e: MouseEvent) {
    e.stopPropagation();
    showMenu = !showMenu;
    if (showMenu) {
      // Add event listener to close menu when clicking outside
      document.addEventListener("click", closeMenu);
    } else {
      document.removeEventListener("click", closeMenu);
    }
  }

  function closeMenu() {
    showMenu = false;
  }

  function handleMenuAction(action: () => void) {
    return (e: MouseEvent) => {
      e.stopPropagation();
      action();
      closeMenu();
    };
  }

  function authorName(userId: string) {
    return userProfiles[userId]?.display_name || `User ${userId}`;
  }

  function isGuestAuthor(userId: string) {
    const profile = userProfiles[userId];
    if (!profile) return false;
    return profile.user_type === "guest";
  }

  function authorColor(userId: string) {
    const safeUserId = String(userId ?? "");
    const seed = safeUserId
      .split("")
      .reduce((acc, char) => acc + char.charCodeAt(0), 0);
    const hue = (seed * 47) % 360;
    return `hsl(${hue} 55% 45%)`;
  }

  function profilePicSrc(userId: string) {
    return getProfilePicUrl(userId);
  }

  function handleAvatarLoad(userId: string) {
    loadedProfilePics = { ...loadedProfilePics, [userId]: true };
  }

  function hasLoadedAvatar(userId: string) {
    return !!loadedProfilePics[userId];
  }

  let canDeleteThisComment = $derived(canDeleteComments);

  let canShowActionMenu = $derived(canComment || canDeleteThisComment);
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  class="comment-thread"
  class:resolved={comment.resolved}
  class:active={isActive}
  class:hovered={isHovered || showMenu}
  onclick={() => onSelect?.(comment.id)}
  onmouseenter={() => onHover?.(comment.id)}
  onmouseleave={() => onHoverEnd?.(comment.id)}
>
  <div class="comment-header">
    <div class="author-info">
      <div
        class="author-avatar"
        style="background-color: {authorColor(comment.authorId)}"
        title={authorName(comment.authorId)}
        onclick={(e) => e.stopPropagation()}
      >
        <span
          class="avatar-fallback"
          class:avatar-fallback-hidden={hasLoadedAvatar(comment.authorId)}
        >
          {authorName(comment.authorId).charAt(0).toUpperCase()}
        </span>
        <img
          class="avatar-image"
          class:avatar-image-loaded={hasLoadedAvatar(comment.authorId)}
          src={profilePicSrc(comment.authorId)}
          alt={`${authorName(comment.authorId)} avatar`}
          onload={() => handleAvatarLoad(comment.authorId)}
          onerror={() => {}}
        />
      </div>
      <div class="author-details">
        <span class="author-name">
          {#if isGuestAuthor(comment.authorId)}
            <Tooltip
              text="Guest account: temporary access via shared link"
            >
              <VenetianMask size={16} />
            </Tooltip>
          {/if}
          {authorName(comment.authorId)}
        </span>
        <span class="comment-time">{formatDate(comment.createdAt)}</span>
      </div>
    </div>
    <div class="comment-actions">
      <div class="menu-container">
        {#if canShowActionMenu}
          <Tooltip text="More actions">
            <button
              class="menu-btn"
              onclick={toggleMenu}
            >
              <Ellipsis size={16} />
            </button>
          </Tooltip>
        {/if}
        {#if canShowActionMenu && showMenu}
          <!-- svelte-ignore a11y_click_events_have_key_events -->
          <!-- svelte-ignore a11y_no_static_element_interactions -->
          <div
            class="menu-backdrop"
            onclick={handleMenuAction(closeMenu)}
          ></div>
          <div class="menu-dropdown">
            {#if !comment.resolved}
              <button
                class="menu-item"
                onclick={handleMenuAction(handleResolve)}
              >
                <span class="menu-icon">✓</span> Resolve
              </button>
            {:else}
              <button
                class="menu-item"
                onclick={handleMenuAction(handleReopen)}
              >
                <span class="menu-icon">⟳</span> Reopen
              </button>
            {/if}
            {#if canDeleteThisComment}
              <button
                class="menu-item menu-item-danger"
                onclick={handleMenuAction(handleDelete)}
              >
                <span class="menu-icon">✕</span> Delete
              </button>
            {/if}
          </div>
        {/if}
      </div>
    </div>
  </div>

  <div class="comment-content">
    {comment.content}
  </div>

  {#if comment.replies.length > 0}
    <div class="replies">
      {#each comment.replies as reply}
        <div class="reply">
          <div class="reply-header">
            <div
              class="reply-avatar"
              style="background-color: {authorColor(reply.authorId)}"
              title={authorName(reply.authorId)}
              onclick={(e) => e.stopPropagation()}
            >
              <span
                class="avatar-fallback"
                class:avatar-fallback-hidden={hasLoadedAvatar(reply.authorId)}
              >
                {authorName(reply.authorId).charAt(0).toUpperCase()}
              </span>
              <img
                class="avatar-image"
                class:avatar-image-loaded={hasLoadedAvatar(reply.authorId)}
                src={profilePicSrc(reply.authorId)}
                alt={`${authorName(reply.authorId)} avatar`}
                onload={() => handleAvatarLoad(reply.authorId)}
                onerror={() => {}}
              />
            </div>
            <span class="reply-author">
              {#if isGuestAuthor(reply.authorId)}
                <Tooltip
                  text="Guest account: temporary access via shared link"
                >
                  <VenetianMask size={14} />
                </Tooltip>
              {/if}
              {authorName(reply.authorId)}
            </span>
            <span class="reply-time">{formatDate(reply.createdAt)}</span>
          </div>
          <div class="reply-content">{reply.content}</div>
        </div>
      {/each}
    </div>
  {/if}

  {#if canComment && !comment.resolved}
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div
      class="reply-form"
      class:reply-form-active={replyFocused || replyText.trim()}
      onclick={(e: MouseEvent) => e.stopPropagation()}
    >
      <textarea
        bind:this={replyTextarea}
        bind:value={replyText}
        placeholder="Reply..."
        rows="1"
        onclick={() => onSelect?.(comment.id)}
        onfocus={() => (replyFocused = true)}
        onblur={() => (replyFocused = false)}
        onkeydown={(e: KeyboardEvent) => {
          if (e.key === "Enter" && !e.shiftKey) {
            e.preventDefault();
            handleSubmitReply();
          }
          if (e.key === "Escape") {
            handleCancelReply();
          }
        }}
      ></textarea>
    </div>
  {/if}
</div>

<style></style>
