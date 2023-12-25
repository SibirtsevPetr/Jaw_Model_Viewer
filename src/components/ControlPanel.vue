<template>
  <div class="controls">
    <button @click="$emit('toggleJaw')">{{ jawOpen ? 'Close Jaw' : 'Open Jaw' }}</button>
  </div>
</template>

<script>
export default {
  name: 'ControlPanel',
  props: {
    jawOpen: Boolean
  }
};
</script>

<style>
.controls {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  z-index: 10;
}
</style>
