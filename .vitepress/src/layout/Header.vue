<script lang="ts" setup>
import { useRouter } from 'vitepress';
import { computed } from 'vue';

const router = useRouter()
</script>
<template>
  <header class="flex flex-row items-center pl-10">
    <div class="flex flex-row items-center gap-2">
      <div>
        <img src="@/assets/images/logo.png"  class="h-[24px]" @click="router.go('/whale-docs/')">
      </div>
      <span class="text-lg" @click="router.go('/whale-docs/')">whale-docs</span>
    </div>
    <div></div>
  </header>
</template>

<style scoped>
header {
  @apply h-[60px] sticky top-0 w-full z-50 border-solid border-[1px] border-gray-200 bg-white ;
  display: flex;
  flex-direction: row;
}
header:hover {
  cursor: pointer;
}
</style>