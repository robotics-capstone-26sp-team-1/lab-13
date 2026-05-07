<script setup lang="ts">
import { computed, ref } from 'vue'
import { ActionClient, Goal, type Ros } from 'roslib'
import type {
  GetPoseResponse,
  NamedLink,
  SetPoseFeedback,
  SetPoseGoal,
  SetPoseResult,
} from '@/types/messages.ts'
import type { RosTransformStamped } from '@/types/ros.ts'

// Props.
const { ros, poses, sequence } = defineProps<{
  ros: Ros
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

function getTargetPoseFromLink(pose: GetPoseResponse, link: string): RosTransformStamped {
  if (link === 'base_link') return pose.robot_pose
  if (link === 'odom') return pose.world_pose
  if (link === 'column_4') return pose.aruco_pose
  throw new Error(`Unsupported sequence link: ${link}`)
}

function asSetPoseFeedback(event: unknown): SetPoseFeedback | null {
  if (!event || typeof event !== 'object') return null
  const payload = 'feedback' in event ? event.feedback : event
  if (!payload || typeof payload !== 'object' || !('status' in payload)) return null
  if (typeof payload.status !== 'string') return null
  return payload as SetPoseFeedback
}

function asSetPoseResult(event: unknown): SetPoseResult | null {
  if (!event || typeof event !== 'object') return null
  const payload = 'result' in event ? event.result : event
  if (!payload || typeof payload !== 'object') return null
  if (!('success' in payload) || !('message' in payload)) return null
  if (typeof payload.success !== 'boolean' || typeof payload.message !== 'string') return null
  return payload as SetPoseResult
}

async function runSetPoseGoal(goalMessage: SetPoseGoal): Promise<void> {
  const actionClient = new ActionClient({
    ros,
    serverName: '/set_pose',
    actionName: 'stretch_pose_interfaces/action/SetPose',
  })
  const goal = new Goal({
    actionClient,
    goalMessage,
  })

  await new Promise<void>((resolve, reject) => {
    goal.on('feedback', (feedbackEvent: unknown) => {
      console.log('set_pose feedback:', asSetPoseFeedback(feedbackEvent) ?? feedbackEvent)
    })
    goal.on('result', (resultEvent: unknown) => {
      const result = asSetPoseResult(resultEvent)
      if (!result) {
        actionClient.dispose()
        reject(new Error('set_pose returned an invalid result payload'))
        return
      }
      console.log('set_pose result:', result)
      actionClient.dispose()

      if (!result.success) {
        reject(new Error(result.message))
        return
      }
      resolve()
    })
    goal.on('timeout', () => {
      actionClient.dispose()
      reject(new Error('set_pose action timed out'))
    })

    goal.send()
  })
}

async function runSequence() {
  for (const poseLink of sequence) {
    const pose = poses[poseLink.name]
    if (!pose) {
      console.error(`Pose "${poseLink.name}" does not exist.`)
      return
    }

    const targetPose = getTargetPoseFromLink(pose, poseLink.link)
    try {
      await runSetPoseGoal({ target_pose: targetPose })
    } catch (error) {
      console.error('Failed to run sequence pose:', error)
      return
    }
  }
}
</script>

<template>
  <Panel header="Sequencer">
    <Listbox
      v-model="selectedPose"
      :options="sequenceOptions"
      optionLabel="name"
      optionValue="index"
    />
    <br />
    <div class="flex gap-4">
      <Button
        label="Delete Pose"
        severity="danger"
        :disabled="selectedPose === undefined"
        @click="deletePose"
      />
      <Button label="Run" @click="runSequence" />
    </div>
  </Panel>
</template>
