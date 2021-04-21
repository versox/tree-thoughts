<svelte:head>
	<title>geþencan</title>
	<style>
		body {
			margin: 0;
			width: 100%;
			height: 100%;
		}
	</style>
</svelte:head>

<script>
	import { onMount } from 'svelte';
	let canvas;

	onMount(async () => {
		const { Renderer, Camera, Program, Transform } = await import('$lib/ogl/index.mjs');
		const { Box, Mesh } = await import('ogl');
		const renderer = new Renderer({
			canvas: canvas,
			width: 1920,
			height: 1080
		});
		const camera = new Camera(renderer.gl, {
			near: 0.1,
			far: 100,
			fov: 45,
			aspect: canvas.width / canvas.height
		});
		camera.position.z = 5;

		const scene = new Transform();
		const box = new Box(renderer.gl);
		const program = new Program(renderer.gl, {
			vertex: `
            attribute vec3 position;

            uniform mat4 modelViewMatrix;
            uniform mat4 projectionMatrix;

            void main() {
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
            `,
			fragment: `
            void main() {
                gl_FragColor = vec4(0.92, 0.48, 0.84, 1.0); // Pink!
            }
        `
		});
		const mesh = new Mesh(renderer.gl, { geometry: box, program });
		scene.addChild(mesh);
		requestAnimationFrame(update);
		function update(t) {
			requestAnimationFrame(update);

			mesh.rotation.y -= 0.04;
			mesh.rotation.x += 0.03;
			renderer.render({ scene, camera });
		}
	});
</script>

<canvas bind:this={canvas} />

<style>
	canvas {
		width: 100%;
		height: auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4rem;
		font-weight: 100;
		line-height: 1.1;
		margin: 4rem auto;
		max-width: 14rem;
	}
</style>
