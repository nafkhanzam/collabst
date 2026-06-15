<script lang="ts">
  import type { Snippet } from 'svelte'
  
  interface InputProps {
    type?: 'text' | 'email' | 'password' | 'number' | 'url' | 'search'
    value?: string
    placeholder?: string
    disabled?: boolean
    required?: boolean
    fullWidth?: boolean
    error?: string
    label?: string
    class?: string
    autofocus?: boolean
    inputElement?: HTMLInputElement
    oninput?: (e: Event) => void
    onchange?: (e: Event) => void
  }
  
  let {
    type = 'text',
    value = $bindable(''),
    placeholder = '',
    disabled = false,
    required = false,
    fullWidth = false,
    error = '',
    label = '',
    class: className = '',
    autofocus = false,
    inputElement = $bindable(undefined),
    oninput,
    onchange
  }: InputProps = $props()
</script>

<div class="input-wrapper {fullWidth ? 'input-full' : ''} {className}">
  {#if label}
    <label class="input-label">
      {label}
      {#if required}<span class="required">*</span>{/if}
    </label>
  {/if}
  
  <input
    bind:this={inputElement}
    {type}
    bind:value
    {placeholder}
    {disabled}
    {required}
    autofocus={autofocus}
    class="input {error ? 'input-error' : ''}"
    {oninput}
    {onchange}
  />
  
  {#if error}
    <span class="error-message">{error}</span>
  {/if}
</div>

<style></style>
