<script lang="ts">
	import { onMount } from 'svelte';
	import Papa from 'papaparse';
	import { v4 } from 'uuid';

	let isTemplateLoaded = false;
	let origDimensions: { width: number; height: number } = { width: 0, height: 0 };
	let csvData: any[] = [];
	let fields: {
		id: string;
		value: string;
		position: { x: number; y: number };
		fontSize: number;
		fontFamily: string;
		color: string;
	}[] = [];

	let templateCanvas: HTMLCanvasElement;
	let templateDisplay: HTMLImageElement;
	let csvReader: HTMLInputElement;
	let fieldsContainer: HTMLDivElement;

	function render(ctx: CanvasRenderingContext2D) {
		ctx.font = '16px Arial';
		ctx.fillStyle = 'black';
		ctx.drawImage(templateCanvas, 0, 0);
	}

	const handleFileInput = (e: Event) => {
		const target = e.target as HTMLInputElement;
		if (!target.files || !target.files[0]) {
			return;
		}
		const file = target.files[0];
		const reader = new FileReader();
		reader.readAsDataURL(file);
		reader.onloadend = () => {
			isTemplateLoaded = true;

			const ctx = templateCanvas.getContext('2d');
			const img = new Image();
			img.src = reader.result as string;
			templateCanvas.width = img.width;
			templateCanvas.height = img.height;
			origDimensions = { width: img.width, height: img.height };
			ctx?.drawImage(img, 0, 0);

			// display template
			templateDisplay.src = templateCanvas.toDataURL() as string;
			templateDisplay.addEventListener('click', templateDisplayClickHandler);
		};
	};

	const templateDisplayClickHandler = (event: MouseEvent) => {
		const boundingRect = templateDisplay.getBoundingClientRect();
		const offsetX = event.clientX - boundingRect.left;
		const offsetY = event.clientY - boundingRect.top;

		const offsetXRelative = offsetX / boundingRect.width;
		const offsetYRelative = offsetY / boundingRect.height;

		const offsetXCanvas = offsetXRelative * origDimensions.width;
		const offsetYCanvas = offsetYRelative * origDimensions.height;

		const ctx = templateCanvas.getContext('2d');
		if (!ctx) {
			return;
		}
		ctx.font = '14px Arial';
		ctx.fillText('Hello World', offsetXCanvas, offsetYCanvas);
		templateDisplay.src = templateCanvas.toDataURL() as string;

		fields.push({
			id: v4(),
			value: '',
			position: { x: offsetXCanvas, y: offsetYCanvas },
			fontSize: 14,
			fontFamily: 'Arial',
			color: 'black'
		});

		fieldsContainer.innerHTML = '';

		fields.forEach((field) => {
			const textbox = document.createElement('input');
			textbox.type = 'text';
			['bg-transparent', 'border', 'border-white'].forEach((className) =>
				textbox.classList.add(className)
			);
			textbox.id = field.id;
			textbox.addEventListener('change', (e) => {
				const target = e.target as HTMLInputElement;
				const index = fields.findIndex((field) => field.id === target.id);
				console.log(target.value);
				fields[index].value = target.value;
				console.log(fields);
			});
			fieldsContainer.append(textbox);
		});
	};

	onMount(() => {
		const csvReaderInput = document.getElementById('csv-reader') as HTMLInputElement;

		csvReaderInput.onchange = () => {
			const file = csvReaderInput.files?.[0];
			if (!file) {
				return;
			}
			const reader = new FileReader();
			reader.readAsText(file);
			reader.onloadend = () => {
				const csv = reader.result as string;

				Papa.parse(csv, {
					header: true, // Set this to true if your CSV string has headers
					skipEmptyLines: true,
					complete: (result: any) => {
						csvData.push(...result.data);
					},
					error: (error: Error) => {
						console.error('Error parsing CSV:', error);
					}
				});
			};
		};
	});
</script>

{#if !isTemplateLoaded}
	<label for="template-img">Add template</label>
{:else}
	<label for="template-img">Change template</label>
{/if}
<input
	type="file"
	id="template-img"
	name="template-img"
	accept="image/png, image/jpeg"
	on:change={handleFileInput}
/>
<img id="display-template" bind:this={templateDisplay} alt="template-display" />
<canvas bind:this={templateCanvas} class="hidden" />

<div id="fields" bind:this={fieldsContainer} />

<input id="csv-reader" type="file" bind:this={csvReader} accept=".csv" />

<style>
	#display-template {
		width: 100%;
		height: 100%;
		object-fit: contain;
		max-width: 900px;
		display: block;
	}
</style>
