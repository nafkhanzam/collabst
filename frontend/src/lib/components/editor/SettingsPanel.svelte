<script lang="ts">
  import { editorSettings } from "$lib/stores/editorSettings";
  import RotateCcw from "@lucide/svelte/icons/rotate-ccw";
  import ChevronUp from "@lucide/svelte/icons/chevron-up";
  import ChevronDown from "@lucide/svelte/icons/chevron-down";
  import Ligature from "@lucide/svelte/icons/ligature";
  import Checkbox from "../ui/Checkbox.svelte";

  let fontSize = $state($editorSettings.fontSize);
  let fontFamily = $state($editorSettings.fontFamily);
  let ligatures = $state($editorSettings.ligatures);

  // Sync local state with store
  $effect(() => {
    fontSize = $editorSettings.fontSize;
  });

  $effect(() => {
    fontFamily = $editorSettings.fontFamily;
  });

  $effect(() => {
    ligatures = $editorSettings.ligatures;
  });

  function handleFontSizeInput(event: Event) {
    const target = event.target as HTMLInputElement;
    const value = target.value;

    // Only allow digits
    const numericValue = value.replace(/[^\d]/g, "");

    if (numericValue !== value) {
      target.value = numericValue;
    }

    if (numericValue) {
      const parsed = parseInt(numericValue, 10);
      if (!isNaN(parsed) && parsed > 0 && parsed <= 999) {
        fontSize = parsed;
        editorSettings.setFontSize(parsed);
      }
    }
  }

  function incrementFontSize() {
    const newSize = Math.min(fontSize + 1, 999);
    fontSize = newSize;
    editorSettings.setFontSize(newSize);
  }

  function decrementFontSize() {
    const newSize = Math.max(fontSize - 1, 1);
    fontSize = newSize;
    editorSettings.setFontSize(newSize);
  }

  function handleFontFamilyInput(event: Event) {
    const target = event.target as HTMLInputElement;
    fontFamily = target.value;
    editorSettings.setFontFamily(target.value);
  }

  function resetFontSize() {
    editorSettings.resetFontSize();
  }

  function resetFontFamily() {
    editorSettings.resetFontFamily();
  }

  function toggleLigatures(checked: boolean) {
    ligatures = checked;
    editorSettings.setLigatures(checked);
  }
</script>

<div class="settings-panel">
  <div class="panel-header">
    <h3>Settings</h3>
  </div>
  <div class="panel-content">
    <section class="settings-section">
      <h4>Editor Settings</h4>

      <div class="setting-item">
        <div class="setting-row">
          <label for="font-size">Font size</label>
          <div class="input-with-controls">
            <input
              id="font-size"
              type="text"
              inputmode="numeric"
              value={fontSize}
              oninput={handleFontSizeInput}
              class="font-size-input"
            />
            <div class="increment-buttons">
              <button
                class="increment-btn"
                onclick={incrementFontSize}
                aria-label="Increase font size"
              >
                <ChevronUp size={14} />
              </button>
              <button
                class="increment-btn"
                onclick={decrementFontSize}
                aria-label="Decrease font size"
              >
                <ChevronDown size={14} />
              </button>
            </div>
            <button
              class="reset-btn"
              onclick={resetFontSize}
              aria-label="Reset font size to default"
            >
              <RotateCcw size={16} />
            </button>
          </div>
        </div>
      </div>

      <div class="setting-item">
        <label for="font-family">Font family</label>
        <div class="input-with-reset">
          <input
            id="font-family"
            type="text"
            value={fontFamily}
            oninput={handleFontFamilyInput}
            class="font-family-input"
            placeholder='"JetBrains Mono", monospace'
          />
          <button
            class="reset-btn"
            onclick={resetFontFamily}
            aria-label="Reset font family to default"
          >
            <RotateCcw size={16} />
          </button>
        </div>
      </div>

      <div class="setting-item">
        <div class="setting-row">
          <label class="checkbox-label" for="ligatures">
            <Ligature size={16} />
            <span>Ligatures</span>
          </label>
          <Checkbox
            id="ligatures"
            checked={ligatures}
            onchange={toggleLigatures}
          />
        </div>
      </div>
    </section>
  </div>
</div>

<style></style>
