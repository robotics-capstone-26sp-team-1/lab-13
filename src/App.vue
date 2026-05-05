<script setup lang="ts">
import ROSBridgeConnection from '@/components/ROSBridgeConnection.vue'
import PoseManager from '@/components/PoseManager.vue'
import PoseSequencer from '@/components/PoseSequencer.vue'

import { ref } from 'vue'

const isConnected = ref(true)
</script>

<template>
  <div class="container mx-auto">
    <Menubar>
      <template #start>
        <h1 class="text-2xl">Lab 13: Programming by Demonstration</h1>
      </template>
    </Menubar>
    <br />
    <ROSBridgeConnection @connected="isConnected = true" @disconnected="isConnected = false" />
    <br />
    <div v-if="isConnected">
      <PoseManager />
      <br />
      <PoseSequencer />
    </div>
  </div>
</template>
