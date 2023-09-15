<script lang="ts">
	import { onMount } from 'svelte';
	import Papa from 'papaparse';
	import JSZip from 'jszip';
	import FileSaver from 'file-saver';
	import { v4 } from 'uuid';

	let isTemplateLoaded = false;
	let generated = false;
	let origDimensions: { width: number; height: number } = { width: 0, height: 0 };
	let origImage: HTMLImageElement;
	let csvData: any[] = [];
	let fields: {
		id: string;
		value: string;
		demo: string;
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
			ctx.fillText(field.demo, field.position.x, field.position.y);
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

		fields = [
			...fields,
			{
				id: v4(),
				value: `field-${fields.length + 1}`,
				demo: 'hello world',
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

	const setFontSize = (e: Event) => {
		const target = e.target as HTMLInputElement;
		const selectedField = fields.find(
			(item) =>
				item.id ===
				(document.querySelector('select[name="field-details"]') as HTMLSelectElement).value
		);

		if (!selectedField) {
			return;
		}

		selectedField.fontSize = parseInt(target.value);
		render(templateCanvas.getContext('2d') as CanvasRenderingContext2D);
	};

	const handleSelectChange = (e: Event) => {
		const target = e.target as HTMLSelectElement;
		const selectedField = fields.find((item) => item.id === target.value);

		if (!selectedField) {
			return;
		}

		const fontSizeInput = document.querySelector('input[name="font-size"]') as HTMLInputElement;
		const valueInput = document.querySelector('input[name="value"]') as HTMLInputElement;
		const demoTextInput = document.querySelector('input[name="demo-text"]') as HTMLInputElement;
		const colorInput = document.querySelector('input[name="color"]') as HTMLInputElement;

		fontSizeInput.value = selectedField.fontSize.toString();
		valueInput.value = selectedField.value;
		demoTextInput.value = selectedField.value;
		colorInput.value = selectedField.color;
	};

	const setValue = (e: Event) => {
		const target = e.target as HTMLInputElement;
		const selectedField = fields.find(
			(item) =>
				item.id ===
				(document.querySelector('select[name="field-details"]') as HTMLSelectElement).value
		);

		if (!selectedField) {
			return;
		}

		selectedField.value = target.value;
		render(templateCanvas.getContext('2d') as CanvasRenderingContext2D);
	};

	const setDemoText = (e: Event) => {
		const target = e.target as HTMLInputElement;
		const selectedField = fields.find(
			(item) =>
				item.id ===
				(document.querySelector('select[name="field-details"]') as HTMLSelectElement).value
		);

		if (!selectedField) {
			return;
		}

		selectedField.demo = target.value;
		render(templateCanvas.getContext('2d') as CanvasRenderingContext2D);
	};

	const setColor = (e: Event) => {
		const target = e.target as HTMLInputElement;
		const selectedField = fields.find(
			(item) =>
				item.id ===
				(document.querySelector('select[name="field-details"]') as HTMLSelectElement).value
		);

		if (!selectedField) {
			return;
		}

		selectedField.color = target.value;
		render(templateCanvas.getContext('2d') as CanvasRenderingContext2D);
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

	<select on:change={handleSelectChange} name="field-details">
		{#each fields as field}
			<option value={field.id}>
				{field.value}
			</option>
		{/each}
	</select>

	<div>
		<label for="font-size">Font size</label>
		<input
			type="number"
			name="font-size"
			min="1"
			max="100"
			on:change={setFontSize}
			value={fields[0].fontSize}
		/>
	</div>

	<div>
		<label for="value">Value</label>
		<input type="text" name="value" on:change={setValue} />
	</div>

	<div>
		<label for="demo-text">Demo Text</label>
		<input type="text" name="demo-text" on:change={setDemoText} />
	</div>

	<div>
		<label for="color">Color</label>
		<input type="color" on:change={setColor} name="color" />
	</div>

	<button>Update</button>

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
