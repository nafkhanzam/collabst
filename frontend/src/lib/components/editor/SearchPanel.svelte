<script lang="ts">
  import CaseSensitive from "@lucide/svelte/icons/case-sensitive";
  import WholeWord from "@lucide/svelte/icons/whole-word";
  import Regex from "@lucide/svelte/icons/regex";
  import ReplaceAll from "@lucide/svelte/icons/replace-all";
  import Replace from "@lucide/svelte/icons/replace";
  import IconButton from "../ui/IconButton.svelte";
  import type { File } from "$lib/types";
  import * as Y from "yjs";
  import { getFileText } from "$lib/yjs";
  import Icon from "../ui/Icon.svelte";

  interface SearchPanelProps {
    caseSensitive?: boolean;
    wholeWord?: boolean;
    regex?: boolean;
    files?: File[];
    ydoc?: Y.Doc | null;
    extraChar?: number;
    gotoMatch?: (
      filePath: string,
      startLine: number,
      startChar: number,
      endLine?: number,
      endChar?: number,
    ) => void;
  }

  let {
    caseSensitive = false,
    wholeWord = false,
    regex = false,
    files = [],
    ydoc = null,
    extraChar = 10,
    gotoMatch = () => {},
  }: SearchPanelProps = $props();

  let searchInput: HTMLInputElement | undefined;
  let replaceInput: HTMLInputElement | undefined;

  // Auto-focus search input when component mounts
  $effect(() => {
    if (searchInput) {
      setTimeout(() => {
        searchInput?.focus();
        searchInput?.select();
      }, 0);
    }
  });

  function handleSearchKeydown(e: KeyboardEvent) {
    if (e.key === "Tab" && !e.shiftKey) {
      e.preventDefault();
      replaceInput?.focus();
    }
  }

  function handleReplaceKeydown(e: KeyboardEvent) {
    if (e.key === "Tab" && e.shiftKey) {
      e.preventDefault();
      searchInput?.focus();
    }
  }

  interface SearchMatch {
    filePath: string;
    startLine: number;
    startChar: number;
    startIndex: number;
    endLine: number;
    endChar: number;
    endIndex: number;
    preMatchText: string;
    matchText: string;
    postMatchText: string;
  }

  let searchResults: SearchMatch[][] = $state([]);

  function caseSensitiveToggle() {
    caseSensitive = !caseSensitive;
    updateSearch();
  }

  function wholeWordToggle() {
    wholeWord = !wholeWord;
    updateSearch();
  }

  function regexToggle() {
    regex = !regex;
    updateSearch();
  }

  function updateSearch() {
    const searchInput = document.getElementById(
      "search-input",
    ) as HTMLInputElement;
    const query = searchInput.value;
    search(query);
  }

  function search(query: string) {
    if (!ydoc) return;
    if (!query) {
      searchResults = [];
      return;
    }

    searchResults = [];
    for (const file of files) {
      const fileResults: SearchMatch[] = [];

      const text = getFileText(ydoc, file.id)?.toString() || "";
      const matches = text.matchAll(
        new RegExp(
          regex
            ? query
            : wholeWord
              ? `\\b${query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")}\\b`
              : query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&"),
          caseSensitive ? "g" : "gi",
        ),
      );

      matches.forEach((matche) => {
        const startIndex = matche.index || 0;
        const endIndex = startIndex + matche[0].length;

        const startLine = text.substring(0, startIndex).split("\n").length - 1;
        const endLine = text.substring(0, endIndex).split("\n").length - 1;

        const startChar =
          text.substring(0, startIndex).split("\n").pop()?.length || 0;
        const endChar =
          text.substring(0, endIndex).split("\n").pop()?.length || 0;

        const extraStartIndex = Math.max(0, startIndex - extraChar);
        const extraEndIndex = Math.min(text.length, endIndex + extraChar);

        let preMatchText = text.substring(extraStartIndex, startIndex);
        const matchText = text.substring(startIndex, endIndex);
        let postMatchText = text.substring(endIndex, extraEndIndex);

        if (extraStartIndex > 0) {
          preMatchText = "…" + preMatchText;
        }
        if (extraEndIndex < text.length) {
          postMatchText = postMatchText + "…";
        }

        fileResults.push({
          filePath: file.path,
          startLine,
          startChar,
          startIndex,
          endLine,
          endChar,
          endIndex,
          preMatchText,
          matchText,
          postMatchText,
        });
      });

      if (fileResults.length > 0) {
        searchResults.push(fileResults);
      }
    }
  }

  function replaceAllInFile(filePath: string) {
    const file = files.find((f) => f.path === filePath);
    if (!file) return;

    if (!ydoc) return;

    const searchInput = document.getElementById(
      "search-input",
    ) as HTMLInputElement;
    const replaceInput = document.getElementById(
      "replace-input",
    ) as HTMLInputElement;
    const query = searchInput.value;
    const replaceText = replaceInput.value;

    const text = getFileText(ydoc, file.id)?.toString() || "";
    const newText = text.replace(
      new RegExp(
        regex
          ? query
          : wholeWord
            ? `\\b${query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")}\\b`
            : query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&"),
        caseSensitive ? "g" : "gi",
      ),
      replaceText,
    );
    const ytext = ydoc.getText(`file-${file.id}`);
    ytext.delete(0, ytext.length);
    ytext.insert(0, newText);

    updateSearch();
  }

  function replaceAll() {
    for (const fileResults of searchResults) {
      replaceAllInFile(fileResults[0].filePath);
    }
  }

  function replaceOne(filePath: string, matchIndex: number) {
    const file = files.find((f) => f.path === filePath);
    if (!file) return;

    if (!ydoc) return;

    const searchInput = document.getElementById(
      "search-input",
    ) as HTMLInputElement;
    const replaceInput = document.getElementById(
      "replace-input",
    ) as HTMLInputElement;
    const query = searchInput.value;
    const replaceText = replaceInput.value;

    const text = getFileText(ydoc, file.id)?.toString() || "";
    const matches = Array.from(
      text.matchAll(
        new RegExp(
          regex
            ? query
            : wholeWord
              ? `\\b${query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")}\\b`
              : query.replace(/[.*+?^${}()|[\]\\]/g, "\\$&"),
          caseSensitive ? "g" : "gi",
        ),
      ),
    );

    if (matchIndex < 0 || matchIndex >= matches.length) return;

    const match = matches[matchIndex];
    const startIndex = match.index || 0;
    const endIndex = startIndex + match[0].length;
    const newEndIndex = startIndex + replaceText.length;

    const ytext = ydoc.getText(`file-${file.id}`);
    ytext.delete(startIndex, endIndex - startIndex);
    ytext.insert(startIndex, replaceText);

    gotoMatch?.(
      filePath,
      text.substring(0, newEndIndex).split("\n").length - 1,
      text.substring(0, newEndIndex).split("\n").pop()?.length || 0,
    );

    updateSearch();
  }
