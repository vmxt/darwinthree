<template>
  <div class="mainContainer">
    <div ref="container" id="container">
    </div>

      <video id="video" class="hideVideo" type="video/mp4">
        <source src="../static/darwin.mp4" type="video/mp4" autoplay />
      </video>

    <div class="controls">
      <button type="button" name="translate" @click="controlSelect">
        Translate
      </button>
      <button type="button" name="rotate" @click="controlSelect">Rotate</button>
      <button type="button" name="scale" @click="controlSelect">Scale</button>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { TransformControls } from "three/examples/jsm/controls/TransformControls.js";

class Asset {
  constructor(name, meshes) {
    this.name = name;
    this.meshes = meshes;
  }
}

export default {
  name: "THREETest",
  data: function() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      gltf: null,
      orbit_controls: null,
      transform_controls: null,

      assets: [],
      gltf_clicked_name: ""
    };
  },
  mounted: function() {
    let container = this.$refs.container;

    this.camera = new THREE.PerspectiveCamera(
      100,
      window.innerWidth / window.innerHeight,
      1,
      100
    );
    this.camera.position.set(0, 2, 2);

    this.scene = new THREE.Scene();

    this.light = new THREE.AmbientLight(0xffffff, 5.3);
    this.scene.add(this.light);

    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setClearColor("#262626");
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.gammaFactor = 1.5;
    this.renderer.gammaOutput = true;
    container.appendChild(this.renderer.domElement);

    const video = document.getElementById("video");
    video.onloadeddata = function() {
      console.log("Keep Refreshing the video won't allow to load on time.");
      video.play();
    };
    
    const texture = new THREE.VideoTexture(video);
    texture.minFilter = THREE.LinearFilter;
    texture.magFilter = THREE.LinearFilter;
    texture.format = THREE.RGBFormat;
    texture.needsUpdate = true;

    const geometry = new THREE.PlaneGeometry(4, 2);
    const material = new THREE.MeshBasicMaterial({ map: texture });

    const plane = new THREE.Mesh(geometry, material);
    plane.material.side = THREE.DoubleSide;
    plane.position.y = 1;
    this.scene.add(plane);

    this.orbit_controls = new OrbitControls(
      this.camera,
      this.renderer.domElement
    );
    this.orbit_controls.addEventListener("change", () =>
      this.renderer.render(this.scene, this.camera)
    );

    this.transform_controls = new TransformControls(
      this.camera,
      this.renderer.domElement
    );
    this.scene.add(this.transform_controls);

    this.transform_controls.setSize(1);
    this.transform_controls.addEventListener("change", () => {
      this.renderer.render(this.scene, this.camera);
    });

    this.transform_controls.addEventListener("dragging-changed", event => {
      this.orbit_controls.enabled = !event.value;
    });

    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    const onMouseClick = event => {
      event.preventDefault();
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, this.camera);

      this.assets.push(new Asset("Screen", plane));
      const meshes = this.assets
        .map(asset => {
          return asset.meshes;
        })
        .flat();

      const intersects = raycaster.intersectObjects(meshes, true);
      if (intersects[0] !== undefined) {
        this.transform_controls.attach(intersects[0].object);
        intersects[0].object.traverseAncestors(anc => {
          if (anc.userData.asset !== undefined) {
            const model_name = anc.userData.asset.name;
            this.gltf_clicked_name = model_name;
          }
        });
      }
    };

    window.addEventListener("click", onMouseClick, true);
    this.animate();
  },

  methods: {
    controlSelect: function(event) {
      this.transform_controls.setMode(event.target.name);
    },

    animate: function() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    }
  }
};
</script>

<style>
#container {
  height: 100%;
}
* {
  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}

html,
body {
  height: 100%;
}

canvas {
  width: 100% !important;
  height: 100%;
}

.hideVideo {
  display: none;
}

.mainContainer {
  width: 100%;
  height: 100%;
  position: relative;
}

.controls {
  position: absolute;
  bottom: 50px;
  left: 50%;
  transform: translate(-50%);
}
</style>