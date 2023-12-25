<script setup> // Import ThreeScene
import { ref } from 'vue';
import ControlPanel from './components/ControlPanel.vue';
import JawModelViewer from './components/JawModelViewer.vue';

const jawOpen = ref(false);
const lowerJawFile = ref(null);
const upperJawFile = ref(null);
const jawViewerRef = ref(null);

function toggleJaw() {
  jawOpen.value = !jawOpen.value;
}

function setLowerJawFile(file) {
  lowerJawFile.value = file;
}

function setUpperJawFile(file) {
  upperJawFile.value = file;
}

function rotateUpperJaw(direction) {
  if (jawViewerRef.value) {
    jawViewerRef.value.rotateUpperJaw(direction);
  }
}

function rotateLowerJaw(direction) {
  if (jawViewerRef.value) {
    jawViewerRef.value.rotateLowerJaw(direction);
  }
}

</script>

<template>
  <header>
    <div class="wrapper"></div>
  </header>

  <main>
    <ControlPanel
        :jawOpen="jawOpen"
        @toggleJaw="toggleJaw"
        @loadLowerJaw="setLowerJawFile"
        @loadUpperJaw="setUpperJawFile"
        @rotateUpperJaw="rotateUpperJaw"
        @rotateLowerJaw="rotateLowerJaw"
    />
    <JawModelViewer
        ref="jawViewerRef"
        :jawOpen="jawOpen"
        :lowerJawFile="lowerJawFile"
        :upperJawFile="upperJawFile"
    />
  </main>
</template>

<style scoped>

header {
  line-height: 1.5;
}
</style>
