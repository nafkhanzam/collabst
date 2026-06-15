<script lang="ts">
  import { browser } from "$app/environment";
  import { fade } from "svelte/transition";
  import type { Comment } from "$lib/types";
  import type { UserProfile } from "$lib/types";
  import { usersApi } from "$lib/services/api";
  import CommentThread from "./CommentThread.svelte";

  interface CommentsPanelProps {
    comments?: Comment[];
    currentUserId: string;
    canComment?: boolean;
    canDeleteComments?: boolean;
    newCommentDraft?: {
      text: string;
      range: { from: number; to: number };
      selectedText: string;
    } | null;
    activeCommentId?: string | null;
    hoveredCommentId?: string | null;
    commentPositions?: Map<string, number>;
    editorScrollTop?: number;
    editorContentHeight?: number;
    draftPosition?: number | null;
    onResolve?: (commentId: string) => void;
    onReopen?: (commentId: string) => void;
    onDelete?: (commentId: string) => void;
    onReply?: (commentId: string, content: string) => void;
    onSubmitNew?: (content: string) => void;
    onCancelNew?: () => void;
    onSelect?: (commentId: string) => void;
    onHover?: (commentId: string) => void;
    onHoverEnd?: (commentId: string) => void;
    onPanelScroll?: (scrollTop: number) => void;
  }

  let {
    comments = [],
    currentUserId,
    canComment = true,
    canDeleteComments = false,
    newCommentDraft = null,
    activeCommentId = null,
    hoveredCommentId = null,
    commentPositions = new Map(),
    editorScrollTop = 0,
    editorContentHeight = 2000,
    draftPosition = null,
    onResolve,
    onReopen,
    onDelete,
    onReply,
    onSubmitNew,
    onCancelNew,
    onSelect,
    onHover,
    onHoverEnd,
    onPanelScroll,
  }: CommentsPanelProps = $props();

  let showResolved = $state(
    browser && localStorage.getItem("editor.comments.showResolved") !== null
      ? localStorage.getItem("editor.comments.showResolved") === "true"
      : false,
  );
  let draftCommentText = $state("");
  let userProfiles = $state<Record<string, UserProfile>>({});
  let requestedUserIds = $state<Set<string>>(new Set());
  let panelScrollEl: HTMLElement | undefined = $state();
  let threadHeights = $state<Map<string, number>>(new Map());
  let isSyncingScroll = false;

  let visibleComments = $derived(
    comments.filter((c) => showResolved || !c.resolved),
  );
  let resolvedCount = $derived(comments.filter((c) => c.resolved).length);
  let isEmptyStateVisible = $derived(
    visibleComments.length === 0 && !newCommentDraft,
  );

  $effect(() => {
    if (browser) {
      localStorage.setItem(
        "editor.comments.showResolved",
        String(showResolved),
      );
    }
  });

  $effect(() => {
    const userIds = new Set<string>();
    for (const comment of comments) {
      userIds.add(comment.authorId);
      for (const reply of comment.replies) {
        userIds.add(reply.authorId);
      }
    }

    const toFetch = [...userIds].filter(
      (id) => !!id && !userProfiles[id] && !requestedUserIds.has(id),
    );
    if (!toFetch.length) return;

    requestedUserIds = new Set([...requestedUserIds, ...toFetch]);

    Promise.all(
      toFetch.map(async (id) => {
        try {
          const profile = await usersApi.getProfile(id);
          return { id, profile };
        } catch {
          return null;
        }
      }),
    ).then((results) => {
      const nextProfiles = { ...userProfiles };
      for (const item of results) {
        if (item?.profile) {
          nextProfiles[item.id] = item.profile;
        }
      }
      userProfiles = nextProfiles;
    });
  });

  // Focus and clear when draft changes
  $effect(() => {
    if (newCommentDraft && canComment) {
      draftCommentText = "";
      setTimeout(() => {
        const textarea = document.querySelector(
          ".new-comment-textarea",
        ) as HTMLTextAreaElement;
        if (textarea) textarea.focus();
      }, 0);
    }
  });

  // Sync panel scroll from editor scroll (one-way: editor → panel)
  $effect(() => {
    if (panelScrollEl && editorScrollTop != null) {
      if (isEmptyStateVisible) {
        panelScrollEl.scrollTop = 0;
        return;
      }

      // Guard to avoid loop when we're already syncing from panel → editor
      isSyncingScroll = true;
      panelScrollEl.scrollTop = editorScrollTop;
      requestAnimationFrame(() => {
        isSyncingScroll = false;
      });
    }
  });

  function handlePanelScroll(e: Event) {
    if (isSyncingScroll) return;
    const target = e.target as HTMLElement;
    onPanelScroll?.(target.scrollTop);
  }

  // Compute positioned comments with overlap resolution.
  let positionedComments = $derived.by(() => {
    const MIN_GAP = 4;

    type PositionedItem = {
      type: "draft" | "comment";
      comment?: Comment;
      id: string;
      idealTop: number;
      actualTop: number;
      height: number;
    };

    const items: PositionedItem[] = [];

    // Add draft if present
    if (canComment && newCommentDraft && draftPosition != null) {
      items.push({
        type: "draft",
        id: "__draft__",
        idealTop: draftPosition,
        actualTop: draftPosition,
        height: threadHeights.get("__draft__") || 120,
      });
    }

    const sortedComments = [...visibleComments].sort((a, b) => {
      if (a.line !== b.line) return a.line - b.line;
      return a.createdAt.localeCompare(b.createdAt);
    });

    // Skip comments without a mapped position (not yet computed)
    for (const comment of sortedComments) {
      const mappedPos = commentPositions.get(comment.id);
      if (mappedPos == null) continue;

      items.push({
        type: "comment",
        comment,
        id: comment.id,
        idealTop: mappedPos,
        actualTop: mappedPos,
        height: threadHeights.get(comment.id) || 80,
      });
    }

    // Sort by ideal position
    items.sort((a, b) => a.idealTop - b.idealTop);

    // Find the interactive item that must stay accessible (clamped to >= 0).
    // Draft takes priority (user is typing a new comment), then active comment
    // (user may be typing a reply).
    let anchorIdx = items.findIndex((item) => item.type === "draft");
    if (anchorIdx < 0) {
      anchorIdx = items.findIndex((item) => item.id === activeCommentId);
    }

    if (anchorIdx >= 0) {
      // Anchor item stays at its ideal position, clamped to >= 0
      // so the user can always interact with it
      items[anchorIdx].actualTop = Math.max(0, items[anchorIdx].idealTop);

      // Push items BELOW the anchor downward
      for (let i = anchorIdx + 1; i < items.length; i++) {
        const prev = items[i - 1];
        const minTop = prev.actualTop + prev.height + MIN_GAP;
        if (items[i].actualTop < minTop) {
          items[i].actualTop = minTop;
        }
      }

      // Push items ABOVE the anchor upward — they CAN go out of bounds
      // (negative top). Non-interactive comments can be pushed off-screen
      // to keep the anchor item in place.
      for (let i = anchorIdx - 1; i >= 0; i--) {
        const next = items[i + 1];
        const maxBottom = next.actualTop - MIN_GAP;
        if (items[i].actualTop + items[i].height > maxBottom) {
          items[i].actualTop = maxBottom - items[i].height;
        }
      }
    } else {
      // No active comment — simple top-down push
      for (let i = 1; i < items.length; i++) {
        const prev = items[i - 1];
        const minTop = prev.actualTop + prev.height + MIN_GAP;
        if (items[i].actualTop < minTop) {
          items[i].actualTop = minTop;
        }
      }
    }

    return items;
  });

  function handleSubmitNewComment() {
    if (draftCommentText.trim()) {
      onSubmitNew?.(draftCommentText.trim());
      draftCommentText = "";
    }
  }

  function handleCancelNewComment() {
    draftCommentText = "";
    onCancelNew?.();
  }

  function measureThread(el: HTMLElement, id: string) {
    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        const h =
          entry.borderBoxSize?.[0]?.blockSize ?? entry.contentRect.height;
        threadHeights.set(id, h);
        threadHeights = new Map(threadHeights);
      }
    });
    observer.observe(el);
    return {
      destroy() {
        observer.disconnect();
      },
    };
  }


