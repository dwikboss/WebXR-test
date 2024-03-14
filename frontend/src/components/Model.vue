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

export default defineComponent({
    name: 'ModelComponent',
    data() {
        return {
            imageCounter: 0 as number,
        }
    },
    mounted() {
        let reticle: THREE.Mesh;
        let renderer: THREE.WebGLRenderer;
        let controls: OrbitControls;
        let hitTestSource: any = null;
        let hitTestSourceRequested: boolean = false;

        const scene: THREE.Scene = new THREE.Scene();
        const canvasRef: HTMLCanvasElement = this.$refs.canvasRef;
        const camera: THREE.PerspectiveCamera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);

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

        let loop = (timestamp: number, frame: XRFrame) => {
            if (frame) {
                const referenceSpace: XRReferenceSpace | null = renderer.xr.getReferenceSpace();
                let session: XRSession | null = renderer.xr.getSession();

                if (referenceSpace && session && hitTestSourceRequested === false) {
                    session.requestReferenceSpace("viewer").then((referenceSpace: XRReferenceSpace) => {
                        session
                            ?.requestHitTestSource({ space: referenceSpace })
                            .then((source: XRHitTestSource) => {
                                hitTestSource = source;
                            });
                    });

                    hitTestSourceRequested = true;

                    session?.addEventListener("end", () => {
                        hitTestSourceRequested = false;
                        hitTestSource = null;
                    });
                }

                if (hitTestSource) {
                    const hitTestResults: XRHitTestResult[] = frame.getHitTestResults(hitTestSource);
                    if (hitTestResults.length > 0) {
                        const hit: XRHitTestResult = hitTestResults[0];
                        reticle.visible = true;
                        reticle.matrix.fromArray(hit.getPose(referenceSpace!).transform.matrix);
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

        renderer = new THREE.WebGLRenderer({
            canvas: canvasRef as HTMLCanvasElement,
            antialias: true,
            alpha: true
        });
        renderer.setSize(window.innerWidth - 200, window.innerHeight - 200);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.xr.enabled = true;

        renderer.setAnimationLoop(loop);

        document.body.appendChild(
            ARButton.createButton(renderer as any, {
                requiredFeatures: ["hit-test"],
            })
        );

        controls = new OrbitControls(camera, canvasRef);
        controls.enableDamping = true;

        window.addEventListener("resize", resizeCallback);
        placePainting();

        function placePainting() {
            if (reticle.visible) {
                const textureLoader: THREE.TextureLoader = new THREE.TextureLoader();
                const texture: THREE.Texture = textureLoader.load('/images/ivewave.jpg');

                const geometry: THREE.PlaneGeometry = new THREE.PlaneGeometry(1, 0.5);
                const material: THREE.MeshBasicMaterial = new THREE.MeshBasicMaterial({ map: texture });

                const painting: THREE.Mesh = new THREE.Mesh(geometry, material);
                painting.position.setFromMatrixPosition(reticle.matrix);
                scene.add(painting);
            }
        }
        // if (this.imageCounter == 0) {

        // } else if (this.imageCounter == 1) {
        //     const textureLoader = new THREE.TextureLoader();
        //     const texture = textureLoader.load('/images/ivegroup.jpg');

        //     const geometry = new THREE.PlaneGeometry(0.6, 1);
        //     const material = new THREE.MeshBasicMaterial({ map: texture });

        //     const painting = new THREE.Mesh(geometry, material);
        //     painting.position.setFromMatrixPosition(reticle.matrix);
        //     this.imageCounter++;
        //     scene.add(painting);
        // }
        // console.log(this.imageCounter);


        let controller = renderer.xr.getController(0);
        controller.addEventListener("select", placePainting);
    },
});

</script>

<style lang="scss" scoped></style>