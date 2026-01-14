<script setup lang="ts">
import Header from './Header.vue'
import Footer from './Footer.vue'
import Siderbar from '@/components/Siderbar.vue'
import { useLayout } from '@/router'
import { useSiderBar } from '@/composables/use-siderbar'
import { computed, onMounted, nextTick, onBeforeUnmount, onUpdated, watch, ref } from 'vue'
import { TAG_ORIGIN } from '@/constance'
import { on } from 'events'
const { currentView } = useLayout()
const { currentSiderbarItem } = useSiderBar()
const inIframe = computed(() => window.self !== window.top)

function calcHeight() {
  const html = document.documentElement
  const body = document.body
  return Math.max(
    html.scrollHeight, html.offsetHeight, html.clientHeight,
    body?.scrollHeight ?? 0, body?.offsetHeight ?? 0, body?.clientHeight ?? 0,
  )
}

const post = () => {
  parent.postMessage({ type: 'IFRAME_RESIZE', height: calcHeight() }, TAG_ORIGIN)
}

onMounted(() => {
  if (!inIframe) return
  post()
})

window.onresize = function () {
  if (!inIframe) return
  post()
}

</script>

<template>
  <div class="layout relative" v-if="!inIframe">
    <Header></Header>
    <main>
      <Siderbar></Siderbar>
      <div class="main-content">
        <div class="flex-1 p-5 flex flex-row">
          <div class="main-content--left"></div>
          <div class="flex-1">
            <h1 class="text-5xl font-bold p-[10px] pl-0 mb-5">{{ currentSiderbarItem?.text }}</h1>
            <component :is="currentView"></component>
          </div>
        </div>
        <Footer></Footer>
      </div>
    </main>
  </div>
  <component :is="currentView" v-else></component>
</template>

<style scoped>
.layout {
  min-height: 100vh;
}

.main-content {
  min-height: calc(100vh - 60px);
  @apply flex flex-col ml-[300px] justify-center;
}

.main-content--left {
  width: 0;
}

@media screen and (min-width: 1500px) {
  .main-content--left {
    width: 8vw;
  }
}

@media screen and (max-width: 996px) {
  .main-content {
    margin-left: 0;
  }
}
</style>
