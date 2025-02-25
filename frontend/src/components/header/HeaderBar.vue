<template>
  <header>
    <div
      v-if="showLogo"
      class="logo-container"
      :style="{
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }"
    >
      <a href="https://pras.me" target="_blank">
        <img
          title="PRAS"
          v-if="showLogo"
          :src="prasLogo"
          alt="pras logo"
          :style="{ marginTop: '-10px', height: '22px' }"
        />
      </a>

      <div
        v-if="showLogo"
        :style="{
          width: '2px',
          height: '50px',
          backgroundColor: '#636363',
          borderRadius: '9999px',
          rotate: '25deg',
          marginLeft: '-5px',
          marginRight: '15px',
        }"
      ></div>
      <a href="/">
        <img
          title="Web File Manager"
          v-if="showLogo"
          :src="logoURL"
          :style="{ height: '35px', marginTop: '20px' }"
        />
      </a>
    </div>
    <Action
      v-if="showMenu"
      class="menu-button"
      icon="menu"
      :label="t('buttons.toggleSidebar')"
      @action="layoutStore.showHover('sidebar')"
    />

    <slot />

    <div
      id="dropdown"
      :class="{ active: layoutStore.currentPromptName === 'more' }"
    >
      <slot name="actions" />
    </div>

    <Action
      v-if="ifActionsSlot"
      id="more"
      icon="more_vert"
      :label="t('buttons.more')"
      @action="layoutStore.showHover('more')"
    />

    <div
      class="overlay"
      v-show="layoutStore.currentPromptName == 'more'"
      @click="layoutStore.closeHovers"
    />
  </header>
</template>

<script setup lang="ts">
import { useLayoutStore } from "@/stores/layout";

import { logoURL, prasLogo } from "@/utils/constants";

import Action from "@/components/header/Action.vue";
import { computed, useSlots } from "vue";
import { useI18n } from "vue-i18n";

defineProps<{
  showLogo?: boolean;
  showMenu?: boolean;
}>();

const layoutStore = useLayoutStore();
const slots = useSlots();

const { t } = useI18n();

const ifActionsSlot = computed(() => (slots.actions ? true : false));
</script>

<style></style>
