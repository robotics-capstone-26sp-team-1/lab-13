<script setup lang="ts">
import { computed, ref } from 'vue'
import { type NamedLink, frames, type GetPoseResponse, type GetPoseRequest } from '@/types/messages'
import { Service, type Ros } from 'roslib'

// Props.
const { ros, poses } = defineProps<{ ros: Ros; poses: Record<string, GetPoseResponse> }>()
const getPoseService = new Service<GetPoseRequest, GetPoseResponse>({
  ros,
  name: '/get_pose',
  serviceType: 'stretch_pose_interfaces/srv/GetPose',
})

// State.
const poseOptions = computed(() => Object.keys(poses))
const poseName = ref('')
const selectedPose = ref<string | undefined>()
const selectedFrame = ref('base_link')

// Events.
const emit = defineEmits<{
  (e: 'onPoseSave', name: string, pose: GetPoseResponse): void
  (e: 'onDeletePose', name: string): void
  (e: 'onPoseSent', pose: NamedLink): void
}>()

// Functions.
function addPose() {
  // Ignore on blank name.
  if (poseName.value === '') return

  const request: GetPoseRequest = {}
  getPoseService.callService(
    request,
    (response) => {
      emit('onPoseSave', poseName.value, response)
      poseName.value = ''
    },
    (error) => {
      console.error(`Failed to call /get_pose: ${error}`)
    },
  )
}
function deletePose() {
  emit('onDeletePose', selectedPose.value!)
  selectedPose.value = undefined
}
function sendPose() {
  emit('onPoseSent', { name: selectedPose.value!, link: selectedFrame.value})
}
</script>

<template>
  <Panel header="Pose Manager">
    <h3 class="text-lg">Add a Pose</h3>
    <div class="flex gap-4">
      <IftaLabel>
        <InputText id="pose-name" v-model="poseName" />
        <label for="pose-name">Pose name</label>
      </IftaLabel>
      <Button label="Save" @click="addPose" />
    </div>
    <br />
    <h3 class="text-lg">Select a Pose</h3>
    <Listbox v-model="selectedPose" :options="poseOptions" filter />
    <br />
    <Button label="Delete Pose" severity="danger" :disabled="!selectedPose" @click="deletePose" />
    <br />
    <br />
    <div class="flex gap-4">
      <Select v-model="selectedFrame" :options="frames" optionLabel="name" optionValue="link" />
      <Button label="Add to Sequence" :disabled="!selectedPose" @click="sendPose" />
    </div>
  </Panel>
</template>
