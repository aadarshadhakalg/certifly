<script lang="ts">
	import { onMount } from 'svelte';
	import Papa from 'papaparse';
	import { v4 } from 'uuid';
	import JSZip from 'jszip';
	import FileSaver from 'file-saver';

	let isTemplateLoaded = false;
	let generated = false;
	let origDimensions: { width: number; height: number } = { width: 0, height: 0 };
	let origImage: HTMLImageElement;
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
	let certificatesContainer: HTMLDivElement;
	let previewsContainer: HTMLDivElement;

	function render(ctx: CanvasRenderingContext2D) {
		ctx.clearRect(0, 0, templateCanvas.width, templateCanvas.height);
		ctx.drawImage(origImage, 0, 0);

		fields.forEach((field) => {
			ctx.font = `${field.fontSize}px ${field.fontFamily}`;
			ctx.fillStyle = field.color;
			ctx.fillText(field.value, field.position.x, field.position.y);
		});

		templateDisplay.src = templateCanvas.toDataURL() as string;
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

			origImage = new Image();
			origImage.src = reader.result as string;
			templateCanvas.width = origImage.width;
			templateCanvas.height = origImage.height;
			origDimensions = { width: origImage.width, height: origImage.height };
			ctx?.drawImage(origImage, 0, 0);

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

				if (fields[index].value === target.value) {
					return alert('Please enter a different value');
				}

				fields[index].value = target.value;

				render(templateCanvas.getContext('2d') as CanvasRenderingContext2D);
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

	function generateCertificates() {
		if (!isTemplateLoaded) {
			return alert('Please upload template');
		}

		if (csvData.length === 0) {
			return alert('Please upload a non-empty csv file');
		}

		previewsContainer.innerHTML = '';

		csvData.forEach((row) => {
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;
			canvas.width = origDimensions.width;
			canvas.height = origDimensions.height;

			ctx.drawImage(origImage, 0, 0);

			fields.forEach((field) => {
				ctx.font = `${field.fontSize}px ${field.fontFamily}`;
				ctx.fillStyle = field.color;
				ctx.fillText(row[field.value], field.position.x, field.position.y);
			});

			certificatesContainer.append(canvas);

			const preview = document.createElement('img');
			preview.src = canvas.toDataURL() as string;
			preview.classList.add('preview');
			previewsContainer.append(preview);
		});

		generated = true;
	}

	function download() {
		const certificates = certificatesContainer.querySelectorAll('canvas');
		const zip = new JSZip();

		certificates.forEach((certificate, index) => {
			const img = document.createElement('img');
			img.src = certificate.toDataURL('image/png');

			zip
				.folder('certificates')
				?.file(`certificate-${index + 1}.png`, img.src.split(',')[1], { base64: true });
		});

		zip.generateAsync({ type: 'blob' }).then((content) => {
			FileSaver.saveAs(content, 'certificates.zip');
		});
	}
</script>

{#if !isTemplateLoaded}
	<label for="template-img">Add template</label>
{:else}
	<label for="template-img">Change template</label>
{/if}
<input
	type="file"
	class="preview"
	name="template-img"
	accept="image/png, image/jpeg"
	on:change={handleFileInput}
/>
<img id="display-template" bind:this={templateDisplay} alt="template-display" />
<canvas bind:this={templateCanvas} class="hidden" />

<div id="fields" bind:this={fieldsContainer} />

{#if fields.length !== 0}
	<button on:click={generateCertificates}>Generate</button>
{/if}

{#if generated}
	<button on:click={download}>Download</button>
{/if}

<input id="csv-reader" type="file" bind:this={csvReader} accept=".csv" />

<div class="hidden" bind:this={certificatesContainer} />
<div bind:this={previewsContainer} />

<style>
	.preview {
		width: 100%;
		height: 100%;
		object-fit: contain;
		max-width: 900px;
		display: block;
	}
</style>
