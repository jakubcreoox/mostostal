<script setup lang="ts">
import LeftPanel from '@/components/LeftPanel.vue'
import XeokitViewerJakub from '@/components/XeokitViewerJakub.vue';
import Splitter from 'primevue/splitter';
import SplitterPanel from 'primevue/splitterpanel';
</script>

<template>
  <Splitter class="h-screen">
    <SplitterPanel class="flex items-center justify-center p-2" :size="25" :minSize="10">
      <LeftPanel></LeftPanel>
    </SplitterPanel>
    <SplitterPanel class="flex items-center justify-center class p-2" :size="75">
      <XeokitViewerJakub />
    </SplitterPanel>
  </Splitter>
</template>
