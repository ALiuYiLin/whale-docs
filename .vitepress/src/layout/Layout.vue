<script setup lang="ts">
import Header from './Header.vue'
import Footer from './Footer.vue'
import Siderbar from '@/components/Siderbar.vue'
import { useLayout } from '@/router'
import { useSiderBar } from '@/composables/use-siderbar'
import { computed, onMounted } from 'vue'
const { currentView } = useLayout()
const { currentSiderbarItem } = useSiderBar()
const hasParent = computed(()=> !!parent)

onMounted(() => {
  const htmlElement = document.documentElement;
      console.log('htmlElement: ', htmlElement);
      console.log('htmlElement.scrollHeight: ', htmlElement.scrollHeight);
      console.log('htmlElement.offsetHeight: ', htmlElement.offsetHeight);
      console.log('htmlElement.clientHeight: ', htmlElement.clientHeight);
  parent.postMessage({
    height: document.documentElement.scrollHeight,
    width: document.documentElement.scrollWidth,
  }, 'http://localhost:3000')
})
</script>

<template>
  <div class="layout relative">
    <Header v-if="!hasParent"></Header>
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
