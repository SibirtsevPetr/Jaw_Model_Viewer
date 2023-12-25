<template>
  <div ref="threeContainer" class="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import ControlPanel from "@/components/ControlPanel.vue";
import { ref } from 'vue';
import { markRaw } from 'vue';

const jawOpen = ref(false);

export default {
  name: 'JawModelViewer',
  components: {ControlPanel},

  props: {
    jawOpen: Boolean,
    lowerJawFile: File,
    upperJawFile: File
  },
  data() {
    return {
      lowerJaw: null,
      upperJaw: null,
    };
  },

  mounted() {
    // Initialize the Three.js scene when the component is mounted
    this.initThreeJS();
  },

  methods: {
    initThreeJS() {
      // Creating the main scene
      this.scene = new THREE.Scene();

      // Setting up the camera with perspective view
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

      // Creating the WebGL renderer
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setClearColor(0xeeeeee); // Setting a light grey background color

      // Mounting the renderer's DOM element to the Vue component
      this.$refs.threeContainer.appendChild(renderer.domElement);

      // Adding ambient light (soft, from all directions) otherwise teeth are black
      const ambientLight = new THREE.AmbientLight(0xffffff); // White light
      this.scene.add(ambientLight);

      // Setting the camera's position
      this.camera.position.set(0, 100, 0); // Positioned to view both jaws

      // OrbitControls setup for interactive scene manipulation (rotate, zoom, pan)
      const controls = new OrbitControls(this.camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;

      // Adding directional light for better illumination
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(0, 1, 1);
      this.scene.add(directionalLight);

      // Adding hemisphere light for soft, ambient lighting
      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444);
      hemiLight.position.set(0, 20, 0);
      this.scene.add(hemiLight);

      // Setting up the GLTF loader to load 3D models
      // const loader = new GLTFLoader();

      // // Loading the upper jaw model, rotating it, and adding to the scene
      // loader.load('/glb/upper-jaw-vertical.glb', (gltf) => {
      //   gltf.scene.position.set(0, 20, 0); // Adjusted position for visibility
      //   this.scene.add(gltf.scene);
      // });
      //
      // // Loading the lower jaw model and positioning it
      // loader.load('/glb/lower-jaw-vertical.glb', (gltf) => {
      //   gltf.scene.rotation.y = Math.PI; // Flipping the lower jaw as it's upside down
      //   gltf.scene.position.set(0, 20, 0); // Adjusted position for visibility
      //   this.lowerJaw = gltf.scene; // Store a reference to the lower jaw model
      //   this.scene.add(gltf.scene);
      // });

      // Animation loop for rendering the scene
      const animate = () => {
        requestAnimationFrame(animate);
        controls.update(); // Required for OrbitControls to function
        renderer.render(this.scene, this.camera);
      };
      animate();

      // Handling window resize to adjust camera aspect ratio and renderer size
      const onWindowResize = () => {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      };
      window.addEventListener('resize', onWindowResize);

      // Cleanup function to remove event listeners and the renderer's DOM element
      this.$once('hook:beforeDestroy', () => {
        window.removeEventListener('resize', onWindowResize);
        this.$refs.threeContainer.removeChild(renderer.domElement);
        // Additional cleanup here if necessary
      });
    },

    loadModel(file, position, rotation, assignTo) {
      const reader = new FileReader();
      reader.onload = (event) => {
        const loader = new GLTFLoader();
        loader.parse(event.target.result, '', (gltf) => {
          // Add the new model with a unique identifier
          const model = gltf.scene;
          model.name = assignTo; // Assign a unique name
          model.position.set(...position);
          model.rotation.set(...rotation);

          const existingModel = this.scene.getObjectByName(assignTo);
          if (existingModel) {
            this.scene.remove(existingModel);
            if (typeof existingModel.dispose === 'function') {
              existingModel.dispose();
            }
          }

          // Change the color of each tooth
          this.colorizeTeeth(model);
          this.updateUpperJawInfo(model)
          this.updateLowerJawInfo(model)
          this[assignTo] = markRaw(model);
          this.scene.add(model);
        });
      };
      reader.readAsArrayBuffer(file);

    },

    colorizeTeeth(model) {
      model.traverse((child) => {
        if (child.isMesh) {
          // Change a tooth material applying random color to each tooth
          const color = this.getRandomColor();
          child.material = new THREE.MeshPhongMaterial({ color });
        }
      });
    },

    getRandomColor() {
      // Generate a random color
      const letters = '456789ABCDEF';
      let color = '#';
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 12)];
      }
      return color;
    },

    // Method to update jaw position (moved from toggleJaw)
    updateJawPosition() {
      if (this.lowerJaw) {
        this.upperJaw.position.z -= 5;
      }
    },

    rotateUpperJaw(direction) {
      const angle = direction === 'left' ? -Math.PI / 32 : Math.PI / 32;
      if (this.upperJaw) {
        this.upperJaw.rotation.y += angle;
      }
    },

    rotateLowerJaw(direction) {
      const angle = direction === 'left' ? -Math.PI / 32 : Math.PI / 32;
      if (this.lowerJaw) {
        this.lowerJaw.rotation.y += angle;
      }
    },

    translateUpperJaw() {
      const step = 0.1; // Distance to move in each step
      const lowerJawPosition = this.lowerJaw.position;
      const upperJawPosition = this.upperJaw.position;

      // Check if the upper jaw has collided with the lower jaw
      if (upperJawPosition.z + step < lowerJawPosition.z) {

        // Move the upper jaw closer to the lower jaw
        this.upperJaw.position.z += step;
        // Use requestAnimationFrame for smoother animation
        requestAnimationFrame(this.translateUpperJaw);
      }
    },

    detectAndHighlightCollisions() {
      if (this.upperJaw && this.lowerJaw) {
        for (let i = 1; i <= 14; i++) {
          const upperToothName = `appworkdirroot-estimationmodelsupper${i}_crownstl`;
          const lowerToothName = `appworkdirroot-estimationmodelslower${i}_crownstl`;

          const upperTooth = this.findToothByName(this.upperJaw, upperToothName);
          const lowerTooth = this.findToothByName(this.lowerJaw, lowerToothName);

          if (upperTooth && lowerTooth) {
            const distance = this.calculateDistanceBetweenTeeth(upperTooth, lowerTooth);
            if (distance > 7.5) {
              this.changeMeshColor(upperTooth, 'yellow');
              this.changeMeshColor(lowerTooth, 'yellow');
            }
          }
        }
      }
    },

    findToothByName(jaw, toothName) {
      let tooth = null;
      jaw.traverse((child) => {
        if (child.name === toothName) {
          tooth = child;
        }
      });
      return tooth;
    },

    calculateDistanceBetweenTeeth(upperTooth, lowerTooth) {
      const upperBox = new THREE.Box3().setFromObject(upperTooth);
      const lowerBox = new THREE.Box3().setFromObject(lowerTooth);
      const upperCenter = new THREE.Vector3();
      const lowerCenter = new THREE.Vector3();
      upperBox.getCenter(upperCenter);
      lowerBox.getCenter(lowerCenter);
      return upperCenter.distanceTo(lowerCenter);
    },

    changeMeshColor(mesh, color) {
      if (mesh) {
        mesh.material.color.set(color);
      }
    },

    updateUpperJawInfo(model) {
      if (model.name === 'upperJaw') {
        const bbox = new THREE.Box3().setFromObject(model);
        const center = bbox.getCenter(new THREE.Vector3());
        const sizeVector = bbox.getSize(new THREE.Vector3());

        // Calculate the magnitude of the size vector
        const sizeMagnitude = Math.sqrt(sizeVector.x ** 2 + sizeVector.y ** 2 + sizeVector.z ** 2);

        // Format the center coordinates to two decimal places
        const formattedCenter = `(${center.x.toFixed(2)}, ${center.y.toFixed(2)}, ${center.z.toFixed(2)})`;

          // Format the size magnitude to two decimal places
        const formattedSize = sizeMagnitude.toFixed(2);

        const formattedMin = `(${bbox.min.x.toFixed(2)}, ${bbox.min.y.toFixed(2)}, ${bbox.min.z.toFixed(2)})`;
        const formattedMax = `(${bbox.max.x.toFixed(2)}, ${bbox.max.y.toFixed(2)}, ${bbox.max.z.toFixed(2)})`;

        // Emitting the information to the parent
        this.$emit('updateUpperJawInfo', {
          center: formattedCenter.toString(),
          size: formattedSize.toString(),
          min: formattedMin.toString(),
          max: formattedMax.toString()
        });
      }
    },

    updateLowerJawInfo(model) {
      if (model.name === 'lowerJaw') {
        const bbox = new THREE.Box3().setFromObject(model);
        const center = bbox.getCenter(new THREE.Vector3());
        const sizeVector = bbox.getSize(new THREE.Vector3());

        // Calculate the magnitude of the size vector
        const sizeMagnitude = Math.sqrt(sizeVector.x ** 2 + sizeVector.y ** 2 + sizeVector.z ** 2);

        // Format the center coordinates to two decimal places
        const formattedCenter = `(${center.x.toFixed(2)}, ${center.y.toFixed(2)}, ${center.z.toFixed(2)})`;

        // Format the size magnitude to two decimal places
        const formattedSize = sizeMagnitude.toFixed(2);

        const formattedMin = `(${bbox.min.x.toFixed(2)}, ${bbox.min.y.toFixed(2)}, ${bbox.min.z.toFixed(2)})`;
        const formattedMax = `(${bbox.max.x.toFixed(2)}, ${bbox.max.y.toFixed(2)}, ${bbox.max.z.toFixed(2)})`;

        // Emitting the information to the parent
        this.$emit('updateLowerJawInfo', {
          center: formattedCenter.toString(),
          size: formattedSize.toString(),
          min: formattedMin.toString(),
          max: formattedMax.toString()
        });
      }
    },
  },
  watch: {
    lowerJawFile(file) {
      if (file) {
        this.loadModel(file, [0, 20, 0], [0, Math.PI, 0], 'lowerJaw');
      }
    },
    upperJawFile(file) {
      if (file) {
        this.loadModel(file, [0, 20, 0], [0, 0, 0], 'upperJaw');
      }
    },
    jawOpen() {
      this.updateJawPosition();
    }
  }
};

function toggleJaw() {
  jawOpen.value = !jawOpen.value;
}

</script>

<style>
.three-container {
  width: 100%;
  height: 100vh;
}
</style>
