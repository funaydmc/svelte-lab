<script lang="ts">
    import { onMount } from 'svelte';
    import { BoxGeometry, Color, Mesh, MeshBasicMaterial, PerspectiveCamera, Scene, WebGLRenderer, BufferGeometry, LineBasicMaterial, Line, Vector3, ConeGeometry, SphereGeometry }  from 'three';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
    


    let forward: number = $state(0);
    let upward: number = $state(0);
    let right: number = $state(0);
    let steps: number = $state(1);
    let angle: number = $state(0);
    let startAngle: number = $state(0);
    let hTranslation: number = $state(0);
    let vTranslation: number = $state(0);
    let hCycles: number = $state(1);
    let vCycles: number = $state(1);
    let radius: number = $state(0.05);
    let duration: number = $state(1);
    let frequency: number = $state(0.05);

    let points: Vector3[] = $state([
        new Vector3(0, 0, 0)
    ]);

    let container: HTMLDivElement;
    let canvas: HTMLCanvasElement;
    let renderer: WebGLRenderer;
    let scene: Scene;
    let camera: PerspectiveCamera;
    let controls: OrbitControls;
    let spheres: Mesh[] = [];

    function initScene() {
        scene = new Scene();
        renderer = new WebGLRenderer({antialias: true, canvas});
        scene.background = new Color('#303030');
        camera = new PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 10);
        camera.position.set(3, 3, 3);
        camera.lookAt(0, 0, 0);
        
        // Create axes
        createAxes();
        
        // Create initial spheres
        updateSpheres();
        
        controls = new OrbitControls(camera, canvas);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        
        resizeCanvas();
        animate();
    }

    function createAxes() {
        const axisLength = 2;
        const arrowSize = 0.05;
        
        // X axis positive (Red, solid)
        const xPosGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(axisLength, 0, 0)
        ]);
        const xPosMaterial = new LineBasicMaterial({ color: 0xff0000 });
        const xPosAxis = new Line(xPosGeometry, xPosMaterial);
        scene.add(xPosAxis);
        
        // X axis arrowhead
        const xArrowGeometry = new ConeGeometry(arrowSize, arrowSize * 2, 8);
        const xArrowMaterial = new MeshBasicMaterial({ color: 0xff0000 });
        const xArrow = new Mesh(xArrowGeometry, xArrowMaterial);
        xArrow.position.set(axisLength, 0, 0);
        xArrow.rotation.z = -Math.PI / 2;
        scene.add(xArrow);
        
        // X axis negative (Red, dashed)
        const xNegGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(-axisLength, 0, 0)
        ]);
        const xNegMaterial = new LineBasicMaterial({ 
            color: 0xff0000,
            transparent: true,
            opacity: 0.6
        });
        const xNegAxis = new Line(xNegGeometry, xNegMaterial);
        xNegAxis.computeLineDistances();
        scene.add(xNegAxis);
        
        // Y axis positive (Green, solid) - Vertical
        const yPosGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(0, axisLength, 0)
        ]);
        const yPosMaterial = new LineBasicMaterial({ color: 0x00ff00 });
        const yPosAxis = new Line(yPosGeometry, yPosMaterial);
        scene.add(yPosAxis);
        
        // Y axis arrowhead
        const yArrowGeometry = new ConeGeometry(arrowSize, arrowSize * 2, 8);
        const yArrowMaterial = new MeshBasicMaterial({ color: 0x00ff00 });
        const yArrow = new Mesh(yArrowGeometry, yArrowMaterial);
        yArrow.position.set(0, axisLength, 0);
        scene.add(yArrow);
        
        // Y axis negative (Green, dashed) - Vertical down
        const yNegGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(0, -axisLength, 0)
        ]);
        const yNegMaterial = new LineBasicMaterial({ 
            color: 0x00ff00,
            transparent: true,
            opacity: 0.6
        });
        const yNegAxis = new Line(yNegGeometry, yNegMaterial);
        scene.add(yNegAxis);
        
        // Z axis positive (Blue, solid)
        const zPosGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(0, 0, axisLength)
        ]);
        const zPosMaterial = new LineBasicMaterial({ color: 0x0000ff });
        const zPosAxis = new Line(zPosGeometry, zPosMaterial);
        scene.add(zPosAxis);
        
        // Z axis arrowhead
        const zArrowGeometry = new ConeGeometry(arrowSize, arrowSize * 2, 8);
        const zArrowMaterial = new MeshBasicMaterial({ color: 0x0000ff });
        const zArrow = new Mesh(zArrowGeometry, zArrowMaterial);
        zArrow.position.set(0, 0, axisLength);
        zArrow.rotation.x = Math.PI / 2;
        scene.add(zArrow);
        
        // Z axis negative (Blue, dashed)
        const zNegGeometry = new BufferGeometry().setFromPoints([
            new Vector3(0, 0, 0),
            new Vector3(0, 0, -axisLength)
        ]);
        const zNegMaterial = new LineBasicMaterial({ 
            color: 0x0000ff,
            transparent: true,
            opacity: 0.6
        });
        const zNegAxis = new Line(zNegGeometry, zNegMaterial);
        scene.add(zNegAxis);
    }

    function updateSpheres() {
        spheres.forEach(sphere => {
            scene.remove(sphere);
        });
        spheres = [];
        if (radius > 0) {
            points.forEach(point => {
                const sphereGeometry = new SphereGeometry(radius, 16, 12);
                const sphereMaterial = new MeshBasicMaterial({ 
                    color: 0xffffff,
                    transparent: true,
                    opacity: 0.8
                });
                const sphere = new Mesh(sphereGeometry, sphereMaterial);
                sphere.position.copy(point);
                scene.add(sphere);
                spheres.push(sphere);
            });
        }
    }

    function updatePoints() {
        const newPoints: Vector3[] = [];

        const degToRad = Math.PI / 180;
        const durationSteps = steps * Math.floor(20 * duration);
        const hCycleLength = durationSteps / hCycles;
        const vCycleLength = durationSteps / vCycles;
        const rotationPerStep = (angle * degToRad) / durationSteps;
        let dirX = 0;
        let dirZ = 1;
        
        let offsetX = right;
        let offsetY = upward;
        let offsetZ = forward;
        
        const startAngleRad = startAngle * degToRad;
        const startCos = Math.cos(startAngleRad);
        const startSin = Math.sin(startAngleRad);
        const tempOffsetX = offsetX * startCos - offsetZ * startSin;
        offsetZ = offsetX * startSin + offsetZ * startCos;
        offsetX = tempOffsetX;
        const tempDirX = dirX * startCos - dirZ * startSin;
        dirZ = dirX * startSin + dirZ * startCos;
        dirX = tempDirX;
        for (let step = 0; step < Math.min(durationSteps, steps * 50); step++) {
            const heightOffset = vTranslation * (vCycleLength - Math.abs(vCycleLength - (step % (2 * vCycleLength)))) / vCycleLength;
            const radialOffset = hTranslation * (hCycleLength - Math.abs(hCycleLength - (step % (2 * hCycleLength)))) / hCycleLength;
            const currentY = upward + heightOffset;
            const point = new Vector3(offsetX, currentY, offsetZ);
            newPoints.push(point);
            if (step < durationSteps - 1) {
                const cos = Math.cos(rotationPerStep);
                const sin = Math.sin(rotationPerStep);
                const newOffsetX = offsetX * cos - offsetZ * sin;
                offsetZ = offsetX * sin + offsetZ * cos;
                offsetX = newOffsetX;
                const newDirX = dirX * cos - dirZ * sin;
                dirZ = dirX * sin + dirZ * cos;
                dirX = newDirX;
                const nextRadialOffset = hTranslation * (hCycleLength - Math.abs(hCycleLength - ((step + 1) % (2 * hCycleLength)))) / hCycleLength;
                const radialDelta = nextRadialOffset - radialOffset;
                offsetX += radialDelta * dirX;
                offsetZ += radialDelta * dirZ;
            }
        }

        points = newPoints;
    }

    function resizeCanvas() {
        if (!container || !renderer || !camera) return;
        
        const width = container.clientWidth;
        const height = container.clientHeight;
        
        renderer.setSize(width, height, false);
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
    }

    function animate() {
        requestAnimationFrame(animate);
        controls?.update();
        renderer?.render(scene, camera);
    }

    $effect(() => {
        initScene();
        
        // Handle window resize
        const resizeObserver = new ResizeObserver(() => {
            resizeCanvas();
        });
        
        resizeObserver.observe(container);
        
        return () => {
            resizeObserver.disconnect();
        };
    });

    // Update spheres when points or radius changes
    $effect(() => {
        if (scene) {
            updateSpheres();
        }
    });

    // Update points when control parameters change (excluding points and radius)
    $effect(() => {
        updatePoints();
    });

