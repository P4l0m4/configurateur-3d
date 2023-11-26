<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

import { gsap } from "gsap";
// import GUI from "lil-gui";
const points = ref([]);

const raycaster = new THREE.Raycaster();
let sceneReady = false;

// Scene
const scene = new THREE.Scene();

const controls = ref();

let camera = ref();

import { onMounted, ref } from "vue";
onMounted(() => {
  //DEBUG
  // const gui = new GUI();

  /**
   * Loaders
   */
  // let sceneReady = false;
  const loadingBarElement = document.querySelector(".loading-bar");
  const loadingManager = new THREE.LoadingManager(
    // Loaded
    () => {
      // Wait a little
      window.setTimeout(() => {
        // Animate overlay
        gsap.to(overlayMaterial.uniforms.uAlpha, {
          duration: 3,
          value: 0,
          delay: 1,
        });

        // Update loadingBarElement
        loadingBarElement.classList.add("ended");
        loadingBarElement.style.transform = "";
      }, 500);
      window.setTimeout(() => {
        sceneReady = true;
      }, 2000);
    },

    // Progress
    (itemUrl, itemsLoaded, itemsTotal) => {
      // Calculate the progress and update the loadingBarElement
      const progressRatio = itemsLoaded / itemsTotal;
      loadingBarElement.style.transform = `scaleX(${progressRatio})`;
    }
  );
  const gltfLoader = new GLTFLoader(loadingManager);

  //IMAGE LOADER
  const textureLoader = new THREE.TextureLoader(loadingManager);

  /**
   * Base
   */
  // Debug
  const debugObject = {};

  // Canvas
  const canvas = document.querySelector("canvas.webgl");

  /**
   * Overlay
   */
  const overlayGeometry = new THREE.PlaneGeometry(2, 2, 1, 1);
  const overlayMaterial = new THREE.ShaderMaterial({
    // wireframe: true,
    transparent: true,
    uniforms: {
      uAlpha: { value: 1 },
    },
    vertexShader: `
        void main()
        {
            gl_Position = vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform float uAlpha;

        void main()
        {
            gl_FragColor = vec4(0.0, 0.0, 0.0, uAlpha);
        }
    `,
  });
  const overlay = new THREE.Mesh(overlayGeometry, overlayMaterial);
  scene.add(overlay);

  /**
   * Update all materials
   */
  const updateAllMaterials = () => {
    scene.traverse((child) => {
      if (
        child instanceof THREE.Mesh &&
        child.material instanceof THREE.MeshStandardMaterial
      ) {
        // child.material.envMap = environmentMap
        child.material.envMapIntensity = debugObject.envMapIntensity;
        child.material.needsUpdate = true;
        child.castShadow = true;
        child.receiveShadow = true;
      }
    });
  };

  /**
   * Environment maps
   */

  //LDR

  const environmentMap = textureLoader.load(
    "/textures/environmentMaps/hdri/cyberpunk4.webp"
  );
  environmentMap.mapping = THREE.EquirectangularReflectionMapping;
  environmentMap.colorSpace = THREE.SRGBColorSpace;
  scene.background = environmentMap;
  scene.environment = environmentMap;

  debugObject.envMapIntensity = 2.5;

  /**
   * Models
   */

  gltfLoader.load("/models/Thermometer/thermometer.gltf", (gltf) => {
    gltf.scene.scale.set(2.5, 2.5, 2.5);
    gltf.scene.rotation.y = Math.PI * 0.5;
    scene.add(gltf.scene);

    updateAllMaterials();
  });

  //RAYCASTER
  // const raycaster = new THREE.Raycaster();

  //POINTS

  points.value = [
    {
      position: new THREE.Vector3(-0.47449, 0.73983, -0.38845),
      element: document.querySelector(".point-0"),
    },
    {
      position: new THREE.Vector3(-0.81869, 1.59066, -0.47449),
      element: document.querySelector(".point-1"),
    },
    {
      position: new THREE.Vector3(-1.50707, 1.93485, -0.90473),
      element: document.querySelector(".point-2"),
    },
    {
      position: new THREE.Vector3(0.55808, -1.16288, 0.38598),
      element: document.querySelector(".point-3"),
    },
    {
      position: new THREE.Vector3(2.0209, -2.53965, 1.59066),
      element: document.querySelector(".point-4"),
    },
  ];

  //GUI DEBUG FOR POINTS POSITION

  // for (let i = 0; i < points.value.length; i++) {
  //   gui
  //     .add(points.value[i].position, "x")
  //     .min(-3)
  //     .max(4)
  //     .step(0.00001)
  //     .name("Point " + (i + 1) + " - X axis");

  //   gui
  //     .add(points.value[i].position, "y")
  //     .min(-3)
  //     .max(4)
  //     .step(0.00001)
  //     .name("Point " + (i + 1) + " - Y axis");

  //   gui
  //     .add(points.value[i].position, "z")
  //     .min(-3)
  //     .max(4)
  //     .step(0.00001)
  //     .name("Point " + (i + 1) + " - Z axis");
  // }

  /**
   * Lights
   */
  const directionalLight = new THREE.DirectionalLight("#ffffff", 3);
  directionalLight.castShadow = true;
  directionalLight.shadow.camera.far = 15;
  directionalLight.shadow.mapSize.set(1024, 1024);
  directionalLight.shadow.normalBias = 0.05;
  directionalLight.position.set(0.25, 3, -2.25);
  scene.add(directionalLight);

  /**
   * Sizes
   */
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight,
  };

  window.addEventListener("resize", () => {
    // Update sizes
    sizes.width = window.innerWidth;
    sizes.height = window.innerHeight;

    // Update camera
    camera.value.aspect = sizes.width / sizes.height;
    camera.value.updateProjectionMatrix();

    // Update renderer
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  });

  /**
   * Camera
   */
  // Base camera
  camera.value = new THREE.PerspectiveCamera(
    75,
    sizes.width / sizes.height,
    0.1,
    100
  );
  camera.value.position.set(4, 1, -4);
  scene.add(camera.value);

  //CONTROLS
  controls.value = new OrbitControls(camera.value, canvas);
  controls.value.enableDamping = true;

  /**
   * Renderer
   */
  const renderer = new THREE.WebGLRenderer({
    canvas: canvas,
    antialias: true,
  });
  renderer.toneMapping = THREE.ReinhardToneMapping;
  renderer.toneMappingExposure = 3;
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

  /**
   * Animate
   */
  const tick = () => {
    // Update controls
    controls.value.update();
    if (sceneReady) {
      // Update points
      for (const point of points.value) {
        const screenPosition = point.position.clone();

        //PROJECT THE POINTS ON THE SCREEN
        screenPosition.project(camera.value);

        //RAYCASTER IS POSITIONNED TO SHOOT FROM THE CAMERA TO THE POINTS
        raycaster.setFromCamera(screenPosition, camera.value);

        //TEST IF THE RAYCASTER INTERSECTS WITH THE SCENE AND THE OBJECTS (children of the scene)
        const intersects = raycaster.intersectObjects(scene.children, true);

        //IF THE RAYCASTER INTERSECTS WITH THE SCENE AND THE OBJECTS (the points must be before the model to be visible)
        if (intersects.length === 0) {
          point.element.classList.add("visible");
        } else {
          const intersectionDistance = intersects[0].distance;
          const pointDistance = point.position.distanceTo(
            camera.value.position
          );

          if (intersectionDistance < pointDistance) {
            point.element.classList.remove("visible");
          } else {
            point.element.classList.add("visible");
          }
        }

        //MAKE THE POINTS MOVE
        const translateX = screenPosition.x * sizes.width * 0.5;
        const translateY = -screenPosition.y * sizes.height * 0.5;

        point.element.style.transform = `translateX(${translateX}px) translateY(${translateY}px)`;
      }
    }

    // Render
    renderer.render(scene, camera.value);

    // Call tick again on the next frame
    window.requestAnimationFrame(tick);
  };

  tick();
});
let zoomToggle = ref(false);
function lookAtPoints(x, y, z) {
  const target = new THREE.Vector3(x, y, z);
  // controls.value.target = target;
  gsap.to(controls.value.target, {
    duration: 2,
    x: target.x,
    y: target.y,
    z: target.z,
  });

  if (zoomToggle.value === false) {
    gsap.to(camera.value.position, {
      duration: 2,
      x: x / 2,
      y: y,
      z: z * 2,
    });

    zoomToggle.value = true;
  } else if (zoomToggle.value === true) {
    gsap.to(camera.value.position, {
      duration: 2,
      x: 4,
      y: 1,
      z: -4,
    });
    zoomToggle.value = false;
  }
}
</script>
<template>
  <canvas class="webgl"></canvas>
  <div
    @click="lookAtPoints(0, 0, 0)"
    class="dezoom"
    :class="{ dezoomVisible: zoomToggle === true }"
  ></div>
  <div class="loading-bar"></div>

  <div
    class="point point-3"
    @click="
      (zoomToggle = false),
        lookAtPoints(
          points[3].position.x,
          points[3].position.y,
          points[3].position.z
        )
    "
  >
    <div class="label">
      <img
        class="label__icon"
        src="@/assets/icons/self_improvement.svg"
        alt="icone"
      />
    </div>
    <p class="text">
      Flexible and soft probe, provides added comfort during insertion.
    </p>
  </div>
  <div
    class="point point-4"
    @click="
      (zoomToggle = false),
        lookAtPoints(
          points[4].position.x,
          points[4].position.y,
          points[4].position.z
        )
    "
  >
    <div class="label">
      <img
        class="label__icon"
        src="@/assets/icons/heat_pump_balance.svg"
        alt="icone"
      />
    </div>
    <p class="text">
      The tip, inserted into the rectum to accurately measure body temperature.
    </p>
  </div>
  <div
    class="point point-2"
    @click="
      (zoomToggle = false),
        lookAtPoints(
          points[2].position.x,
          points[2].position.y,
          points[2].position.z
        )
    "
  >
    <div class="label">
      <img class="label__icon" src="@/assets/icons/charger.svg" alt="icone" />
    </div>
    <p class="text">
      Battery compartment, allows for quick and easy battery changes.
    </p>
  </div>
  <div
    class="point point-1"
    @click="
      (zoomToggle = false),
        lookAtPoints(
          points[1].position.x,
          points[1].position.y,
          points[1].position.z
        )
    "
  >
    <div class="label">
      <img
        class="label__icon"
        src="@/assets/icons/power_settings_new.svg"
        alt="icone"
      />
    </div>
    <p class="text">
      Power button, designed for easy manipulation during use to turn the device
      on and off.
    </p>
  </div>
  <div
    class="point point-0"
    @click="
      lookAtPoints(
        points[0].position.x,
        points[0].position.y,
        points[0].position.z
      )
    "
  >
    <div class="label">
      <img
        class="label__icon"
        src="@/assets/icons/browse_activity.svg"
        alt="icone"
      />
    </div>
    <p class="text">
      The display, shows the temperature reading by providing a clear and
      easy-to-read numerical value.
    </p>
  </div>
</template>
