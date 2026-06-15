<script lang="ts">
    import type { Component } from "svelte";
    import Check from "@lucide/svelte/icons/check";

    interface CheckboxProps {
        id?: string;
        checked?: boolean;
        disabled?: boolean;
        onchange?: (checked: boolean) => void;
        icon?: Component;
        size?: number;
        class?: string;
        ariaLabel?: string;
    }

    let {
        id = "",
        checked = false,
        disabled = false,
        onchange,
        icon = Check,
        size = 18,
        class: className = "",
        ariaLabel = "",
    }: CheckboxProps = $props();

    function handleChange(event: Event) {
        const target = event.target as HTMLInputElement;
        if (onchange) {
            onchange(target.checked);
        }
    }
</script>

<div class="checkbox-wrapper {className}">
    <input
        type="checkbox"
        {id}
        {checked}
        {disabled}
        onchange={handleChange}
        class="checkbox-input"
        aria-label={ariaLabel}
    />
    <div class="checkbox-custom" style="width: {size}px; height: {size}px;">
        {#if checked}
            {@const IconComponent = icon}
            <IconComponent size={size * 0.75} strokeWidth={3.0} />
        {/if}
    </div>
</div>

<style></style>
