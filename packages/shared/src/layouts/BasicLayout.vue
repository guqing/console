<script lang="ts" setup>
import {
  IconMore,
  IconSearch,
  IconUserSettings,
  VRoutesMenu,
  VTag,
  VAvatar,
  VSpace,
  VButton,
  useDialog,
} from "@halo-dev/components";
import type { MenuGroupType, MenuItemType } from "../types/menus";
import type { User } from "@halo-dev/api-client";
import logo from "@/assets/logo.svg";
import { RouterView, useRoute, useRouter } from "vue-router";
import { computed, inject, ref, type Ref } from "vue";
import axios from "axios";

const menus = inject<MenuGroupType[]>("menus");
const minimenus = inject<MenuItemType[]>("minimenus");
const route = useRoute();
const router = useRouter();
const dialog = useDialog();

const moreMenuVisible = ref(false);
const moreMenuRootVisible = ref(false);

const currentUser = inject<User>("currentUser");
const apiUrl = inject<string>("apiUrl");

const handleRouteToProfile = () => {
  router.push({ path: `/users/${currentUser?.metadata.name}/detail` });
};

const handleLogout = () => {
  dialog.warning({
    title: "是否确认退出登录？",
    onConfirm: async () => {
      try {
        await axios.post(`${apiUrl}/logout`, undefined, {
          withCredentials: true,
        });
        router.replace({ name: "Login" });
      } catch (error) {
        console.error("Failed to logout", error);
      }
    },
  });
};

const currentRole = computed(() => {
  return JSON.parse(
    currentUser?.metadata.annotations?.[
      "rbac.authorization.halo.run/role-names"
    ] || "[]"
  )[0];
});

const globalSearchVisible = inject<Ref<boolean>>(
  "globalSearchVisible",
  ref(false)
);

const isMac = /macintosh|mac os x/i.test(navigator.userAgent);
</script>

