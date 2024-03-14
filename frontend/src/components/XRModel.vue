<template>
    <div class="model">
        <div ref="container">
            <canvas ref="canvasRef"></canvas>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { ARButton } from "three/examples/jsm/webxr/ARButton";
import * as THREE from 'three';

let reticle;
let renderer;
let controls;

let hitTestSource = null;
let hitTestSourceRequested = false;

let scene = new THREE.Scene();
let canvasRef = ref();
let camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100) as any;

camera.position.y = 0;
camera.position.z = 2;
camera.lookAt(new THREE.Vector3(0, 0, 0));

scene.add(camera);

reticle = new THREE.Mesh(
    new THREE.RingGeometry(0.15, 0.2, 32).rotateX(-Math.PI / 2),
    new THREE.MeshStandardMaterial()
);
reticle.matrixAutoUpdate = false;
reticle.visible = false;

scene.add(reticle);

let loop = (timestamp, frame) => {
    if (frame) {
        const referenceSpace = renderer.xr.getReferenceSpace();
        let session = renderer.xr.getSession();

        if (hitTestSourceRequested === false) {
            session.requestReferenceSpace("viewer").then((referenceSpace) => {
                session
                    .requestHitTestSource({ space: referenceSpace })
                    .then((source) => {
                        hitTestSource = source;
                    });
            });

            hitTestSourceRequested = true;

            session.addEventListener("end", () => {
                hitTestSourceRequested = false;
                hitTestSource = null;
            });
        }

        if (hitTestSource) {
            const hitTestResults = frame.getHitTestResults(hitTestSource);
            if (hitTestResults.length > 0) {
                const hit = hitTestResults[0];
                reticle.visible = true;
                reticle.matrix.fromArray(hit.getPose(referenceSpace).transform.matrix);
            } else {
                reticle.visible = false;
            }
        }
    }
    controls.update();
    renderer.render(scene, camera);
};

let resizeCallback = () => {
    renderer.setSize(window.innerWidth, window.innerHeight);

    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
};

export default defineComponent({
    name: 'ModelComponent',
    mounted() {
        placePainting();
        renderer = new THREE.WebGLRenderer({
            canvas: canvasRef.value as HTMLCanvasElement,
            antialias: true,
            alpha: true
        }) as any;

        renderer.setSize(window.innerWidth - 200, window.innerHeight - 200);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.render(scene, camera);
        (renderer as any).xr.enabled = true;

        renderer.setAnimationLoop(loop);

        document.body.appendChild(
            ARButton.createButton(renderer as any, {
                requiredFeatures: ["hit-test"],
            })
        );

        controls = new OrbitControls(camera, canvasRef.value);
        controls.enableDamping = true;

        window.addEventListener("resize", resizeCallback);

        function placePainting() {
            console.log('select');
            if (reticle.visible && loadedModel) {
                const textureLoader = new THREE.TextureLoader();
                const texture = textureLoader.load('/images/ivewave.jpg');

                const geometry = new THREE.PlaneGeometry(1, 0.5);
                const material = new THREE.MeshBasicMaterial({ map: texture });

                const painting = new THREE.Mesh(geometry, material);
                painting.position.setFromMatrixPosition(reticle.matrix);
                scene.add(painting);
            }
        }

        let controller = renderer.xr.getController(0);
        controller.addEventListener("select", placePainting);


        let resizeCallback = () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        };
    },
});

</script>

<style lang="scss" scoped></style>