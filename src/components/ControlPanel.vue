<template>
  <!-- Toggle Button to open the panel -->
  <button class="open-panel" @click="panelClosed = false" v-if="panelClosed">►</button>

  <div class="control-panel" :class="{ closed: panelClosed }">
    <!-- Close Button (X) -->
    <button class="close-panel" @click="panelClosed = true">X</button>

    <div class="panel-content">
      <h2 class="panel-title">Control Panel</h2>

      <div class="upload">
        <h3 class="upload-title">Upload</h3>
        <div class="file-upload">
          <label for="upper-jaw-upload">Upload Upper Jaw</label>
          <input id="upper-jaw-upload" type="file" @change="setUpperJawFile" accept=".glb">
        </div>

        <div class="file-upload">
          <label for="lower-jaw-upload">Upload Lower Jaw</label>
          <input id="lower-jaw-upload" type="file" @change="setLowerJawFile" accept=".glb">
        </div>

        <button class="apply-btn" @click="applyFiles">Apply</button>
      </div>

      <button class="control-btn" @click="$emit('toggleJaw')">{{ 'Open Jaw' }}</button>

      <h3 class="rotation-title">Rotate Jaws</h3>
      <div class="rotation-controls">
        <div class="jaw-control">
          <div class="jaw-label">Rotate Upper Jaw</div>
          <button class="rotate-btn left-btn" @click="$emit('rotateUpperJaw', 'left')">&lt;</button>
          <button class="rotate-btn right-btn" @click="$emit('rotateUpperJaw', 'right')">&gt;</button>
        </div>
        <div class="jaw-control">
          <div class="jaw-label">Rotate Lower Jaw</div>
          <button class="rotate-btn left-btn" @click="$emit('rotateLowerJaw', 'left')">&lt;</button>
          <button class="rotate-btn right-btn" @click="$emit('rotateLowerJaw', 'right')">&gt;</button>
        </div>
      </div>

      <button class="control-btn" @click="$emit('startTranslation')">Translate Upper Jaw</button>
      <button class="control-btn" @click="$emit('checkCollision')">Region of collision</button>

      <table class="bounding-box-info">
        <tbody>
        <tr>
          <td class="info-text">Jaw</td>
          <td class="info-text">Upper Jaw</td>
          <td class="info-text">Lower Jaw</td>
        </tr>
        <tr>
          <td class="info-text">Center</td>
          <td class="info-text">{{ upperJawInfo.center }}</td>
          <td class="info-text">{{ lowerJawInfo.center }}</td>
        </tr>
        <tr>
          <td class="info-text">Size</td>
          <td class="info-text">{{ upperJawInfo.size }}</td>
          <td class="info-text">{{ lowerJawInfo.size }}</td>
        </tr>
        <tr>
          <td class="info-text">Min</td>
          <td class="info-text">{{ upperJawInfo.min }}</td>
          <td class="info-text">{{ lowerJawInfo.min }}</td>
        </tr>
        <tr>
          <td class="info-text">Max</td>
          <td class="info-text">{{ upperJawInfo.max }}</td>
          <td class="info-text">{{ lowerJawInfo.max }}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ControlPanel',
  props: {
    jawOpen: Boolean,
    upperJawInfo: Object,
    lowerJawInfo: Object,
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
    },
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
  margin-bottom: 10px;
  margin-top: 20px;
  font-size: 1.5em;
  color: #333;
}

.upload-title {
  font-size: 1.5em;
  margin-left: 3px;
  color: #333;
}

.control-btn {
  background-color: #2196F3;
  color: white;
  border: none;
  padding: 10px 15px;
  margin-bottom: 10px;
  margin-top: 10px;
  cursor: pointer;
  width: 100%;
}

.file-upload {
  margin-bottom: 10px;
  margin-left: 3px;
  color: #333;
}

.file-upload input {
  width: 100%;
}

.apply-btn {
  margin-bottom: 10px;
  margin-left: 10px;
  margin-right: 30px;
  min-height: 30px;
  max-width: 200px;
  background-color: #2196F3;
  color: white;
  border: none;
  cursor: pointer;
  width: 100%;
}

.upload {
  border: 2px solid #4CAF50;
}

.rotation-title {
  font-size: 1.2em;
  color: #333;
  margin-bottom: 10px;
}

.rotation-controls {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.jaw-control {
  display: flex;
  align-items: center;
  gap: 10px;
}

.jaw-label {
  font-weight: bold;
  color: #444;
}

.rotate-btn {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  font-size: 0.9em;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.rotate-btn:hover {
  background-color: #367c39;
}

.left-btn {
  margin-right: 5px;
}

.right-btn {
  margin-left: 5px;
}

.bounding-box-info {
  /* Styling for the table */
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.bounding-box-info th, .bounding-box-info td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.info-text {
  color: #444;
}

.bounding-box-info th {
  background-color: #f2f2f2;
}
</style>