<template>
  <div class="flex h-full">
    <aside class="navbar fixed hidden h-full overflow-y-auto md:block">
      <div class="logo flex justify-center pt-5 pb-7">
        <img :src="logo" alt="Halo Logo" style="width: 78px" />
      </div>
      <div class="px-3">
        <div
          class="flex cursor-pointer items-center rounded bg-gray-100 p-2 text-gray-400 transition-all hover:text-gray-900"
          @click="globalSearchVisible = true"
        >
          <span class="mr-3">
            <IconSearch />
          </span>
          <span class="flex-1 select-none text-base font-normal">搜索</span>
          <div class="text-sm">
            {{ `${isMac ? "⌘" : "Ctrl"}+K` }}
          </div>
        </div>
      </div>
      <VRoutesMenu :menus="menus" />
      <div class="current-profile">
        <div v-if="currentUser?.spec.avatar" class="profile-avatar">
          <VAvatar
            :src="currentUser?.spec.avatar"
            :alt="currentUser?.spec.displayName"
            size="md"
            circle
          ></VAvatar>
        </div>
        <div class="profile-name">
          <div class="flex text-sm font-medium">
            {{ currentUser?.spec.displayName }}
          </div>
          <div class="flex">
            <VTag>
              <template #leftIcon>
                <IconUserSettings />
              </template>
              {{ currentRole }}
            </VTag>
          </div>
        </div>
        <FloatingDropdown
          class="profile-control cursor-pointer rounded p-1 transition-all hover:bg-gray-100"
        >
          <IconMore />
          <template #popper>
            <div class="w-48 p-2">
              <VSpace class="w-full" direction="column">
                <VButton
                  v-close-popper
                  block
                  type="secondary"
                  @click="handleRouteToProfile"
                >
                  个人资料
                </VButton>
                <VButton
                  v-close-popper
                  block
                  type="default"
                  @click="handleLogout"
                >
                  退出登录
                </VButton>
              </VSpace>
            </div>
          </template>
        </FloatingDropdown>
      </div>
    </aside>
    <main class="content w-full overflow-y-auto pb-12 mb-safe md:pb-0">
      <slot v-if="$slots.default" />
      <RouterView v-else />
    </main>

    <!--bottom nav bar-->
    <div
      v-if="minimenus"
      class="bottom-nav-bar fixed left-0 bottom-0 right-0 grid grid-cols-6 border-t-2 border-black drop-shadow-2xl mt-safe pb-safe md:hidden bg-secondary"
    >
      <div
        v-for="(menu, index) in minimenus"
        :key="index"
        :class="{ 'bg-black': route.path === menu?.path }"
        class="nav-item"
        @click="router.push(menu?.path)"
      >
        <div
          class="flex w-full cursor-pointer items-center justify-center p-1 text-white"
        >
          <div
            class="is-active is-active0 flex h-10 w-10 flex-col items-center justify-center"
          >
            <div class="text-base">
              <Component :is="menu?.icon" />
            </div>
            <div class="mt-0.5 text-xs">
              {{ menu?.name }}
            </div>
          </div>
        </div>
      </div>
      <div class="nav-item" @click="moreMenuVisible = true">
        <div
          class="flex w-full cursor-pointer items-center justify-center p-1 text-white"
        >
          <div
            class="is-active is-active0 flex h-10 w-10 flex-col items-center justify-center"
          >
            <div class="text-base">
              <IconMore />
            </div>
            <div class="mt-0.5 text-xs">更多</div>
          </div>
        </div>
      </div>

      <Teleport to="body">
        <div
          v-show="moreMenuRootVisible"
          class="drawer-wrapper fixed top-0 left-0 z-[99999] flex h-full w-full flex-row items-end justify-center"
        >
          <transition
            enter-active-class="ease-out duration-200"
            enter-from-class="opacity-0"
            enter-to-class="opacity-100"
            leave-active-class="ease-in duration-100"
            leave-from-class="opacity-100"
            leave-to-class="opacity-0"
            @before-enter="moreMenuRootVisible = true"
            @after-leave="moreMenuRootVisible = false"
          >
            <div
              v-show="moreMenuVisible"
              class="drawer-layer absolute top-0 left-0 h-full w-full flex-none bg-gray-500 bg-opacity-75 transition-opacity"
              @click="moreMenuVisible = false"
            ></div>
          </transition>
          <transition
            enter-active-class="transform transition ease-in-out duration-500 sm:duration-700"
            enter-from-class="translate-y-full"
            enter-to-class="translate-y-0"
            leave-active-class="transform transition ease-in-out duration-500 sm:duration-700"
            leave-from-class="translate-y-0"
            leave-to-class="translate-y-full"
          >
            <div
              v-show="moreMenuVisible"
              class="drawer-content relative flex h-3/4 w-screen flex-col items-stretch overflow-y-auto rounded-t-md bg-white shadow-xl"
            >
              <div class="drawer-body">
                <VRoutesMenu
                  :menus="menus"
                  class="p-0"
                  @select="moreMenuVisible = false"
                />
              </div>
            </div>
          </transition>
        </div>
      </Teleport>
    </div>
  </div>
</template>

<style lang="scss">
.navbar {
  @apply w-64;
  @apply bg-white;
  z-index: 999;
  box-shadow: 0 4px 4px #f6c6ce;
  padding-bottom: 70px;

  .current-profile {
    height: 70px;
    @apply w-64
    bg-white
    p-3
    flex
    fixed
    left-0
    bottom-0
    gap-3;

    .profile-avatar {
      @apply self-center
      flex 
      items-center;
    }

    .profile-name {
      @apply self-center
      flex-1;
    }

    .profile-control {
      @apply self-center;
    }
  }
}

.content {
  @apply ml-0
  flex
  flex-auto
  flex-col
  overflow-x-hidden
  md:ml-64;
}
</style>
