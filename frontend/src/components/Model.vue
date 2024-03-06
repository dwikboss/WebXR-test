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
import * as THREE from 'three';

export default defineComponent({
    name: 'ModelComponent',
    mounted() {
        const scene = new THREE.Scene();
        const canvasRef = this.$refs.canvasRef;

        const camera = new THREE.PerspectiveCamera(75, canvasRef.clientWidth / canvasRef.clientHeight, 0.1, 1000);
        camera.position.z = 2;
        camera.lookAt(new THREE.Vector3(0, 0, 0));

        const renderer = new THREE.WebGLRenderer({ 
            canvas: canvasRef,
            antialias: true,
            alpha: true
        });
        renderer.setSize(canvasRef.clientWidth, canvasRef.clientHeight);
        (renderer as any).xr.enabeled = true;

        (document.body.appendChild(ARButton.createButton(renderer) as any));

        const geometry = new THREE.BoxGeometry();
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);

        const animate = () => {
            requestAnimationFrame(animate);
            cube.rotation.x += 0.01;
            cube.rotation.y += 0.01;
            renderer.render(scene, camera);
        };
        animate();
    },
});

</script>

<style lang="scss" scoped></style>