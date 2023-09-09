<script lang="ts">
	import { onMount } from 'svelte';
	import Papa from 'papaparse';
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

		// fields.push(;
		fields = [
			...fields,
			{
				id: `field-${fields.length + 1}`,
				value: '',
				position: { x: offsetXCanvas, y: offsetYCanvas },
				fontSize: 14,
				fontFamily: 'Arial',
				color: 'black'
			}
		];
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
				if (row[field.value]) {
					ctx.font = `${field.fontSize}px ${field.fontFamily}`;
					ctx.fillStyle = field.color;
					ctx.fillText(row[field.value], field.position.x, field.position.y);
				}
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

	const updateField = (e: Event) => {
		e.preventDefault();
		const target = e.target as HTMLFormElement;

		const htmlFields = Object.entries(target.elements).map((item) => item[1]) as (
			| HTMLInputElement
			| HTMLSelectElement
		)[];

		const sd = fields.find(
			(item) =>
				item.id ===
				(htmlFields.find((item) => item.name === 'field-details') as HTMLSelectElement).value
		);

		console.log(sd);
	};
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

{#if fields.length !== 0}
	<h2 class="text-xl">Change field details</h2>
	<form on:submit={updateField}>
		<select name="field-details">
			{#each fields as field}
				<option value={field.id}>
					{#if field.value === ''}
						{field.id}
					{:else}
						{field.value}
					{/if}
				</option>
			{/each}
		</select>
		<div>
			<label for="font-size">Font size</label>
			<input type="number" name="font-size" min="1" max="100" />
		</div>
		<div>
			<label for="value">Value</label>
			<input type="text" name="value" />
		</div>

		<div>
			<label for="color">Color</label>
			<input type="color" name="color" />
		</div>

		<button>Update</button>
	</form>

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

	input,
	select {
		background: transparent;
	}
</style>
