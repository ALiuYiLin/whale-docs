<script lang="ts" setup>
import '@/assets/styles/base.css'
import '@/assets/styles/fonts.css'
import '@/assets/styles/icons.css'
import '@/assets/styles/utils.css'
import '@/assets/styles/vars.css'
import '@/assets/styles/components/custom-block.css'
import '@/assets/styles/components/vp-code-group.css'
import '@/assets/styles/components/vp-code.css'
import '@/assets/styles/components/vp-doc.css'
import '@/assets/styles/components/vp-sponsor.css'
import VPDocAsideOutline from '@/components/VPDocAsideOutline.vue'
import { computed } from 'vue'

const inIframe = computed(()=> window.self !== window.top)

</script>
<template>
  <div class="flex flex-row" v-if="!inIframe">
    <div class="max-w-[960px] content">
      <Content class="vp-doc"></Content>
    </div>
    <div class="pl-[60px] relative hidden asider-container">
      <div class="w-[224px] fixed top-0 pt-[112px] h-screen">
        <VPDocAsideOutline></VPDocAsideOutline>
      </div>
    </div>
  </div>
  <Content class="vp-doc" v-else></Content>
</template>

<style scoped>
.vp-doc--left {
  width: 0vw;
}
.content {
  width: 100%;
  overflow: hidden;
}
@media screen and (min-width: 1280px) {
  .vp-doc--left {
    width: 10vw;
  }
  .content {
    min-width: 640px;
  }
  .asider-container {
    display: block;

  }
}
</style>
