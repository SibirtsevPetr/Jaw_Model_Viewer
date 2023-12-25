<script setup> // Import ThreeScene
import JawModelViewer from './components/JawModelViewer.vue';
import { ref } from 'vue';
import ControlPanel from './components/ControlPanel.vue';

const jawOpen = ref(false);

function toggleJaw() {
  jawOpen.value = !jawOpen.value;
}
</script>

<template>
  <header>
    <div class="wrapper">
    </div>
  </header>

  <main>
    <ControlPanel :jawOpen="jawOpen" @toggleJaw="toggleJaw" />
    <JawModelViewer :jawOpen="jawOpen" />
  </main>
</template>

<style scoped>

header {
  line-height: 1.5;
}
</style>
