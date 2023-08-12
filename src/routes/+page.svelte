<script lang="ts">
	import { onMount } from 'svelte';
	import Template from '../components/template.svelte';
	import { v4 } from 'uuid';

	let isTemplateLoaded = false;
	let templatePath = '';
	let width = 0;
	let textboxes: {
		id: string;
		x: number;
		y: number;
		name: string;
	}[] = [];

	onMount(() => {
		// set width of image
		width = 0.9 * window.innerWidth;
		window.addEventListener('resize', () => {
			width = 0.9 * window.innerWidth;
		});

		const canvas = document.getElementById('canvas') as HTMLCanvasElement;
		const img = document.getElementById('template-image') as HTMLImageElement;
		const fieldsContainer = document.getElementById('form-container') as HTMLDivElement;

		window.addEventListener('click', (event) => {
			const x = event.clientX;
			const y = event.clientY;

			if (isMouseOverCanvas(x, y)) {
				textboxes.push({ id: v4(), x, y, name: '' });
				const textbox = document.createElement('input');
				textbox.type = 'text';
				textbox.id = textboxes[textboxes.length - 1].id;
				fieldsContainer.append(textbox);

				const fields = document.querySelectorAll('#form-container>input');
				fields.forEach((field) => {
					field.addEventListener('change', (event) => {
						const id = (event.target as HTMLInputElement).id;
						const value = (event.target as HTMLInputElement).value;
						const index = textboxes.findIndex((textbox) => textbox.id === id);
						textboxes[index].name = value;
					});
				});
			}
		});

		const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

		img.onload = () => {
			// img.width = width;
			canvas.width = img.width;
			canvas.height = img.height;

			ctx.drawImage(img, 0, 0, img.width, img.height);
		};

		function isMouseOverCanvas(cursorX: number, cursorY: number) {
			const { x, y, height, width } = canvas.getBoundingClientRect();
			return cursorX >= x && cursorX <= x + width && cursorY >= y && cursorY <= y + height;
		}
	});
</script>

<Template
	on:templatestatus={(event) => {
		isTemplateLoaded = event.detail.loaded;
		templatePath = event.detail.imagePath;
	}}
/>

{#if isTemplateLoaded}
	<p>Template loaded!</p>
{:else}
	<p>Add template</p>
{/if}

<img style="display: none;" src={templatePath} alt="template" id="template-image" />
<canvas id="canvas" />
<div id="form-container" />

<style>
	canvas {
		outline: 1px solid black;
	}
</style>
