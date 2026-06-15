<script lang="ts">
    import ChevronsUpDown from "@lucide/svelte/icons/chevrons-up-down";
    import Check from "@lucide/svelte/icons/check";

    interface DropdownOption {
        value: string;
        label: string;
    }

    interface Props {
        options: DropdownOption[];
        value: string;
        onchange?: (value: string) => void;
    }

    let { options, value = $bindable(), onchange }: Props = $props();
    let isOpen = $state(false);
    let buttonRef = $state<HTMLButtonElement>();

    function toggleDropdown() {
        isOpen = !isOpen;
    }

    function selectOption(optionValue: string) {
        value = optionValue;
        isOpen = false;
        onchange?.(optionValue);
    }

    function handleClickOutside(event: MouseEvent) {
        if (buttonRef && !buttonRef.contains(event.target as Node)) {
            isOpen = false;
        }
    }

    $effect(() => {
        if (isOpen) {
            document.addEventListener("click", handleClickOutside);
            return () =>
                document.removeEventListener("click", handleClickOutside);
        }
    });

    const currentLabel = $derived(
        options.find((opt) => opt.value === value)?.label || "",
    );
</script>

<div class="dropdown-container">
    <button
        bind:this={buttonRef}
        class="dropdown-btn"
        onclick={toggleDropdown}
        aria-expanded={isOpen}
    >
        <span class="label">{currentLabel}</span>
        <ChevronsUpDown size={16} style="color: var(--color-primary-500);" />
    </button>

    {#if isOpen}
        <div class="dropdown-menu">
            {#each options as option}
                <button
                    class="dropdown-item"
                    class:active={option.value === value}
                    onclick={() => selectOption(option.value)}
                >
                    <span class="dropdown-item-text">{option.label}</span>
                    {#if option.value === value}
                        <Check size={16} class="check-icon" />
                    {/if}
                </button>
            {/each}
        </div>
    {/if}
</div>

<style></style>