</script>

<div class="placeholder-panel">
  <div class="panel-header">
    <h3>Search & Replace</h3>
  </div>
  <div class="panel-content">
    <div class="search-replace-container">
      <div class="search-area">
        <input
          bind:this={searchInput}
          id="search-input"
          class="search-input"
          type="text"
          placeholder="Search..."
          oninput={updateSearch}
          onkeydown={handleSearchKeydown}
        />
        <IconButton
          variant="flat"
          icon={CaseSensitive}
          size="find-toggle"
          onclick={caseSensitiveToggle}
          title="Case Sensitive"
          selected={caseSensitive}
        />
        <IconButton
          variant="flat"
          icon={WholeWord}
          size="find-toggle"
          onclick={wholeWordToggle}
          title="Match Whole Word"
          selected={wholeWord}
        />
        <IconButton
          variant="flat"
          icon={Regex}
          size="find-toggle"
          onclick={regexToggle}
          title="Use Regular Expressions"
          selected={regex}
        />
      </div>
      <div class="replace-area">
        <input
          bind:this={replaceInput}
          id="replace-input"
          class="replace-input"
          type="text"
          placeholder="Replace..."
          onkeydown={handleReplaceKeydown}
        />
        <IconButton
          variant="replace-all"
          icon={ReplaceAll}
          size="sm"
          onclick={replaceAll}
          title="Replace All"
        />
      </div>
    </div>
    <div class="results">
      {#each searchResults as fileResults}
        <div class="file-results-header">
          <div>
            <strong>{fileResults[0].filePath}</strong> - {fileResults.length}{" "}
            {fileResults.length === 1 ? "match" : "matches"}
          </div>
          <IconButton
            variant="replace-all"
            icon={ReplaceAll}
            size="sm"
            title="Replace All in File"
            onclick={() => replaceAllInFile(fileResults[0].filePath)}
          />
        </div>
        {#each fileResults as result, index}
          <div
            class="result-item"
            role="button"
            tabindex="0"
            onclick={(e) => {
              if ((e.target as HTMLElement).closest(".btn")) {
                return;
              }
              gotoMatch?.(
                result.filePath,
                result.startLine,
                result.startChar,
                result.endLine,
                result.endChar,
              );
            }}
            onkeydown={(e) => {
              if (e.key === "Enter" || e.key === " ") {
                e.preventDefault();
                gotoMatch?.(
                  result.filePath,
                  result.startLine,
                  result.startChar,
                  result.endLine,
                  result.endChar,
                );
              }
            }}
          >
            <div>
              <div class="line-text">
                <span>{result.preMatchText}</span><mark>{result.matchText}</mark
                ><span>{result.postMatchText}</span>
              </div>
              <div class="line-number">
                Line {result.startLine + 1}
              </div>
            </div>
            <IconButton
              class="btn"
              variant="replace-all"
              icon={Replace}
              size="sm"
              title="Replace"
              onclick={() => replaceOne(result.filePath, index)}
            />
          </div>
        {/each}
      {/each}
    </div>
  </div>
</div>

<style></style>
