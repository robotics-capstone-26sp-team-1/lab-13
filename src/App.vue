<script setup lang="ts">
import ROSBridgeConnection from '@/components/ROSBridgeConnection.vue'
import PoseManager from '@/components/PoseManager.vue'
import PoseSequencer from '@/components/PoseSequencer.vue'

import { ref } from 'vue'
import { Ros } from 'roslib'
import type { GetPoseResponse, NamedLink } from '@/types/messages.ts'

/// ROS instance.
const ros = new Ros()

/// Connected to ROS bridge
const isConnected = ref(true)

/// Recording of all poses.
const poses = ref<Record<string, GetPoseResponse>>({})

/// Ordered pose names.
const sequence = ref<NamedLink[]>([])

function addPose(name: string, pose: GetPoseResponse) {
  poses.value[name] = pose
}
function deletePose(name: string) {
  delete poses.value[name]
}
function addPoseToSequence(pose: NamedLink) {
  sequence.value.push(pose)
}
function removePoseFromSequence(index: number) {
  sequence.value.splice(index, 1)
}
</script>

<template>
  <div class="container mx-auto">
    <Menubar>
      <template #start>
        <h1 class="text-2xl">Lab 13: Programming by Demonstration</h1>
      </template>
    </Menubar>
    <br />
    <ROSBridgeConnection
      :ros="ros"
      @connected="isConnected = true"
      @disconnected="isConnected = false"
    />
    <br />
    <div v-if="isConnected">
      <PoseManager
        :poses="poses"
        @onPoseSave="addPose"
        @onDeletePose="deletePose"
        @onPoseSent="addPoseToSequence"
      />
      <br />
      <PoseSequencer
        :poses="poses"
        :sequence="sequence"
        @onDeleteFromSequence="removePoseFromSequence"
      />
    </div>
  </div>
</template>
