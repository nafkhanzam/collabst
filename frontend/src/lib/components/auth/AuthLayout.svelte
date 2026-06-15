<script lang="ts">
    import { theme } from "$lib/stores/theme";
    import Sun from "@lucide/svelte/icons/sun";
    import Moon from "@lucide/svelte/icons/moon";
    import { browser } from "$app/environment";
    import collabstTextLogo from "../../../assets/collabst-text.svg";
    import type { Snippet } from "svelte";

    interface Props {
        children: Snippet;
    }

    let { children }: Props = $props();

    let currentTheme = $state($theme);

    $effect(() => {
        currentTheme = $theme;
    });

    function toggleTheme() {
        theme.toggle();
    }

    // Calculate number of circles based on viewport size
    let circles = $state<Array<{ delay: number; row: number }>>([]);

    function calculateCircles() {
        if (!browser) return [];

        const circleSize = 50; // Increased from 30 to reduce total circles
        const viewportWidth = window.innerWidth;
        const viewportHeight = window.innerHeight;

        // Only calculate visible circles, no extra padding
        const cols = Math.ceil(viewportWidth / circleSize);
        const rows = Math.ceil(viewportHeight / circleSize);

        const result = [];
        for (let row = 0; row < rows; row++) {
            for (let col = 0; col < cols; col++) {
                result.push({
                    delay: row * 100, // Same delay for all circles in the same row
                    row: row,
                });
            }
        }

        return result;
    }

    $effect(() => {
        if (browser) {
            circles = calculateCircles();

            let resizeTimer: number;
            const handleResize = () => {
                // Debounce resize to avoid excessive recalculations
                clearTimeout(resizeTimer);
                resizeTimer = window.setTimeout(() => {
                    circles = calculateCircles();
                }, 150);
            };

            window.addEventListener("resize", handleResize);

            return () => {
                window.removeEventListener("resize", handleResize);
                clearTimeout(resizeTimer);
            };
        }
    });
</script>

<div class="container" data-theme={currentTheme}>
    <!-- Animated circles background -->
    <div class="circles">
        {#each circles as circle}
            <div
                class="circle"
                style="animation-delay: {circle.delay}ms;"
            ></div>
        {/each}
    </div>

    <!-- Collabst logo in bottom center -->
    <div class="logo-container">
        <img src={collabstTextLogo} alt="collabst" />
    </div>

    <!-- Theme toggle button -->
    <button class="theme-toggle" onclick={toggleTheme} type="button">
        {#if currentTheme === "dark"}
            <Sun size={20} />
        {:else}
            <Moon size={20} />
        {/if}
    </button>

    <div class="card">
        {@render children()}
    </div>
</div>

<style></style>
