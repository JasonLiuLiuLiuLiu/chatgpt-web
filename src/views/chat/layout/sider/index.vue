<script setup lang='ts'>
import type { CSSProperties } from 'vue'
import { computed, ref, watch } from 'vue'
import { NButton, NLayoutSider } from 'naive-ui'
import List from './List.vue'
import Footer from './Footer.vue'
import { useAppStore, useAuthStore, useChatStore } from '@/store'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { HoverButton, PromptStore, SvgIcon } from '@/components/common'

const appStore = useAppStore()
const authStore = useAuthStore()
const chatStore = useChatStore()

const { isMobile } = useBasicLayout()
const show = ref(false)

const collapsed = computed(() => appStore.siderCollapsed)

async function handleAdd() {
  await chatStore.addHistory({ title: 'New Chat', uuid: Date.now(), isEdit: false, usingContext: true })
  if (isMobile.value)
    appStore.setSiderCollapsed(true)
}

function handleUpdateCollapsed() {
  appStore.setSiderCollapsed(!collapsed.value)
}

function openUrl(url: string | URL | undefined) {
  window.open(url)
}

const getMobileClass = computed<CSSProperties>(() => {
  if (isMobile.value) {
    return {
      position: 'fixed',
      zIndex: 50,
    }
  }
  return {}
})

const mobileSafeArea = computed(() => {
  if (isMobile.value) {
    return {
      paddingBottom: 'env(safe-area-inset-bottom)',
    }
  }
  return {}
})

watch(
  isMobile,
  (val) => {
    appStore.setSiderCollapsed(val)
  },
  {
    immediate: true,
    flush: 'post',
  },
)
</script>

<template>
  <NLayoutSider
    :collapsed="collapsed"
    :collapsed-width="0"
    :width="260"
    :show-trigger="isMobile ? false : 'arrow-circle'"
    collapse-mode="transform"
    position="absolute"
    bordered
    :style="getMobileClass"
    @update-collapsed="handleUpdateCollapsed"
  >
    <div class="flex flex-col h-full" :style="mobileSafeArea">
      <main class="flex flex-col flex-1 min-h-0">
        <div class="p-4">
          <NButton dashed block :disabled="!!authStore.session?.auth && !authStore.token" @click="handleAdd">
            {{ $t('chat.newChatButton') }}
          </NButton>
        </div>
        <div class="flex-1 min-h-0 pb-4 overflow-hidden">
          <List />
        </div>
        <div class="flex items-center justify-between min-w-0 p-4 overflow-hidden dark:border-neutral-800">
          <HoverButton :tooltip="$t('store.siderButton')" @click="show = true">
            <span class="text-xl text-[#4f555e] dark:text-white">
              <SvgIcon icon="solar:shop-broken" />
            </span>
          </HoverButton>
          <HoverButton tooltip="new bing" @click="openUrl('https://bing.jason.hi.cn')">
            <span class="text-xl text-[#4f555e] dark:text-white">
              <SvgIcon icon="jam:bing-circle" />
            </span>
          </HoverButton>
          <HoverButton tooltip="next web" @click="openUrl('https://vercel.jason.hi.cn')">
            <span class="text-xl text-[#4f555e] dark:text-white">
              <SvgIcon icon="fluent:bot-sparkle-20-regular" />
            </span>
          </HoverButton>
          <HoverButton tooltip="one api" @click="openUrl('https://one-api.jason.hi.cn')">
            <span class="text-xl text-[#4f555e] dark:text-white">
              <SvgIcon icon="icon-park-twotone:api" />
            </span>
          </HoverButton>
          <!-- <NButton block @click="show = true">
            {{ $t('store.siderButton') }}
          </NButton> -->
        </div>
      </main>
      <Footer />
    </div>
  </NLayoutSider>
  <template v-if="isMobile">
    <div v-show="!collapsed" class="fixed inset-0 z-40 w-full h-full bg-black/40" @click="handleUpdateCollapsed" />
  </template>
  <PromptStore v-model:visible="show" />
</template>
