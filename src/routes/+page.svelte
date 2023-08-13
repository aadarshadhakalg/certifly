<script lang="ts">
	import { onMount } from 'svelte';
	import Template from '../components/template.svelte';
	import { v4 } from 'uuid';

	let isTemplateLoaded = false;
	let isCsvLoaded = false;
	let templatePath = '';
	let width = 0;
	let textboxes: {
		id: string;
		x: number;
		y: number;
		name: string;
	}[] = [];
	let csv: string[][] = [];
	let imgOrigSize = { width: 0, height: 0 };
	let imgDisplaySize = { width: 0, height: 0 };

	onMount(() => {
		// set width of image
		width = 0.9 * window.innerWidth;
		window.addEventListener('resize', () => {
			width = 0.9 * window.innerWidth;
		});

		const canvas = document.getElementById('canvas') as HTMLCanvasElement;
		const img = document.getElementById('template-image') as HTMLImageElement;
		const fieldsContainer = document.getElementById('form-container') as HTMLDivElement;
		const csvInput = document.getElementById('csv-input') as HTMLInputElement;

		const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

		img.onload = () => {
			// aspect ratio of image
			const aspectRatio = img.width / img.height;
			imgOrigSize = { width: img.width, height: img.height };

			// image new size
			img.width = width;
			img.height = img.width / aspectRatio;

			imgDisplaySize = { width: img.width, height: img.height };

			// set canvas size
			canvas.width = img.width;
			canvas.height = img.height;

			// draw template on canvas
			ctx.drawImage(img, 0, 0, img.width, img.height);

			window.addEventListener('click', addPlaces);

			csvInput.addEventListener('change', onCsvLoad);
		};

		function isMouseOverCanvas(cursorX: number, cursorY: number): boolean {
			const { x, y } = canvas.getBoundingClientRect();
			return (
				cursorX >= x &&
				cursorX <= x + imgDisplaySize.width &&
				cursorY >= y &&
				cursorY <= y + imgDisplaySize.height
			);
		}

		function addPlaces(event: MouseEvent) {
			const { x, y } = canvas.getBoundingClientRect();

			if (isMouseOverCanvas(event.clientX, event.clientY)) {
				textboxes.push({ id: v4(), x: event.clientX - x, y: event.clientY - y, name: '' });
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
		}

		function onCsvLoad() {
			const csvReader = new FileReader();
			csvReader.readAsText(csvInput.files?.[0] as Blob);
			csvReader.onload = (e: ProgressEvent<FileReader>) => {
				const csvString = e.target?.result as string;
				csv = csvString.split('\n').map((row) => row.split(','));
				isCsvLoaded = true;
			};
		}
	});

	function handleFieldsSubmit() {
		if (!isCsvLoaded) {
			return alert('Please upload csv file');
		}

		if (!isTemplateLoaded) {
			return alert('Please upload template');
		}

		if (csv.length === 0) {
			return alert('Please upload a non-empty csv file');
		}

		const previewsContainer = document.getElementById('previews-container') as HTMLDivElement;

		const csvFields = csv[0];
		let data: { field: string; x: number; y: number; data: string[] }[] = [];

		csvFields.forEach((csvField: string) => {
			const index = textboxes.findIndex((textbox) => textbox.name.trim() === csvField.trim());

			if (index === -1) {
				return;
			}

			data.push({
				field: csvField,
				x: textboxes[index].x,
				y: textboxes[index].y,
				data: []
			});
		});

		csv.forEach((row, rowIndex) => {
			if (rowIndex === 0) {
				return;
			}

			data.forEach((field) => {
				field.data.push(row[csvFields.indexOf(field.field)]);
			});
		});

		const dataFirst = data[0];

		if (!dataFirst) {
			return alert('No matching field found');
		}

		previewsContainer.textContent = '';
		for (let i = 0; i < dataFirst.data.length; i++) {
			const canvas = document.createElement('canvas') as HTMLCanvasElement;
			const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;
			const img = document.getElementById('template-image') as HTMLImageElement;

			canvas.width = img.width;
			canvas.height = img.height;

			ctx.drawImage(img, 0, 0, img.width, img.height);
			ctx.font = '16px Arial';
			// ctx.fillText('hello world', 5, 10);

			data.forEach((element) => {
				ctx.fillText(element.data[i], element.x, element.y);
			});

			previewsContainer.append(canvas);
		}
	}
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
<form id="form-container" on:submit={handleFieldsSubmit}>
	<button type="submit">Generate</button>
</form>

<p>Drop the CSV here.</p>
<p>Warning: Don't upload csv file before uploading the template</p>
<input type="file" id="csv-input" accept=".csv" />

<!-- download button -->
<button>Click to Download</button>

<!-- previews container -->
<div id="previews-container" />

<style>
	canvas {
		outline: 1px solid black;
	}
</style>
