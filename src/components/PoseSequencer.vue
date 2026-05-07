<script setup lang="ts">
import { computed, ref } from 'vue'
import type { GetPoseResponse, NamedLink } from '@/types/messages.ts'

// Props.
const { sequence } = defineProps<{
  poses: Record<string, GetPoseResponse>
  sequence: NamedLink[]
}>()

// State.
const selectedPose = ref<number | undefined>()
const sequenceOptions = computed(() => sequence.map(({ name }, index) => ({ name, index })))

// Events.
const emit = defineEmits<{ (e: 'onDeleteFromSequence', index: number): void }>()

// Functions.
function deletePose() {
  emit('onDeleteFromSequence', selectedPose.value!)
  selectedPose.value = undefined
}
</script>

<template>
  <Panel header="Sequencer">
    <Listbox v-model="selectedPose" :options="sequenceOptions" optionLabel="name" optionValue="index" />
    <br />
    <div class="flex gap-4">
      <Button label="Delete Pose" severity="danger" :disabled="selectedPose === undefined" @click="deletePose" />
      <Button label="Run" />
    </div>
  </Panel>
</template>
