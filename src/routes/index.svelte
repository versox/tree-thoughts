<script lang="ts">
	import type { Mesh } from '$lib/ogl/core/Mesh';
	import type { Transform } from '$lib/ogl/core/Transform';

	import { onMount } from 'svelte';
	let canvas;

	onMount(async () => {
		const { Camera, Program } = await import('$lib/ogl/index.mjs');
		const { Transform } = await import('$lib/ogl/core/Transform');
		const { Renderer } = await import('$lib/ogl/core/Renderer');
		const { GLTFLoader } = await import('$lib/ogl/extras/GLTFLoader');
		const { TextureLoader } = await import('$lib/ogl/extras/TextureLoader');
		const { Vec3 } = await import('$lib/ogl/math/Vec3');
		const { Box, Mesh } = await import('ogl');
		const { makeLandscape } = await import('$lib/shaders/Landscape');
		const { Orbit } = await import('$lib/ogl/extras/Orbit');
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
		camera.position.set(30, 15, -30);
		const controls = new Orbit(camera);
		// Common textures for uber shader
		const lutTexture = TextureLoader.load(renderer.gl, {
			src: 'assets/pbr/lut.png'
		});
		const envDiffuseTexture = TextureLoader.load(renderer.gl, {
			src: 'assets/sunset-diffuse-RGBM.png'
		});
		const envSpecularTexture = TextureLoader.load(renderer.gl, {
			src: 'assets/sunset-specular-RGBM.png'
		});
		const scene = new Transform();
		const gltf = await GLTFLoader.load(renderer.gl, 'assets/models/mountain_cam.glb');
		console.log(gltf);
		gltf.scene.forEach((root: Transform) => {
			root.setParent(scene);
			root.children.forEach((node: Mesh) => {
				const uniforms = {
					uBaseColorFactor: { value: gltf.baseColorFactor || [1, 1, 1, 1] },
					tBaseColor: { value: gltf.baseColorTexture ? gltf.baseColorTexture.texture : null },

					tRM: {
						value: gltf.metallicRoughnessTexture ? gltf.metallicRoughnessTexture.texture : null
					},
					uRoughness: { value: gltf.roughnessFactor !== undefined ? gltf.roughnessFactor : 1 },
					uMetallic: { value: gltf.metallicFactor !== undefined ? gltf.metallicFactor : 1 },

					tNormal: { value: gltf.normalTexture ? gltf.normalTexture.texture : null },
					uNormalScale: { value: gltf.normalTexture ? gltf.normalTexture.scale || 1 : 1 },

					tOcclusion: { value: gltf.occlusionTexture ? gltf.occlusionTexture.texture : null },

					tEmissive: { value: gltf.emissiveTexture ? gltf.emissiveTexture.texture : null },
					uEmissive: { value: gltf.emissiveFactor || [0, 0, 0] },

					tLUT: { value: lutTexture },
					tEnvDiffuse: { value: envDiffuseTexture },
					tEnvSpecular: { value: envSpecularTexture },
					uEnvDiffuse: { value: 0.5 },
					uEnvSpecular: { value: 0.5 },

					uLightDirection: { value: new Vec3(0, 1, 1) },
					uLightColor: { value: new Vec3(2.5) },

					uAlpha: { value: 1 },
					uAlphaCutoff: { value: gltf.alphaCutoff }
				};
				node.program = makeLandscape(renderer, uniforms);
			});
		});
		requestAnimationFrame(update);
		function update(t) {
			requestAnimationFrame(update);
			controls.update();
			renderer.render({ scene, camera });
		}
	});
</script>

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
