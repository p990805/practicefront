<template>
  <div>
    <h1>3D Avatar Simulator</h1>
    <div ref="avatarContainer" class="avatar-container"></div>
    <div class="controls">
      <label>
        Height (cm):
        <input type="range" min="150" max="200" v-model="height" @input="updateScale" />
        {{ height }}
      </label>
      <label>
        Weight (kg):
        <input type="range" min="50" max="100" v-model="weight" @input="updateScale" />
        {{ weight }}
      </label>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

export default {
  name: "AvatarSimulator",
  data() {
    return {
      height: 170, // Default height
      weight: 70,  // Default weight
    };
  },
  methods: {
    initThree() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0xf0f0f0);

      this.camera = new THREE.PerspectiveCamera(
        75,
        this.$refs.avatarContainer.clientWidth /
          this.$refs.avatarContainer.clientHeight,
        0.1,
        1000
      );
      this.camera.position.set(0, 1.5, 5);

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(
        this.$refs.avatarContainer.clientWidth,
        this.$refs.avatarContainer.clientHeight
      );
      this.$refs.avatarContainer.appendChild(this.renderer.domElement);

      const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1);
      light.position.set(0, 1, 0);
      this.scene.add(light);

      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.05;

      const loader = new GLTFLoader();
      loader.load(
        "/male_model.glb",
        (gltf) => {
          this.model = gltf.scene;
          this.model.scale.set(2, 2, 2);
          this.model.position.set(0, -1, 0);
          this.scene.add(this.model);
          this.animate();
        },
        undefined,
        (error) => {
          console.error("Error loading model:", error);
        }
      );
    },
    updateScale() {
      if (this.model) {
        const heightScale = this.height / 170;
        const weightScale = 1 + (this.weight - 70) / 200;
        this.model.scale.set(weightScale, heightScale, weightScale);
      }
    },
    animate() {
      requestAnimationFrame(() => this.animate());
      this.controls.update();
      this.renderer.render(this.scene, this.camera);
    },
  },
  mounted() {
    this.initThree();
  },
};
</script>

<style>
.avatar-container {
  width: 100%;
  height: 500px;
  border: 1px solid #ccc;
  margin-bottom: 20px;
}
.controls {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
label {
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>
