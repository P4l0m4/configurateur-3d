<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";

import { gsap } from "gsap";
// import GUI from "lil-gui";
const points = ref([]);

const raycaster = new THREE.Raycaster();
let sceneReady = false;

// Scene
const scene = new THREE.Scene();

const controls = ref();

import { onMounted, ref } from "vue";
onMounted(() => {
  //DEBUG
  //   const gui = new GUI();

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

  //HDRI only
  //   const cubeTextureLoader = new THREE.CubeTextureLoader(loadingManager);

  //   const rgbeLoader = new RGBELoader(loadingManager);

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

  //   const environmentMap = cubeTextureLoader.load([
  //     `/textures/environmentMaps/${1}/px.jpg`,
  //     `/textures/environmentMaps/${1}/nx.jpg`,
  //     `/textures/environmentMaps/${1}/py.jpg`,
  //     `/textures/environmentMaps/${1}/ny.jpg`,
  //     `/textures/environmentMaps/${1}/pz.jpg`,
  //     `/textures/environmentMaps/${1}/nz.jpg`,
  //   ]);

  //   environmentMap.colorSpace = THREE.SRGBColorSpace;

  //   scene.background = environmentMap;
  //   scene.environment = environmentMap;

  //HDRI (don't use, too heavy)
  //   rgbeLoader.load(
  //     "/textures/environmentMaps/hdri/cyberpunk1.hdr",
  //     (environmentMap) => {
  //       environmentMap.mapping = THREE.EquirectangularReflectionMapping;
  //       scene.background = environmentMap;
  //       scene.environment = environmentMap;
  //     }
  //   );

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
  //   gltfLoader.load("/models/DamagedHelmet/glTF/DamagedHelmet.gltf", (gltf) => {
  //     gltf.scene.scale.set(2.5, 2.5, 2.5);
  //     gltf.scene.rotation.y = Math.PI * 0.5;
  //     scene.add(gltf.scene);

  //     updateAllMaterials();
  //   });

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
      position: new THREE.Vector3(-0.47449, 0.73983, -0.73264),
      element: document.querySelector(".point-0"),
    },
    {
      position: new THREE.Vector3(-2.02336, 0.6018, -2.19545),
      element: document.querySelector(".point-1"),
    },
    {
      position: new THREE.Vector3(-2.88384, -0.17148, -2.88384),
      element: document.querySelector(".point-2"),
    },
    {
      position: new THREE.Vector3(1.93485, 0.04978, 1.24646),
      element: document.querySelector(".point-3"),
    },
    {
      position: new THREE.Vector3(2.96742, 0.12784, 2.62323),
      element: document.querySelector(".point-4"),
    },
    // {
    //   position: new THREE.Vector3(-0.02397, -2.4579, 1.59865),
    //   element: document.querySelector(".point-5"),
    // },
  ];

  //GUI DEBUG FOR POINTS POSITION

  //   for (let i = 0; i < points.length; i++) {
  //     gui
  //       .add(points[i].position, "x")
  //       .min(-3)
  //       .max(4)
  //       .step(0.00001)
  //       .name("Point " + (i + 1) + " - X axis");

  //     gui
  //       .add(points[i].position, "y")
  //       .min(-3)
  //       .max(4)
  //       .step(0.00001)
  //       .name("Point " + (i + 1) + " - Y axis");

  //     gui
  //       .add(points[i].position, "z")
  //       .min(-3)
  //       .max(4)
  //       .step(0.00001)
  //       .name("Point " + (i + 1) + " - Z axis");
  //   }

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
    camera.aspect = sizes.width / sizes.height;
    camera.updateProjectionMatrix();

    // Update renderer
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  });

  /**
   * Camera
   */
  // Base camera
  const camera = new THREE.PerspectiveCamera(
    75,
    sizes.width / sizes.height,
    0.1,
    100
  );
  camera.position.set(4, 1, -4);
  scene.add(camera);

  //CONTROLS
  controls.value = new OrbitControls(camera, canvas);
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
        screenPosition.project(camera);

        //RAYCASTER IS POSITIONNED TO SHOOT FROM THE CAMERA TO THE POINTS
        raycaster.setFromCamera(screenPosition, camera);

        //TEST IF THE RAYCASTER INTERSECTS WITH THE SCENE AND THE OBJECTS (children of the scene)
        const intersects = raycaster.intersectObjects(scene.children, true);

        //IF THE RAYCASTER INTERSECTS WITH THE SCENE AND THE OBJECTS (the points must be before the model to be visible)
        if (intersects.length === 0) {
          point.element.classList.add("visible");
        } else {
          const intersectionDistance = intersects[0].distance;
          const pointDistance = point.position.distanceTo(camera.position);

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
    renderer.render(scene, camera);

    // Call tick again on the next frame
    window.requestAnimationFrame(tick);
  };

  tick();
});

function lookAtPoints(x, y, z) {
  const target = new THREE.Vector3(x, y, z);
  controls.value.target = target;
}
</script>
<template>
  <canvas class="webgl"></canvas>

  <div class="loading-bar"></div>

  <div
    class="point point-3"
    @click="
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
