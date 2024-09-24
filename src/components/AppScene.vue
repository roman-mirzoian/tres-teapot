<script setup lang="ts">
import { ref, onMounted } from 'vue';
import * as THREE from 'three';
import { Vector3 } from 'three';
import { OrbitControls } from '@tresjs/cientos'
import { TresCanvas } from '@tresjs/core'
import { TeapotGeometry } from 'three/addons/geometries/TeapotGeometry.js';

class Ground {
    mesh: THREE.Mesh;

    constructor() {
        const plane = new THREE.PlaneGeometry(15, 15);
        const material = new THREE.MeshPhongMaterial({ color: 0xa0adaf, shininess: 150 });
        this.mesh = new THREE.Mesh(plane, material);
        this.mesh.rotation.x = -Math.PI / 2;
        this.mesh.receiveShadow = true;
    }
}

class Teapot {
    mesh: THREE.Mesh;
    currentPosition = ref(new THREE.Vector3(0, 1, 0));
    targetPosition = new THREE.Vector3(0, 1, 0);
    isRotated = true;
    speed = 0.05;

    constructor() {
        const geometry = new TeapotGeometry(1, 1);
        const material = new THREE.MeshMatcapMaterial();
        this.mesh = new THREE.Mesh(geometry, material);
    }

    updatePosition() {
        const { value: teapotPosition } = this.currentPosition;
        const [targetX, targetY, targetZ] = this.targetPosition;
        const direction = new THREE.Vector3(
            targetX - teapotPosition.x,
            targetY - teapotPosition.y,
            targetZ - teapotPosition.z
        );

        const distance = direction.length();

        if (distance > 0.05) {
            direction.normalize();
            
            this.currentPosition.value = new THREE.Vector3(
                teapotPosition.x + direction.x * this.speed,
                teapotPosition.y + direction.y * this.speed,
                teapotPosition.z + direction.z * this.speed
            );
        }

        if(!this.isRotated) {
            this.rotateTowards(direction)
        }
    }

    setTargetPosition(newPosition: number[]) {
        this.targetPosition = newPosition;
    }

    rotateTowards(direction) {
        this.isRotated = true;
        const angle = Math.atan2(direction.x, direction.z);
        this.mesh.rotation.y = angle - Math.PI / 2;
    }
}

const camera = ref<THREE.PerspectiveCamera>();

const teapot = new Teapot();
const groundMesh = new Ground().mesh;

const onSceneClick = (event) => {
    const pointer = new THREE.Vector2(
        (event.clientX / window.innerWidth) * 2 - 1,
        -(event.clientY / window.innerHeight) * 2 + 1
    );

    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(pointer, camera.value);

    const intersects = raycaster.intersectObject(groundMesh);
    if (intersects.length > 0) {
        const { x: iX, z: iZ } = intersects[0].point;
        teapot.setTargetPosition([iX, 1, iZ]);
    }

    teapot.isRotated = false;
};

const animate = () => {
    teapot.updatePosition();
    requestAnimationFrame(animate);
};

onMounted(() => {
    animate();
});
</script>

<template>
    <TresCanvas clear-color="#82DBC5" window-size @pointerdown="onSceneClick">
        <TresPerspectiveCamera ref="camera" :position="[10, 10, 10]" :look-at="[1, 1, 1]" />

        <TresMesh :position="teapot.currentPosition.value">
            <primitive :object="teapot.mesh" />
        </TresMesh>

        <TresMesh>
            <primitive :object="groundMesh" />
        </TresMesh>

        <TresAmbientLight :intensity="1" />
        <TresAmbientLight />
    </TresCanvas>
</template>

<style>
html,
body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
}

#app {
    height: 100%;
    width: 100%;
    background-color: #000;
}
</style>