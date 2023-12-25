<script setup> // Import ThreeScene
import { ref } from 'vue';
import ControlPanel from './components/ControlPanel.vue';
import JawModelViewer from './components/JawModelViewer.vue';

const jawOpen = ref(false);
const lowerJawFile = ref(null);
const upperJawFile = ref(null);
const jawViewerRef = ref(null);
const upperJawInfo = ref({});
const lowerJawInfo = ref({});


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

function startTranslation() {
  if (jawViewerRef.value) {
    jawViewerRef.value.translateUpperJaw();
  }
}

async function checkCollision() {
  if (jawViewerRef.value) {
    jawViewerRef.value.translateUpperJaw();
    jawViewerRef.value.detectAndHighlightCollisions();
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
        :upperJawInfo="upperJawInfo"
        :lowerJawInfo="lowerJawInfo"
        @toggleJaw="toggleJaw"
        @loadLowerJaw="setLowerJawFile"
        @loadUpperJaw="setUpperJawFile"
        @rotateUpperJaw="rotateUpperJaw"
        @rotateLowerJaw="rotateLowerJaw"
        @startTranslation="startTranslation"
        @checkCollision="checkCollision"
    />
    <JawModelViewer
        ref="jawViewerRef"
        :jawOpen="jawOpen"
        :lowerJawFile="lowerJawFile"
        :upperJawFile="upperJawFile"
        @updateUpperJawInfo="upperJawInfo = $event"
        @updateLowerJawInfo="lowerJawInfo = $event"
    />
  </main>
</template>

<style scoped>

header {
  line-height: 1.5;
}
</style>
