<script setup>
import { ref } from "vue";

let modelChosen = ref("microphone");

function getModel(model) {
  modelChosen.value = model;
}
</script>

<template>
  <HeaderComponent @model="getModel" />
  <Microphone v-if="modelChosen === 'microphone'" />
  <DamagedHelmet v-if="modelChosen === 'helmet'" />
</template>
