<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { gsap } from "gsap";
import { onMounted, ref } from "vue";
// import GUI from "lil-gui";

//NON THREEJS STUFF
let selectedStripe = ref(1);
let zoomToggle = ref(false);
const points = ref([]);
const raycaster = new THREE.Raycaster();
let sceneReady = false;
const showElements = ref(false);

// Scene
const scene = new THREE.Scene();
const controls = ref();
const camera = ref();
const viewSelected = ref("custom");

function lookAtPoints(x, y, z) {
  const target = new THREE.Vector3(x, y, z);

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
      z: Math.abs(z) / -2,
    });
    zoomToggle.value = true;
  } else if (zoomToggle.value === true) {
    gsap.to(camera.value.position, {
      duration: 2,
      x: 4,
      y: 0,
      z: -3,
    });
    zoomToggle.value = false;
  }
}
//

function changeView(stripeIndex) {
  selectedStripe.value = stripeIndex;
  zoomToggle.value = false;
  if (stripeIndex === 1) {
    viewSelected.value = "custom";

    gsap.to(camera.value.position, {
      duration: 2,
      x: -3.5,
      y: 0,
      z: -4,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: -3.141592653589793,
      y: -0.7188299996216245,
      z: -3.141592653589793,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  } else if (stripeIndex === 2) {
    viewSelected.value = "front";

    gsap.to(camera.value.position, {
      duration: 2,
      x: 0,
      y: 0,
      z: 4,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  } else if (stripeIndex === 3) {
    viewSelected.value = "bottom";

    gsap.to(camera.value.position, {
      duration: 2,
      x: 0,
      y: -4,
      z: 0,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: -1.5707963267948966,
      y: 0,
      z: 0,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  }
}

function resetZoom() {
  zoomToggle.value = false;
  if (viewSelected.value === "custom") {
    gsap.to(camera.value.position, {
      duration: 2,
      x: -3.5,
      y: 0,
      z: -4,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: -3.141592653589793,
      y: -0.7188299996216245,
      z: -3.141592653589793,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  } else if (viewSelected.value === "front") {
    gsap.to(camera.value.position, {
      duration: 2,
      x: 0,
      y: 0,
      z: 4,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  } else if (viewSelected.value === "bottom") {
    gsap.to(camera.value.position, {
      duration: 2,
      x: 0,
      y: -4,
      z: 0,
    });
    gsap.to(camera.value.rotation, {
      duration: 2,
      x: -1.5707963267948966,
      y: 0,
      z: 0,
    });
    gsap.to(controls.value.target, {
      duration: 2,
      x: 0,
      y: 0,
      z: 0,
    });
  }
}

let exposedGltf = null;

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
        showElements.value = true;
      }, 500);
      window.setTimeout(() => {
        sceneReady = true;
      }, 1000);
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
    "/textures/environmentMaps/hdri/lab.webp"
  );
  environmentMap.mapping = THREE.EquirectangularReflectionMapping;
  environmentMap.colorSpace = THREE.SRGBColorSpace;
  scene.background = environmentMap;
  scene.environment = environmentMap;

  debugObject.envMapIntensity = 0.5;

  /**
   * Models
   */

  gltfLoader.load("/models/Headphones/glTF/headphones.gltf", (gltf) => {
    gltf.scene.scale.set(0.8, 0.8, 0.8);

    if (window.innerWidth > 768) {
      gltf.scene.scale.set(1, 1, 1);
    }

    scene.add(gltf.scene);

    updateAllMaterials();

    //axis helper
    const axesHelper = new THREE.AxesHelper(5);
    scene.add(axesHelper);

    //get each children of the scene position and add them to the gui
    // let objects = scene.children[3].children;
    // for (let i = 0; i < objects.length; i++) {
    //   const positionFolder = gui.addFolder(`${objects[i].name} Position`);

    //   positionFolder
    //     .add(objects[i].position, "x")
    //     .min(-Math.PI)
    //     .max(Math.PI)
    //     .step(0.001)
    //     .name("X");

    //   positionFolder
    //     .add(objects[i].position, "y")
    //     .min(-Math.PI)
    //     .max(Math.PI)
    //     .step(0.001)
    //     .name("Y");

    //   positionFolder
    //     .add(objects[i].position, "z")
    //     .min(-Math.PI)
    //     .max(Math.PI)
    //     .step(0.001)
    //     .name("Z");
    // }
    exposedGltf = gltf;
    let leather = exposedGltf.scene.children.find(
      (obj) => obj.name === "Around002"
    );
    leather.material.color.r = 0.8;
    leather.material.color.g = 0.8;
    leather.material.color.b = 0.8;

    // console.log(exposedGltf.scene.children);
  });

  //POINTS

  if (window.innerWidth < 768) {
    points.value = [
      {
        position: new THREE.Vector3(-0.81869, -1.42102, -1.16288),
        element: document.querySelector(".point-0"),
      },
      {
        position: new THREE.Vector3(-0.81869, 1.7, -0.47449),
        element: document.querySelector(".point-1"),
      },
      {
        position: new THREE.Vector3(-0.90473, -1.8, -0.73264),
        element: document.querySelector(".point-2"),
      },
      {
        position: new THREE.Vector3(0.47203, -1.07, -0.90473),
        element: document.querySelector(".point-3"),
      },
      {
        position: new THREE.Vector3(1.3, 1.8, -0.5),
        element: document.querySelector(".point-4"),
      },
    ];
  } else if (window.innerWidth > 768) {
    points.value = [
      {
        position: new THREE.Vector3(-0.84, -0.5, -1),
        element: document.querySelector(".point-0"),
      },
      {
        position: new THREE.Vector3(-0.81869, 1.76275, -0.47449),
        element: document.querySelector(".point-1"),
      },
      {
        position: new THREE.Vector3(-0.90473, -2.10941, -0.73264),
        element: document.querySelector(".point-2"),
      },
      {
        position: new THREE.Vector3(0.6, -0.3, -0.90473),
        element: document.querySelector(".point-3"),
      },
      {
        position: new THREE.Vector3(1.3, 1.8, -0.5),
        element: document.querySelector(".point-4"),
      },
    ];
  }

  //GUI DEBUG FOR POINTS POSITION

  /**for (let i = 0; i < points.value.length; i++) {
    gui
      .add(points.value[i].position, "x")
      .min(-3)
      .max(4)
      .step(0.00001)
      .name("Point " + (i + 1) + " - X axis");

    gui
      .add(points.value[i].position, "y")
      .min(-3)
      .max(4)
      .step(0.00001)
      .name("Point " + (i + 1) + " - Y axis");

    gui
      .add(points.value[i].position, "z")
      .min(-3)
      .max(4)
      .step(0.00001)
      .name("Point " + (i + 1) + " - Z axis");
  }*/

  /**
   * Lights
   */
  const directionalLight = new THREE.DirectionalLight("#fffdfa", 2);
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
  camera.value.position.set(-3.5, 0, -4);
  scene.add(camera.value);

  //CONTROLS
  controls.value = new OrbitControls(camera.value, canvas);
  controls.value.enableDamping = true;
  // Disable panning
  controls.value.enablePan = false;
  controls.value.maxDistance = 5;
  controls.value.minDistance = 2.5;

  controls.value.update();

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
    // Update Stats
  };

  tick();
});

//OBJECT PARTS

function customColor(color) {
  let back = exposedGltf.scene.children.find((obj) => obj.name === "Back001");
  let leather = exposedGltf.scene.children.find(
    (obj) => obj.name === "Around002"
  );
  back.material.color.r = color[0];
  back.material.color.g = color[1];
  back.material.color.b = color[2];

  leather.material.color.r = color[0];
  leather.material.color.g = color[1];
  leather.material.color.b = color[2];

  console.log(exposedGltf.scene.children);
}

function metalType(value) {
  let metal = [
    exposedGltf.scene.children.find((obj) => obj.name === "MetalDetail"),
    exposedGltf.scene.children.find((obj) => obj.name === "Metal003"),
    exposedGltf.scene.children.find((obj) => obj.name === "Metal002"),
    exposedGltf.scene.children.find((obj) => obj.name === "V"),
  ];

  metal.forEach((obj) => {
    obj.material.color.r = value[0];
    obj.material.color.g = value[1];
    obj.material.color.b = value[2];
  });
}

function animateElectronics() {
  let electronics = [
    exposedGltf.scene.children.find((obj) => obj.name === "Rotor002"),
    exposedGltf.scene.children.find((obj) => obj.name === "Membrane"),
    exposedGltf.scene.children.find((obj) => obj.name === "Glass002"),
    exposedGltf.scene.children.find(
      (obj) => obj.name === "CircularElectronics002"
    ),
  ];

  electronics.forEach((obj) => {
    console.log(obj.rotation);
    setTimeout(() => {
      gsap.to(obj.position, {
        duration: 4,
        x: obj.position.x - 1,
        y: obj.position.y + 0.4,
      });
      gsap.to(obj.rotation, {
        duration: 14,
        x: obj.rotation.x + Math.PI / 4,
      });
    }, 2000);
  });

  // console.log(
  //   exposedGltf.scene.children.find((obj) => obj.name === "Rotor002")
  // );
}
</script>
<template>
  <canvas
    class="webgl"
    @mousedown="showElements = false"
    @mouseup="showElements = true"
    @touchmove="showElements = false"
    @touchend="showElements = true"
  ></canvas>
  <div class="lateral-stripes" v-if="showElements">
    <span
      v-for="stripe in 3"
      :key="stripe"
      class="lateral-stripes__stripe"
      :class="{
        'lateral-stripes__stripe--selected': selectedStripe === stripe,
      }"
      @click="changeView(stripe)"
    ></span>
  </div>
  <button class="buy-button button-primary" v-if="showElements">
    Add to cart
  </button>
  <div
    @click="resetZoom()"
    class="dezoom"
    :class="{ dezoomVisible: zoomToggle === true }"
  ></div>
  <div class="loading-bar"></div>

  <div
    class="point point-3"
    @mouseover="showElements = false"
    @mouseleave="showElements = true"
    @touchstart="showElements = false"
    @touchmove="showElements = false"
    @touchend="showElements = true"
    @click="
      (zoomToggle = false),
        lookAtPoints(
          points[3].position.x,
          points[3].position.y,
          points[3].position.z
        ),
        animateElectronics()
    "
  >
    <div class="label">
      <img class="label__icon" src="@/assets/icons/add.svg" alt="icone" />
    </div>
    <p class="text">Electronics components, click to see the details.</p>
  </div>

  <div
    class="point point-4"
    @mouseover="showElements = false"
    @mouseleave="showElements = true"
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
      <img class="label__icon" src="@/assets/icons/add.svg" alt="icone" />
    </div>
    <p class="text">
      The high-quality hypoallergenic silicone conforms gently to the contours
      of the head and ears, reducing pressure and discomfort during extended
      use. This material is sweat and oil resistant.
    </p>
  </div>
  <div
    class="point point-2"
    @mouseover="showElements = false"
    @mouseleave="showElements = true"
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
      <img class="label__icon" src="@/assets/icons/add.svg" alt="icone" />
    </div>
    <p class="text">
      The high speed USB-C charging port offers a combination of ultra fast data
      transfer and power delivery.
    </p>
  </div>
  <div
    class="point point-1"
    @mouseover="showElements = false"
    @mouseleave="showElements = true"
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
      <img class="label__icon" src="@/assets/icons/add.svg" alt="icone" />
    </div>
    <p class="text">
      The headband is adjustable and padded for comfort. It provides the right
      amount of pressure to keep the headset comfortably on the user's head.
    </p>
  </div>
  <div
    class="point point-0"
    @mouseover="showElements = false"
    @mouseleave="showElements = true"
    @click="
      lookAtPoints(
        points[0].position.x,
        points[0].position.y,
        points[0].position.z
      )
    "
  >
    <div class="label">
      <img class="label__icon" src="@/assets/icons/add.svg" alt="icone" />
    </div>
    <p class="text">
      The leather cushion provides comfort and help isolate external noise for
      better audio quality.
    </p>
  </div>
  <AsideMenu
    @customColor="customColor"
    @metalType="metalType"
    v-if="showElements"
  />
</template>

<style lang="scss">
.pointNotHovered {
  display: none !important;
}

canvas {
  cursor: grab;
}

.lateral-stripes {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  z-index: 1;
  width: 60px;
  height: fit-content;

  &__stripe {
    width: 100%;
    height: 1.25rem;
    background-color: $primary-color;
    transition: background-color 0.3s;
    box-shadow: $shadow;
    cursor: pointer;
    animation: fading 1s;

    &--selected {
      background-color: $text-color;
    }
  }
}

.buy-button {
  position: absolute;
  bottom: 2rem;
  right: 0;
  left: 0;
  margin: auto;
  z-index: 1;
  max-width: 300px;
  animation: fading 1s;
  text-transform: uppercase;
}
</style>
