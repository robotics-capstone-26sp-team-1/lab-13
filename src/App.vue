<script setup lang="ts"></script>

<template>
  <div class="container mx-auto">
    <Menubar>
      <template #start>
        <h1 class="text-2xl">Lab 13: Programming by Demonstration</h1>
      </template>
    </Menubar>
    <br />
    <Panel header="ROS Bridge Connection">
      <div class="flex gap-4">
        <IftaLabel>
          <InputText id="ros-bridge-ip" />
          <label for="ros-bridge-ip">IP address</label>
        </IftaLabel>
        <Button label="Connect" />
      </div>
    </Panel>
    <br />
    <Panel header="Pose Manager">
      <h3 class="text-lg">Add a Pose</h3>
      <div class="flex gap-4">
        <IftaLabel>
          <InputText id="pose-name" />
          <label for="pose-name">Pose name</label>
        </IftaLabel>
        <Button label="Save" />
      </div>
      <br />
      <h3 class="text-lg">Select a Pose</h3>
      <Listbox filter />
      <br />
      <div class="flex gap-4">
        <Button label="Delete Pose" severity="danger" />
        <Button label="Add to Sequence" />
      </div>
    </Panel>
    <br />
    <Panel header="Sequencer">
      <Listbox multiple />
      <br />
      <div class="flex gap-4">
        <Button label="Delete Pose" severity="danger" />
        <Button label="Run" />
      </div>
    </Panel>
  </div>
</template>

<style scoped></style>
