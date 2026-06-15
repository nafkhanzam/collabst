<script lang="ts">
  import type { Component } from "svelte";
  import Icon from "./Icon.svelte";

  type ButtonVariant =
    | "primary"
    | "secondary"
    | "ghost"
    | "danger"
    | "success"
    | "flat"
    | "top-bar"
    | "replace-all";
  type ButtonSize = "sm" | "md" | "lg" | "top-bar" | "find-toggle";

  interface IconButtonProps {
    icon: Component;
    variant?: ButtonVariant;
    size?: ButtonSize;
    disabled?: boolean;
    title?: string;
    onclick?: () => void;
    class?: string;
    ariaLabel?: string;
    selected?: boolean;
  }

  let {
    icon,
    variant = "ghost",
    size = "md",
    disabled = false,
    title = "",
    onclick,
    class: className = "",
    ariaLabel = title,
    selected = false,
  }: IconButtonProps = $props();

  const iconSizeMap = {
    sm: 16,
    md: 20,
    lg: 24,
    "top-bar": 18,
    "find-toggle": 16,
    "replace-all": 10,
  };
</script>

<button
  type="button"
  {disabled}
  {title}
  aria-label={ariaLabel}
  class="icon-btn icon-btn-{variant} icon-btn-{size} icon-btn-{selected
    ? 'selected'
    : 'not-selected'} {className}"
  {onclick}
>
  <Icon {icon} size={iconSizeMap[size]} />
</button>

<style></style>
