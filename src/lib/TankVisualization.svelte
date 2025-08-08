<!-- TankVisualization.svelte -->
<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

    export let level = 0;

    let canvas;
    let levelTextElement;
    let liquidGroup;
    let liquidUniforms;
    let camera, scene, renderer, controls;
    const tankHeight = 8;

    function updateLevel(levelPercent) {
        if (!liquidGroup || !levelTextElement || !renderer) return;

        const liquidBody = liquidGroup.children[0];
        const liquidTop = liquidGroup.children[1];
        
        let targetHeight = tankHeight * (levelPercent / 100);
        if (targetHeight < 0.01) targetHeight = 0.01;

        liquidGroup.position.y = -tankHeight / 2;
        
        liquidBody.scale.y = targetHeight;
        liquidBody.position.y = targetHeight / 2;
        liquidTop.position.y = targetHeight;

        levelTextElement.textContent = `${levelPercent.toFixed(1)}%`;
        
        // Render the scene since the animation loop is removed
        renderer.render(scene, camera);
    }
    
    afterUpdate(() => {
        updateLevel(level);
    });

    onMount(() => {
        const clock = new THREE.Clock();
        const tankRadius = 3;

        Promise.all([
            new Promise(res => new RGBELoader().load('https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/aerodynamics_workshop_1k.hdr', res)),
        ]).then(([loadedEnvMap]) => {
            const envMap = loadedEnvMap;
            envMap.mapping = THREE.EquirectangularReflectionMapping;
            
            init(envMap);
        });

        function init(envMap) {
            scene = new THREE.Scene();
            scene.environment = envMap;

            camera = new THREE.PerspectiveCamera(50, canvas.clientWidth / canvas.clientHeight, 0.1, 1000);
            camera.position.set(0, 0, 14);

            renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true, alpha: true });
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(canvas.clientWidth, canvas.clientHeight);
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.0;

            const tankGeometry = new THREE.CylinderGeometry(tankRadius, tankRadius, tankHeight, 64);
            const tankMaterial = new THREE.MeshPhysicalMaterial({
                color: 0xffffff,
                transparent: true,
                opacity: 0.3,
                roughness: 0.2,
                metalness: 0.1,
                clearcoat: 0.5,
                clearcoatRoughness: 0.1
            });
            const tankCylinder = new THREE.Mesh(tankGeometry, tankMaterial);
            scene.add(tankCylinder);

            const liquidRadius = tankRadius * 0.95;
            const liquidBodyGeom = new THREE.CylinderGeometry(liquidRadius, liquidRadius, 1, 64);
            const liquidTopGeom = new THREE.CircleGeometry(liquidRadius, 64);
            liquidTopGeom.rotateX(-Math.PI / 2);

            const liquidBodyMaterial = new THREE.MeshStandardMaterial({ color: 0x00aaff, roughness: 0.1, metalness: 0.0, envMap: envMap });
            const liquidTopMaterial = liquidBodyMaterial.clone();
            liquidUniforms = { time: { value: 0.0 } }; // Kept for static ripple
            liquidTopMaterial.onBeforeCompile = shader => {
                shader.uniforms.time = liquidUniforms.time;
                shader.vertexShader = 'uniform float time;\n' + shader.vertexShader;
                shader.vertexShader = shader.vertexShader.replace(
                    '#include <begin_vertex>',
                    `#include <begin_vertex>
                    float ripple = sin((position.x * 0.5 + time * 1.5)) * cos((position.z * 0.5 + time * 1.0)) * 0.1;
                    transformed.y += ripple;`
                );
            };

            const liquidBody = new THREE.Mesh(liquidBodyGeom, liquidBodyMaterial);
            const liquidTop = new THREE.Mesh(liquidTopGeom, liquidTopMaterial);
            
            liquidGroup = new THREE.Group();
            liquidGroup.add(liquidBody);
            liquidGroup.add(liquidTop);
            scene.add(liquidGroup);

            // Create 3D Scale with tick marks only
            const scaleGroup = new THREE.Group();
            const textMaterial = new THREE.MeshBasicMaterial({ color: 0x333333 });
            for(let i = 0; i <= 4; i++) {
                const percent = i * 25;
                const y = (-tankHeight / 2) + (tankHeight * (percent / 100));

                const tickGeom = new THREE.PlaneGeometry(0.5, 0.05);
                const tickMesh = new THREE.Mesh(tickGeom, textMaterial);
                tickMesh.position.set(tankRadius + 0.5, y, 0);
                scaleGroup.add(tickMesh);
            }
            scene.add(scaleGroup);

            controls = new OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.minDistance = 8;
            controls.maxDistance = 25;
            controls.target.set(0, 0, 0);
            
            // Listen for changes to re-render the scene
            controls.addEventListener('change', () => renderer.render(scene, camera));

            updateLevel(level);
            renderer.render(scene, camera); // Initial render

            window.addEventListener('resize', onWindowResize);
        }

        function onWindowResize() {
            camera.aspect = canvas.clientWidth / canvas.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(canvas.clientWidth, canvas.clientHeight);
            renderer.render(scene, camera);
        }
        
        return () => {
            if(renderer) renderer.dispose();
            renderer = null;
        }
    });
</script>

<div class="container">
    <canvas bind:this={canvas}></canvas>
    <div class="level-display" bind:this={levelTextElement}></div>
</div>

<style>
    .container {
        width: 100%;
        height: 100%;
        position: relative;
    }
    canvas {
        display: block;
        width: 100%;
        height: 100%;
    }
    .level-display {
        position: absolute;
        bottom: 2rem;
        left: 50%;
        transform: translateX(-50%);
        color: #1e293b;
        font-size: 2.25rem;
        font-weight: 700;
        text-shadow: 0px 1px 3px rgba(255, 255, 255, 0.7);
        pointer-events: none;
    }
</style>
