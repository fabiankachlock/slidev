<script setup lang="ts">
import { ref, computed, shallowRef } from 'vue'
import { showEditor, windowSize, isScreenVertical, slideScale } from '../state'
import { isPrintMode, next, prev, useSwipeControls, isEmbedded } from '../logic/nav'
import { isDrawing } from '../logic/drawings'
import { registerShortcuts } from '../logic/shortcuts'
import { themeVars } from '../env'
import Controls from './Controls.vue'
import SlideContainer from './SlideContainer.vue'
import NavControls from './NavControls.vue'
import SlidesShow from './SlidesShow.vue'

registerShortcuts()

const root = ref<HTMLDivElement>()
function onClick(e: MouseEvent) {
  if (showEditor.value)
    return

  if ((e.target as HTMLElement)?.id === 'slide-container') {
    // click right to next, left to previouse
    if ((e.screenX / window.innerWidth) > 0.6)
      next()
    else
      prev()
  }
}

useSwipeControls(root)

const presistNav = computed(() => isScreenVertical.value || showEditor.value)

const Editor = shallowRef<any>()
if (__DEV__)
  import('./Editor.vue').then(v => Editor.value = v.default)

const DrawingControls = shallowRef<any>()
if (__SLIDEV_FEATURE_DRAWINGS__)
  import('./DrawingControls.vue').then(v => DrawingControls.value = v.default)
</script>

<template>
  <div id="page-root" ref="root" class="grid grid-cols-[1fr,max-content]" :style="themeVars">
    <SlideContainer
      class="w-full h-full"
      :style="{ background: 'var(--slidev-slide-container-background, black)'}"
      :width="isPrintMode ? windowSize.width.value : undefined"
      :scale="slideScale"
      @pointerdown="onClick"
    >
      <template #>
        <SlidesShow />
      </template>
      <template #controls>
        <div
          class="absolute bottom-0 left-0 transition duration-300 opacity-0 hover:opacity-100"
          :class="[
            presistNav ? 'opacity-100 right-0' : 'oapcity-0 p-2',
            isDrawing ? 'pointer-events-none': ''
          ]"
        >
          <NavControls class="m-auto" :persist="presistNav" />
        </div>
        <template v-if="__SLIDEV_FEATURE_DRAWINGS__ && !isEmbedded && DrawingControls">
          <DrawingControls class="ml-0" />
        </template>
      </template>
    </SlideContainer>

    <template v-if="__DEV__">
      <Editor v-if="Editor && showEditor" />
    </template>
  </div>
  <Controls />
</template>
