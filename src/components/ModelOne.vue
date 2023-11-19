<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { gsap } from "gsap";
// import GUI from "lil-gui";
import { onMounted, ref } from "vue";
onMounted(() => {
  //DEBUG
  //   const gui = new GUI();

  /**
   * Loaders
   */
  let sceneReady = false;
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
  const cubeTextureLoader = new THREE.CubeTextureLoader(loadingManager);

  /**
   * Base
   */
  // Debug
  const debugObject = {};

  // Canvas
  const canvas = document.querySelector("canvas.webgl");

  // Scene
  const scene = new THREE.Scene();

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

  const environmentMap = cubeTextureLoader.load([
    `/textures/environmentMaps/${1}/px.jpg`,
    `/textures/environmentMaps/${1}/nx.jpg`,
    `/textures/environmentMaps/${1}/py.jpg`,
    `/textures/environmentMaps/${1}/ny.jpg`,
    `/textures/environmentMaps/${1}/pz.jpg`,
    `/textures/environmentMaps/${1}/nz.jpg`,
  ]);

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

  gltfLoader.load("/models/microphone/glTF/scene.gltf", (gltf) => {
    gltf.scene.scale.set(2.5, 2.5, 2.5);
    gltf.scene.rotation.y = Math.PI * 0.5;
    scene.add(gltf.scene);

    updateAllMaterials();
  });

  //RAYCASTER
  const raycaster = new THREE.Raycaster();

  //POINTS
  //DAMAGED HELMET POINTS
  //   const points = [
  //     {
  //       position: new THREE.Vector3(1.70191, -0.14198, -0.28949),
  //       element: document.querySelector(".point-0"),
  //     },
  //     {
  //       position: new THREE.Vector3(0.5, 0.8, -1.6),
  //       element: document.querySelector(".point-1"),
  //     },
  //     {
  //       position: new THREE.Vector3(1.6, -1.3, -0.7),
  //       element: document.querySelector(".point-2"),
  //     },
  //     {
  //       position: new THREE.Vector3(-1.57284, 0.78, -1.72035),
  //       element: document.querySelector(".point-3"),
  //     },
  //     {
  //       position: new THREE.Vector3(-2.60586, 0.3448, 0.86109),
  //       element: document.querySelector(".point-4"),
  //     },
  //     {
  //       position: new THREE.Vector3(-0.4665, -1.64659, -0.09773),
  //       element: document.querySelector(".point-5"),
  //     },
  //     ];

  //MICROPHONE POINTS
  const points = [
    {
      position: new THREE.Vector3(0.86109, 2.18869, 0.86109),
      element: document.querySelector(".point-0"),
    },
    {
      position: new THREE.Vector3(0.56607, 0.27105, 0.27105),
      element: document.querySelector(".point-1"),
    },
    {
      position: new THREE.Vector3(0.3448, -0.17148, -1.7941),
      element: document.querySelector(".point-2"),
    },
    {
      position: new THREE.Vector3(-0.98279, -0.24524, 0.41856),
      element: document.querySelector(".point-3"),
    },
    {
      position: new THREE.Vector3(-0.02397, -2.4579, 1.59865),
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
  //       .max(3)
  //       .step(0.00001)
  //       .name("Point " + (i + 1) + " - X axis");

  //     gui
  //       .add(points[i].position, "y")
  //       .min(-3)
  //       .max(3)
  //       .step(0.00001)
  //       .name("Point " + (i + 1) + " - Y axis");

  //     gui
  //       .add(points[i].position, "z")
  //       .min(-3)
  //       .max(3)
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

  // Controls
  const controls = new OrbitControls(camera, canvas);
  controls.enableDamping = true;

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
    controls.update();
    if (sceneReady) {
      // Update points
      for (const point of points) {
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
</script>
<template>
  <canvas class="webgl"></canvas>

  <div class="loading-bar"></div>

  <div class="point point-3">
    <div class="label">4</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div>
  <div class="point point-4">
    <div class="label">5</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div>
  <div class="point point-2">
    <div class="label">3</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div>
  <div class="point point-1">
    <div class="label">2</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div>
  <div class="point point-0">
    <div class="label">1</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div>
  <!-- <div class="point point-5">
    <div class="label">6</div>
    <p class="text">
      Lorem ipsum dolor sit amet, consequitur sit elit melasem.
    </p>
  </div> -->

  <!-- <button class="switch-environments" @click="switchEnvironments()">
    Switch environments
  </button> -->
</template>
<style lang="scss">
.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}

.loading-bar {
  position: absolute;
  top: 50%;
  width: 100%;
  height: 2px;
  background: #ffffff;
  transform: scaleX(0.3);
  transform-origin: top left;
  transition: transform 0.5s;

  &.ended {
    transform: scaleX(0);
    transform-origin: 100% 0;
    transition: transform 1.5s ease-in-out;
  }
}

.point {
  position: absolute;
  top: 50%;
  left: 50%;

  &.visible .label {
    transform: scale(1, 1);
  }
}

.label {
  z-index: 1;
  background-color: rgba(255, 255, 255, 0.05);
  position: absolute;
  height: 40px;
  width: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  backdrop-filter: blur(2px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: white;
  text-shadow: black 0px 0px 2px;
  transition: transform 0.4s;
  transform: scale(0, 0);
  font-weight: 600;

  &:hover {
    cursor: pointer;
  }
  &:hover ~ .text {
    opacity: 1;
  }
}

.text {
  z-index: 2;
  font-weight: 600;
  color: white;
  /* display: none; */
  opacity: 0;
  position: absolute;
  left: 40px;
  top: 40px;
  border-radius: 20px;
  width: 160px;
  display: flex;
  padding: 1rem;
  backdrop-filter: blur(2px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: opacity 0.4s;
  pointer-events: none;
  text-shadow: black 0px 0px 2px;
}

.switch-environments {
  display: flex;
  background-color: white;
  padding: 1rem 2rem;
  border-radius: 60px;
  position: absolute;
  right: 2rem;
  bottom: 2rem;
  border: none;
  transition: transform 0.2s;

  &:hover {
    cursor: pointer;
    transform: scale(1.1);
  }
}
</style>
