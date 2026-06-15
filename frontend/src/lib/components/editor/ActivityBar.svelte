<script lang="ts">
  import { Tooltip } from "$lib/components/ui";
  import Notifiable from "$lib/components/ui/Notifiable.svelte";
  import Archive from "@lucide/svelte/icons/archive";
  import Search from "@lucide/svelte/icons/search";
  import Map from "@lucide/svelte/icons/map";
  import CircleAlert from "@lucide/svelte/icons/circle-alert";
  import MessageCircleMore from "@lucide/svelte/icons/message-square-more";
  import CircleHelp from "@lucide/svelte/icons/circle-help";
  import Rocket from "@lucide/svelte/icons/rocket";
  import Settings from "@lucide/svelte/icons/settings";
  import collabstLogo from "../../../assets/collabst-text.svg";
  import type { Diagnostic } from "$lib/types";

  type NotificationType = "error" | "warning" | "info" | "hint" | "comments";
  type IssueSeverity = Exclude<NotificationType, "comments">;

  export let activePanel: string | null = "files";
  export let onActivityClick: (activity: string) => void;
  export let diagnostics: Diagnostic[] = [];
  export let unresolvedCommentsCount = 0;

  type Activity = {
    id: string;
    icon: any;
    label: string;
    href?: string;
  };

  const topActivities: Activity[] = [
    { id: "files", icon: Archive, label: "Files" },
    { id: "search", icon: Search, label: "Search" },
    { id: "outline", icon: Map, label: "Outline" },
    { id: "issues", icon: CircleAlert, label: "Issues and Suggestions" },
    { id: "comments", icon: MessageCircleMore, label: "Comments" },
  ];

  const bottomActivities: Activity[] = [
    { id: "settings", icon: Settings, label: "Settings" },
    {
      id: "universe",
      icon: Rocket,
      label: "Typst Universe",
      href: "https://typst.app/universe",
    },
    {
      id: "help",
      icon: CircleHelp,
      label: "Help",
      href: "https://typst.app/docs",
    },
  ];

  function handleClick(activity: Activity) {
    if (activity.href) {
      window.open(activity.href, "_blank");
    } else {
      onActivityClick(activity.id);
    }
  }

  let issueNotification: boolean = false;
  $: issueNotification = diagnostics.length > 0;
  let commentsNotification: boolean = false;
  $: commentsNotification = unresolvedCommentsCount > 0;

  const issueSeverities: IssueSeverity[] = ["error", "warning", "info", "hint"];

  function isIssueSeverity(value: string): value is IssueSeverity {
    return issueSeverities.includes(value as IssueSeverity);
  }

  function severityValue(severity: IssueSeverity): number {
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

  let issueSeverity: IssueSeverity = "info";
  $: issueSeverity =
    diagnostics
      .map((d) => d.severity)
      .filter((severity): severity is IssueSeverity =>
        isIssueSeverity(severity),
      )
      .sort((a, b) => severityValue(a) - severityValue(b))[0] ?? "info";
</script>

<div class="activity-bar">
  <div class="top-activities">
    {#each topActivities as activity (activity.id)}
      <Tooltip text={activity.label} position="right">
        {#if activity.id === "issues"}
          <button
            class="activity-btn"
            class:active={activePanel === activity.id}
            on:click={() => handleClick(activity)}
            aria-label={activity.label}
          >
            <Notifiable
              hasNotification={issueNotification}
              type={issueSeverity}
              count={diagnostics.length}
            >
              <svelte:component this={activity.icon} size={24} />
            </Notifiable>
          </button>
        {:else if activity.id === "comments"}
          <button
            class="activity-btn"
            class:active={activePanel === activity.id}
            on:click={() => handleClick(activity)}
            aria-label={activity.label}
          >
            <Notifiable
              hasNotification={commentsNotification}
              type="comments"
              count={unresolvedCommentsCount}
            >
              <svelte:component this={activity.icon} size={24} />
            </Notifiable>
          </button>
        {:else}
          <button
            class="activity-btn"
            class:active={activePanel === activity.id}
            on:click={() => handleClick(activity)}
            aria-label={activity.label}
          >
            <svelte:component this={activity.icon} size={24} />
          </button>
        {/if}
      </Tooltip>
    {/each}
  </div>

  <div class="bottom-activities">
    {#each bottomActivities as activity (activity.id)}
      <Tooltip text={activity.label} position="right">
        {#if activity.href}
          <a
            class="activity-btn"
            href={activity.href}
            target="_blank"
            rel="noopener noreferrer"
            aria-label={activity.label}
          >
            <svelte:component this={activity.icon} size={24} />
          </a>
        {:else}
          <button
            class="activity-btn"
            class:active={activePanel === activity.id}
            on:click={() => handleClick(activity)}
            aria-label={activity.label}
          >
            <svelte:component this={activity.icon} size={24} />
          </button>
        {/if}
      </Tooltip>
    {/each}

    <div class="logo-container">
      <img src={collabstLogo} alt="collabst" class="collabst-logo" />
    </div>
  </div>
</div>

<style></style>
