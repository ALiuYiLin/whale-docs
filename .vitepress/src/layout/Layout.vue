<script setup lang="ts">
import Header from './Header.vue'
import Footer from './Footer.vue'
import Siderbar from '@/components/Siderbar.vue'
import { useLayout } from '@/router'
import { useSiderBar } from '@/composables/use-siderbar'
import { computed, onMounted, nextTick,onBeforeUnmount  } from 'vue'
const { currentView } = useLayout()
const { currentSiderbarItem } = useSiderBar()
const inIframe = computed(()=> window.self !== window.top)


function calcHeight() {
  const html = document.documentElement
  const body = document.body
  return Math.max(
    html.scrollHeight, html.offsetHeight, html.clientHeight,
    body?.scrollHeight ?? 0, body?.offsetHeight ?? 0, body?.clientHeight ?? 0
  )
}


// onMounted(() => {
//   console.log('currentView: ', currentView);
//   const htmlElement = document.documentElement;
//       console.log('htmlElement: ', htmlElement);
//       console.log('htmlElement.scrollHeight: ', htmlElement.scrollHeight);
//       console.log('htmlElement.offsetHeight: ', htmlElement.offsetHeight);
//       console.log('htmlElement.clientHeight: ', htmlElement.clientHeight);
//   parent.postMessage({
//     height: document.documentElement.scrollHeight,
//     width: document.documentElement.scrollWidth,
//   }, 'http://localhost:5174')
// })

onMounted(async () => {
  // 必须确认自己确实在 iframe 里
  const inIframe = window.self !== window.top
  if (!inIframe) return

  const targetOrigin = 'http://localhost:5174' // 改成你的父页面 origin（协议+域名+端口）

  const post = () => {
    parent.postMessage({ type: 'IFRAME_RESIZE', height: calcHeight() }, targetOrigin)
  }

  // 初次发送（等 DOM / 样式渲染完）
  await nextTick()
  post()

  // 观察尺寸变化（内容变动、图片加载后高度变化等）
  const ro = new ResizeObserver(() => post())
  ro.observe(document.documentElement)

  // 可选：图片加载也可能触发（保险）
  window.addEventListener('load', post)

  onBeforeUnmount(() => {
     ro.disconnect()
     window.removeEventListener('load', post)
   })
})

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
  @apply flex flex-col ml-[300px]  justify-center;
}
.main-content--left {
  width: 0;
}
@media screen and (min-width: 1500px) {
  .main-content--left{
    width: 8vw;
  }
}

@media screen and (max-width: 996px) {
  .main-content{
    margin-left: 0;
  }
}
</style>
