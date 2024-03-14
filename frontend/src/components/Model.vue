<template>
    <div class="model">
        <div ref="container">
            <canvas ref="canvasRef"></canvas>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { ARButton } from "three/examples/jsm/webxr/ARButton";
import { VRButton } from "three/examples/jsm/webxr/VRButton";
import * as THREE from 'three';

export default defineComponent({
    name: 'ModelComponent',
    mounted() {
        const scene = new THREE.Scene();
        const canvasRef = this.$refs.canvasRef;

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100) as any;
        camera.position.y = 0;
        camera.position.z = 2;
        camera.lookAt(new THREE.Vector3(0, 0, 0));

        const renderer = new THREE.WebGLRenderer({
            canvas: canvasRef as HTMLCanvasElement,
            antialias: true,
            alpha: true
        }) as any;

        const vrButton = VRButton.createButton(renderer) as any;
        document.body.appendChild(vrButton);
        const session = renderer.xr.setSession('immersive-vr', {
            requiredFeatures: ['hit-test']
        });

        const raycaster = new THREE.Raycaster();
        const intersection = new THREE.Vector3();

        let controller = renderer.xr.getController(0);
        controller.addEventListener("select", onSelect);
        console.log(controller);
        placePainting();
        function onSelect() {
            placePainting();
        }

        function placePainting() {
            const textureLoader = new THREE.TextureLoader();
            const texture = textureLoader.load('./src/assets/images/scenes/ivewave.jpg');

            const geometry = new THREE.PlaneGeometry(1, 0.5);
            const material = new THREE.MeshBasicMaterial({ map: texture });

            const painting = new THREE.Mesh(geometry, material);
            painting.position.set(0, 0, -2);
            scene.add(painting);
        }

        let resizeCallback = () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        };

        renderer.setSize(window.innerWidth - 200, window.innerHeight - 200);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.render(scene, camera);
        (renderer as any).xr.enabled = true;

        const animate = () => {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        };
        animate();
    },
});

</script>

<style lang="scss" scoped></style>