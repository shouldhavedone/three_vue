<template>
  <div class="container" id="container">
    <ul id="colorList">
      <li
        v-for="(item, index) in colorList"
        :key="index"
        :style="`background-color: ${item}`"
        @click="changeColor(item)"
      ></li>
    </ul>
    <div class="actionList">
      <span @click="changeAnimate(0)" style="background-color: rgb(81, 45, 168)"
        >Idle</span
      >
      <span @click="changeAnimate(1)" style="background-color: rgb(81, 45, 168)"
        >Run</span
      >
      <span @click="changeAnimate(2)" style="background-color: rgb(81, 45, 168)"
        >TPose</span
      >
      <span @click="changeAnimate(3)" style="background-color: rgb(81, 45, 168)"
        >Walk</span
      >
    </div>
  </div>
</template>

<script>
import * as Three from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
let scene,
  mixer,
  mixers = [],
  group = new Three.Group(),
  clock = new Three.Clock();

export default {
  name: "Demo6Vue",
  data() {
    return {
      renderer: null,
      camera: null,
      controls: null,
      loader: null,
      skeleton: "",
      colorList: [
        "rgb(216, 27, 67)",
        "rgb(142, 36, 170)",
        "rgb(81, 45, 168)",
        "rgb(48, 63, 159)",
        "rgb(30, 136, 229)",
        "rgb(67, 160, 71)",
        "rgb(94, 53, 177)",
        "rgb(251, 140, 0)",
        "rgb(66, 165, 245)",
        "rgb(240, 98, 125)",
        "rgb(38, 166, 154)",
        "rgb(129, 199, 132)",
      ],
    };
  },

  mounted() {
    this.init();
    this.animate();
    document.onkeydown = function (event) {
      let model;
      scene.traverse((child) => {
        if (child.type == "Group") {
          model = child;
        }
      });

      switch (event.keyCode) {
        case 37: {
          model.position.x -= 1;
          break;
        }
        case 38: {
          model.position.z += 1;
          break;
        }
        case 39: {
          model.position.x += 1;
          break;
        }
        case 40: {
          model.position.z -= 1;
          break;
        }
      }
    };
  },

  methods: {
    init() {
      this.setScene();
      this.setCamera();
      this.setLight();
      this.setRenderer();
      this.setControls();
      this.setLoader();
      window.addEventListener("click", this.onMouseDblclick, false);
      window.addEventListener("resize", this.onWindowResize, false);
    },

    setScene() {
      scene = new Three.Scene();
      scene.background = new Three.Color(0xeeeeee);
      scene.add(new Three.HemisphereLight());
      scene.fog = new Three.Fog(0xa0a0a0, 200, 1000);

      let grid = new Three.GridHelper(2000, 20, 0x000000, 0x000000);
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      scene.add(grid);
    },

    setCamera() {
      this.camera = new Three.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        2000
      );
      this.camera.position.set(10, 23, -40);
    },

    setLight() {
      let directionalLight = new Three.DirectionalLight(0xffeedd);
      directionalLight.position.set(0, 200, 100);
      directionalLight.castShadow = true;
      directionalLight.shadow.camera.top = 180;
      directionalLight.shadow.camera.bottom = -100;
      directionalLight.shadow.camera.left = -120;
      directionalLight.shadow.camera.right = 120;
      scene.add(directionalLight);
    },

    setControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.target.set(0, -0.2, -0.2);
      this.controls.update();
    },

    setRenderer() {
      this.renderer = new Three.WebGLRenderer({ antialias: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.gammaOutput = true;
      container.appendChild(this.renderer.domElement);
    },

    animate() {
      requestAnimationFrame(this.animate);
      let delta = clock.getDelta();
      for (let i = 0; i < mixers.length; i++) {
        mixers[i]._mixer.update(delta);
      }
      this.renderer.render(scene, this.camera);
    },

    setLoader() {
      let that = this;
      this.loader = new GLTFLoader();
      this.loader.load("/models/Soldier.glb", function (gltf) {
        gltf.scene.scale.set(10, 10, 10);
        group.add(gltf.scene);
        scene.add(group);
        mixer = new Three.AnimationMixer(gltf.scene);
        for (let i = 0; i < gltf.animations.length; i++) {
          let temp = mixer.clipAction(gltf.animations[i]);
          mixers.push(temp);
        }
      });
      this.loader.load("/models/Schnauzer.glb", function (gltf) {
        gltf.scene.scale.set(100, 100, 100);
        gltf.scene.position.set(0, 0, -20);
        group.add(gltf.scene);
        scene.add(group);
      });
    },

    changeAnimate(value) {
      for (let i = 0; i < mixers.length; i++) {
        if (i === value) {
          mixers[value].setDuration(3.6).play();
        } else {
          mixers[i].stop();
        }
      }
    },

    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },

    changeColor(value) {
      scene.traverse((child) => {
        if (child.isMesh) {
          child.material.color.set(value);
        }
      });
    },

    getIntersects(event) {
      event.preventDefault();
      let raycaster = new Three.Raycaster();
      let mouse = new Three.Vector2();
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, this.camera);
      let intersects = raycaster.intersectObjects(scene.children);
      return intersects;
    },

    onMouseDblclick(event) {
      let intersects = this.getIntersects(event);
      if (intersects.length != 0 && intersects[0].object instanceof Three.Mesh) {
        let selectObject = intersects[0].object;
        if (selectObject.name === "schnauzer1") this.changeColor("rgb(216, 27, 67)");
      }
    },
  },
};
</script>

<style>
.container {
  position: relative;
}
ul {
  position: absolute;
}
li {
  list-style: none;
  display: inline-block;
  width: 60px;
  height: 24px;
  cursor: pointer;
  margin-right: 10px;
}
.actionList {
  position: absolute;
  left: 40px;
  top: 60px;
}

.actionList span {
  width: 60px;
  height: 24px;
  display: inline-block;
  line-height: 24px;
  text-align: center;
  margin-right: 10px;
  cursor: pointer;
}
</style>
