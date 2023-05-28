<script setup lang="ts">
import { h, ref } from 'vue';
import { NDropdown, type DropdownOption, NModal, NInput, NButton, useDialog, useMessage, NImage } from 'naive-ui';
import settingSvgUrl from '@/assets/img/setting.svg?url';
import cookies from '@/utils/cookies';
import { usePromptStore } from '@/stores/modules/prompt';
import { storeToRefs } from 'pinia';
import ChatNavItem from './ChatNavItem.vue';
import type { Component } from 'vue';
import { isMobile } from '@/utils/utils';

const isShowMore = ref(false);
const isShowSetTokenModal = ref(false);
const userToken = ref('');
const userTokenCookieName = '_U';
const randIpCookieName = 'BingAI_Rand_IP';
const message = useMessage();
const promptStore = usePromptStore();
const { isShowPromptSotre } = storeToRefs(promptStore);
const isShowClearCacheModal = ref(false);

const navType = {
  github: 'github',
  setToken: 'setToken',
  compose: 'compose',
  promptStore: 'promptStore',
  reset: 'reset',
  version: 'version',
};
const navConfigs = [
  {
    key: navType.github,
    label: '👍赞助支持',
    url: 'https://github-production-user-asset-6210df.s3.amazonaws.com/120257354/238987102-8dd1c2f3-9b0b-44c8-a662-fd619043c0e9.jpg '
  },
  {
    key: navType.version,
    label: '版本信息',
  },
  {
    key: navType.promptStore,
    label: '提示词库',
  },
  {
    key: navType.setToken,
    label: '设置用户',
  },
  {
    key: navType.compose,
    label: '撰写文章',
    url: '/web/compose.html',
  },
  {
    key: navType.reset,
    label: '一键重置',
  },
];

const renderDropdownLabel = (option: DropdownOption) => {
  return h(ChatNavItem as Component, { navConfig: option });
};

const handleSelect = (key: string) => {
  switch (key) {
    case navType.version:
      {
        message.success(`当前版本号为：${__APP_INFO__.version}`);
      }
      break;
    case navType.promptStore:
      isShowPromptSotre.value = true;
      break;
    case navType.setToken:
      {
        userToken.value = cookies.get(userTokenCookieName) || '';
        isShowSetTokenModal.value = true;
      }
      break;
    case navType.reset:
      isShowClearCacheModal.value = true;
      break;
    default:
      break;
  }
};
const resetCache = async () => {
  isShowClearCacheModal.value = false;
  cookies.set(userTokenCookieName, '', -1);
  cookies.set(randIpCookieName, '', -1);
  await clearCache();
  message.success('清理完成');
  window.location.reload();
};

const saveUserToken = () => {
  if (!userToken.value) {
    message.warning('请先填入用户 Cookie');
    return;
  }
  cookies.set(userTokenCookieName, userToken.value, 7 * 24 * 60, '/');
  isShowSetTokenModal.value = false;
};

const clearCache = async () => {
  // del storage
  localStorage.clear();
  sessionStorage.clear();
  // del sw cache
  const cacheKeys = await caches.keys();
  for (const cacheKey of cacheKeys) {
    await caches.open(cacheKey).then(async (cache) => {
      const requests = await cache.keys();
      return await Promise.all(
        requests.map((request) => {
          console.log(`del cache : `, request.url);
          return cache.delete(request);
        })
      );
    });
  }
};
</script>

<template>
  <NDropdown v-if="isMobile()" class="select-none" :show="isShowMore" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
    <NImage class="fixed top-6 right-4 cursor-pointer" :src="settingSvgUrl" alt="设置菜单" :preview-disabled="true" @click="isShowMore = !isShowMore"></NImage>
  </NDropdown>
  <NDropdown v-else class="select-none" trigger="hover" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
    <NImage class="fixed top-6 right-6 cursor-pointer" :src="settingSvgUrl" alt="设置菜单" :preview-disabled="true"></NImage>
  </NDropdown>
  <NModal v-model:show="isShowSetTokenModal" preset="dialog" :show-icon="false">
    <template #header>
      <div class="text-3xl py-2">设置用户</div>
    </template>
    <NInput size="large" v-model:value="userToken" type="text" placeholder="用户 Cookie ,仅需要 _U 的值" />
    <template #action>
      <NButton size="large" @click="isShowSetTokenModal = false">取消</NButton>
      <NButton ghost size="large" type="info" @click="saveUserToken">保存</NButton>
    </template>
  </NModal>
  <NModal v-model:show="isShowClearCacheModal" preset="dialog" :show-icon="false">
    <template #header>
      <div class="text-xl py-2">将删除包括 Cookie 等的所有缓存？</div>
    </template>
    <template #action>
      <NButton size="large" @click="isShowClearCacheModal = false">取消</NButton>
      <NButton ghost size="large" type="error" @click="resetCache">确定</NButton>
    </template>
  </NModal>
</template>
