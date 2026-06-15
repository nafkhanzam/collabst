<script lang="ts">
    import type { Component } from "svelte";
    import { Tooltip } from "$lib/components/ui";

    interface ToggleOption {
        value: string;
        icon: Component;
        label: string;
    }

    interface Props {
        options: ToggleOption[];
        value: string;
        onchange?: (value: string) => void;
    }

    let { options, value = $bindable(), onchange }: Props = $props();
</script>

<div class="icon-toggle">
    {#each options as option}
        <Tooltip text={option.label} position="top">
            <button
                class="toggle-btn"
                class:active={value === option.value}
                onclick={() => {
                    value = option.value;
                    onchange?.(option.value);
                }}
                aria-label={option.label}
            >
                <option.icon size={18} />
            </button>
        </Tooltip>
    {/each}
</div>

<style></style>