</script>

<div class="comments-panel">
  {#if resolvedCount > 0}
    <div class="filter-section">
      <label class="filter-toggle">
        <input
          class="filter-checkbox"
          type="checkbox"
          bind:checked={showResolved}
        />
        <span class="filter-label">Show resolved ({resolvedCount})</span>
      </label>
    </div>
  {/if}

  <div
    class="panel-scroll"
    bind:this={panelScrollEl}
    onscroll={handlePanelScroll}
  >
    <div
      class="panel-content"
      style={isEmptyStateVisible
        ? undefined
        : `height: ${editorContentHeight}px;`}
    >
      {#each positionedComments as item (item.id)}
        {#if item.type === "draft"}
          <div
            class="positioned-thread"
            style="top: {item.actualTop}px;"
            use:measureThread={"__draft__"}
            in:fade={{ duration: 150 }}
          >
            <div class="new-comment-draft">
              <textarea
                class="new-comment-textarea"
                bind:value={draftCommentText}
                placeholder="Add your comment..."
                rows="2"
                onkeydown={(e: KeyboardEvent) => {
                  if (e.key === "Enter" && !e.shiftKey) {
                    e.preventDefault();
                    handleSubmitNewComment();
                  }
                  if (e.key === "Escape") {
                    handleCancelNewComment();
                  }
                }}
              ></textarea>
              <div class="draft-actions">
                <button class="btn btn-cancel" onclick={handleCancelNewComment}
                  >Cancel</button
                >
                <button
                  class="btn btn-submit"
                  onclick={handleSubmitNewComment}
                  disabled={!draftCommentText.trim()}
                >
                  Comment
                </button>
              </div>
            </div>
          </div>
        {:else if item.comment}
          <div
            class="positioned-thread"
            style="top: {item.actualTop}px;"
            use:measureThread={item.id}
            in:fade={{ duration: 150 }}
          >
            <CommentThread
              comment={item.comment}
              {userProfiles}
              {currentUserId}
              {canComment}
              {canDeleteComments}
              isActive={item.id === activeCommentId}
              isHovered={item.id === hoveredCommentId}
              {onResolve}
              {onReopen}
              {onDelete}
              {onReply}
              {onSelect}
              {onHover}
              {onHoverEnd}
            />
          </div>
        {/if}
      {/each}

      {#if visibleComments.length === 0 && !(canComment && newCommentDraft)}
        <div class="empty-state">
          <div class="empty-icon">💬</div>
          <p>No comments yet</p>
          <span
            >{canComment
              ? "Select text to add a comment"
              : "You have read-only access to comments"}</span
          >
        </div>
      {/if}
    </div>
  </div>
</div>

<style></style>
