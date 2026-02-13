<script lang="ts">
  import { browser } from '$app/environment';
  // import * as dat from 'dat.gui';
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import { objectDirection } from 'three/tsl';

  if (browser) {
    // const gui = new dat.GUI();
    const canvas: SVGElement = document.querySelector('canvas#main-scene')!;
    const scene = new THREE.Scene();

    /**
     * Lights
     */
    const ambientLight = new THREE.AmbientLight(0xffffff, 2.4);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1.8);
    directionalLight.castShadow = true;
    directionalLight.shadow.mapSize.set(1024, 1024);
    directionalLight.shadow.camera.far = 15;
    directionalLight.shadow.camera.left = - 7;
    directionalLight.shadow.camera.top = 7;
    directionalLight.shadow.camera.right = 7;
    directionalLight.shadow.camera.bottom = - 7;
    directionalLight.position.set(5, 5, 5);
    scene.add(directionalLight);

    /**
     * Sizes
     */
    const dimensions = {
        width: window.innerWidth,
        height: window.innerHeight
    };

    window.addEventListener('resize', () => {
        // Update sizes
        dimensions.width = window.innerWidth;
        dimensions.height = window.innerHeight;

        // Update camera
        camera.aspect = dimensions.width / dimensions.height;
        camera.updateProjectionMatrix();

        // Update renderer
        renderer.setSize(dimensions.width, dimensions.height);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    })

    /**
     * Camera
     */
    // Base camera
    const camera = new THREE.PerspectiveCamera(75, dimensions.width / dimensions.height, 0.1, 100);
    camera.position.set(2, 2, 2);
    scene.add(camera);

    // Controls
    const controls = new OrbitControls(camera, canvas);
    controls.target.set(0, 0.75, 0);
    controls.enableDamping = true;

    const gltfLoader = new GLTFLoader();
    gltfLoader.load(
      '/models/hand_sculpture/scene.gltf',
      gltf => {
        gltf.scene.traverse(obj => {
          if (obj.isMesh) obj.material.wireframe = true;
        });

        scene.add(gltf.scene);
      },
      () => {},
      err => {
        console.log('GLTF Loader error', err);
      }
    );

    /**
     * Renderer
     */
    const renderer = new THREE.WebGLRenderer({
        canvas: canvas,
        alpha: true
    });
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    renderer.setClearColor(0x000000, 0);
    renderer.setSize(dimensions.width, dimensions.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    /**
     * Animate
     */
    const clock = new THREE.Clock();
    let previousTime = 0;

    const tick = () => {
        const elapsedTime = clock.getElapsedTime();
        const deltaTime = elapsedTime - previousTime;
        previousTime = elapsedTime;

        // Update controls
        controls.update();

        // Render
        renderer.render(scene, camera);

        // Call tick again on the next frame
        window.requestAnimationFrame(tick);
    }

    tick();
  }
</script>

<canvas id="main-scene" class="h-dvh w-dvw absolute top-0 left-0"></canvas>