</script>
<section>
    <div style="display: flex; align-items: center; justify-content: center; ">
        <table>
            <tbody>

            <tr>
                <td>Forward Offset</td>
                <td><input bind:value={forward} type="number"></td>
                <td><input bind:value={forward} type="range" min="-10" max="10" step="0.01" /></td>
            </tr>
            <tr>
                <td>Upward Offset</td>
                <td><input bind:value={upward} type="number"></td>
                <td><input bind:value={upward} type="range" min="-10" max="10" step="0.01" /></td>
            </tr>
            <tr>
                <td>Right Offset</td>
                <td><input bind:value={right} type="number"></td>
                <td><input bind:value={right} type="range" min="-10" max="10" step="0.01" /></td>
            </tr>
            <tr>
                <td>Steps</td>
                <td><input bind:value={steps} type="number"></td>
                <td><input bind:value={steps} type="range" min="1" max="10" step="1"/></td>
            </tr>
            <tr>
                <td>Angle</td>
                <td><input bind:value={angle} type="number"></td>
                <td><input bind:value={angle} type="range" min="-3600" max="3600" step="0.1" /></td>
            </tr>
            <tr>
                <td>Start Angle</td>
                <td><input bind:value={startAngle} type="number"></td>
                <td><input bind:value={startAngle} type="range" min="-360" max="360" step="0.1" /></td>
            </tr>
            <tr>
                <td>H-Translation</td>
                <td><input bind:value={hTranslation} type="number"></td>
                <td><input bind:value={hTranslation} type="range" min="-10" max="10" step="0.01"/></td>
            </tr>
            <tr>
                <td>V-Translation</td>
                <td><input bind:value={vTranslation} type="number"></td>
                <td><input bind:value={vTranslation} type="range" min="-10" max="10" step="0.01"/></td>
            </tr>
            <tr>
                <td>H-Cycles</td>
                <td><input bind:value={hCycles} type="number"></td>
                <td><input bind:value={hCycles} type="range" min="1" max="20" step="1"/></td>
            </tr>
            <tr>
                <td>V-Cycles</td>
                <td><input bind:value={vCycles} type="number"></td>
                <td><input bind:value={vCycles} type="range" min="1" max="20" step="1"/></td>
            </tr>
            <tr>
                <td>Radius</td>
                <td><input bind:value={radius} type="number"></td>
                <td><input bind:value={radius} type="range" min="0" max="1" step="0.01"/></td>
            </tr>
            <tr>
                <td>Duration</td>
                <td><input bind:value={duration} type="number"></td>
                <td><input bind:value={duration} type="range" min="0.05" max="100" step="0.05"/></td>
            </tr>
            <tr>
                <td>Frequency</td>
                <td><input bind:value={frequency} type="number"></td>
                <td><input bind:value={frequency} type="range" min="0.05" max="30" step="0.05"/></td>
            </tr>
            </tbody>
        </table>
    </div>
    <div bind:this={container}>
        <canvas bind:this={canvas}></canvas>
    </div>
</section>

<style>
    :global(body) {
        margin: 0;
        padding: 0;
        overflow: hidden;
        color: #fff;
    }
    section {
        display: flex;
        width: 100%;
        height: 100vh;
    }
    div {
        width: 50%;
        height: calc(100vh - 2px);
        border: 1px solid #525252;
        position: relative;
    }
    canvas {
        display: block;
        width: 100%;
        height: 100%;
    }
    input[type="range"] {
        width: 30em;
    }
    input[type="number"] {
        width: 4em;
    }
</style>