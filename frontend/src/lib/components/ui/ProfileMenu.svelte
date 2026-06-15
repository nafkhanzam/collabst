<script lang="ts">
  import { auth } from "$lib/stores/auth";
  import { Tooltip, UserSettingsModal } from "$lib/components/ui";
  import CircleUser from "@lucide/svelte/icons/circle-user";
  import LogOut from "@lucide/svelte/icons/log-out";
  import { getProfilePicUrl } from "$lib/utils/urls";

  let showMenu = $state(false);
  let showSettingsModal = $state(false);
  let avatarLoaded = $state(false);

  $effect(() => {
    $auth.user?.id;
    avatarLoaded = false;
  });

  function toggleMenu() {
    showMenu = !showMenu;
  }

  function handleLogout() {
    showMenu = false;
    auth.logout();
  }

  function handleOpenSettings() {
    if (!$auth.user) return;
    showMenu = false;
    showSettingsModal = true;
  }

  function profilePicSrc() {
    if (!$auth.user?.id) return "";
    return getProfilePicUrl($auth.user.id);
  }

  function handleClickOutside(event: MouseEvent) {
    const target = event.target as HTMLElement;
    if (!target.closest(".profile-menu-container")) {
      showMenu = false;
    }
  }

  $effect(() => {
    if (showMenu) {
      document.addEventListener("click", handleClickOutside);
      return () => {
        document.removeEventListener("click", handleClickOutside);
      };
    }
  });
</script>

<div class="profile-menu-container">
  <Tooltip text="Profile menu" position="bottom">
    <button
      class="avatar-trigger"
      onclick={toggleMenu}
      aria-label="Open profile menu"
    >
      <span class="avatar-fallback" class:avatar-fallback-hidden={avatarLoaded}>
        {($auth.user?.display_name || "U")[0].toUpperCase()}
      </span>
      {#if $auth.user?.id}
        <img
          class="avatar-image avatar-image-trigger"
          class:avatar-image-loaded={avatarLoaded}
          src={profilePicSrc()}
          alt="User avatar"
          onload={() => (avatarLoaded = true)}
          onerror={() => (avatarLoaded = false)}
        />
      {/if}
    </button>
  </Tooltip>

  {#if showMenu}
    <div class="profile-dropdown">
      <div class="profile-info">
        <span>{$auth.user?.display_name || "User"}</span>
      </div>
      <div class="divider"></div>
      <button class="menu-item" onclick={handleOpenSettings}>
        <CircleUser size={16} />
        <span>Settings</span>
      </button>
      <button class="menu-item" onclick={handleLogout}>
        <LogOut size={16} />
        <span>Logout</span>
      </button>
    </div>
  {/if}

  <UserSettingsModal bind:open={showSettingsModal} />
</div>

<style></style>
