<script>
	import * as GL from '@sveltejs/gl';
	import { onMount } from 'svelte';

	const counts = [20, 100, 200, 500, 1000, 2000, 10000];
	let count = counts[0];

	const random = () => Math.random() * 360;

	$: boxes = Array(count).fill().map(() => [random(), random(), random()]);
	$: fps = frames.reduce((total, frame) => total + frame) / frames.length;

	let frames = Array(30).fill(0); // for smoothing out FPS counter

	onMount(() => {
		let last = Date.now();

		let frame = requestAnimationFrame(function loop() {
			frame = requestAnimationFrame(loop);

			boxes.forEach(box => {
				box[0] += 0.5;
				box[1] += 0.5;
				box[2] += 0.5;
			});

			boxes = boxes; // tell Svelte the array has changed

			const now = Date.now();
			const elapsed = now - last;
			frames.shift();
			frames[frames.length] = 1000 / elapsed;

			last = now;
		});

		return () => {
			cancelAnimationFrame(frame);
		};
	});
</script>

<main>
	<GL.Scene>
		<GL.PerspectiveCamera location={[0,0,3.2]}/>
		<GL.DirectionalLight direction={[-5,0,-10]}/>

		{#each boxes as r}
			<GL.Mesh
				frag={`
					varying vec3 v_normal;

					void main() {
						gl_FragColor = vec4(mix(vec3(0.8), v_normal, 0.5), 1.0);
					}
				`}
				geometry={GL.box()}
				rotation={r}
			/>
		{/each}
	</GL.Scene>

	<div class="controls">
		<label>
			Amount
			<select bind:value={count}>
				{#each counts as count}<option>{count}</option>{/each}
			</select>

			<!-- {count}
			<input type=range bind:value={count} min={20} max={10000} step={1}> -->
		</label>
	</div>

	<div class="info">
		FPS {Math.round(fps)}
	</div>
</main>

<style>
	main {
		width: 100%;
		height: 100%;
	}

	.controls {
		position: fixed;
		top: 1em;
		right: 1em;
	}

	.info {
		position: fixed;
		top: 1em;
		left: 1em;
	}
</style>