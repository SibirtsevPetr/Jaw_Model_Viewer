<template>
  <!-- Toggle Button to open the panel -->
  <button class="open-panel" @click="panelClosed = false" v-if="panelClosed">►</button>

  <div class="control-panel" :class="{ closed: panelClosed }">
    <!-- Close Button (X) -->
    <button class="close-panel" @click="panelClosed = true">X</button>

    <div class="panel-content">
      <h3 class="panel-title">Control Panel</h3>
      <button class="control-btn" @click="$emit('toggleJaw')">{{ jawOpen ? 'Close Jaw' : 'Open Jaw' }}</button>

      <div class="file-upload">
        <label for="lower-jaw-upload">Upload Lower Jaw</label>
        <input id="lower-jaw-upload" type="file" @change="setLowerJawFile" accept=".glb">
      </div>

      <div class="file-upload">
        <label for="upper-jaw-upload">Upload Upper Jaw</label>
        <input id="upper-jaw-upload" type="file" @change="setUpperJawFile" accept=".glb">
      </div>

      <button class="apply-btn" @click="applyFiles">Apply</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ControlPanel',
  props: {
    jawOpen: Boolean
  },
  data() {
    return {
      panelClosed: false,
      tempLowerJawFile: null,
      tempUpperJawFile: null
    };
  },
  methods: {
    setLowerJawFile(event) {
      this.tempLowerJawFile = event.target.files[0];
    },
    setUpperJawFile(event) {
      this.tempUpperJawFile = event.target.files[0];
    },
    applyFiles() {
      if (this.tempLowerJawFile) {
        this.$emit('loadLowerJaw', this.tempLowerJawFile);
      }
      if (this.tempUpperJawFile) {
        this.$emit('loadUpperJaw', this.tempUpperJawFile);
      }
    }
  }
};
</script>

<style>
.control-panel {
  position: absolute;
  top: 0;
  left: 0;
  background-color: #fff;
  padding: 10px;
  border-right: 2px solid #ddd;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
  height: 100vh;
  width: 250px;
  transition: all 0.3s;
}

.control-panel.closed {
  transform: translateX(-100%);
}

.open-panel {
  background-color: #4CAF50;
  color: white;
  position: absolute;
  top: 10px;
  left: 10px;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 1em;
  z-index: 11;
}

.close-panel {
  background-color: #f44336;
  color: white;
  position: absolute;
  top: 10px;
  right: 10px;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 1em;
}

.panel-content {
  padding-top: 10px;
}

.panel-title {
  margin: 10px 0;
  font-size: 1.2em;
  color: #333;
}

.control-btn {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 10px 15px;
  margin-bottom: 10px;
  cursor: pointer;
  width: 100%;
}

.file-upload {
  margin-bottom: 10px;
  color: #333;
}

.file-upload input {
  width: 100%;
}
</style>
