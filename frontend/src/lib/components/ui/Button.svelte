<script lang="ts">
  import type { Snippet } from "svelte";

  type ButtonVariant = "primary" | "secondary" | "ghost" | "danger" | "success";
  type ButtonSize = "sm" | "md" | "lg" | "top-bar";

  interface ButtonProps {
    variant?: ButtonVariant;
    size?: ButtonSize;
    disabled?: boolean;
    fullWidth?: boolean;
    type?: "button" | "submit" | "reset";
    onclick?: () => void;
    class?: string;
    children?: Snippet;
  }

  let {
    variant = "primary",
    size = "md",
    disabled = false,
    fullWidth = false,
    type = "button",
    onclick,
    class: className = "",
    children,
  }: ButtonProps = $props();
</script>

<button
  {type}
  {disabled}
  class="btn btn-{variant} btn-{size} {fullWidth ? 'btn-full' : ''} {className}"
  {onclick}
>
  {@render children?.()}
</button>

<style></style>
