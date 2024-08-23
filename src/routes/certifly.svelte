<script anyscript lang="ts">
	import { onMount } from 'svelte';
	import Papa from 'papaparse';
	import JSZip from 'jszip';
	import FileSaver from 'file-saver';
	import { v4 } from 'uuid';

	let isTemplateLoaded = false;
	let isCSVfileLoaded = false;
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
		try {
			ctx.clearRect(0, 0, templateCanvas.width, templateCanvas.height);
			ctx.drawImage(origImage, 0, 0);

			fields.forEach((field) => {
				ctx.font = `${field.fontSize}px ${field.fontFamily}`;
				ctx.fillStyle = field.color;
				ctx.fillText(field.demo, field.position.x, field.position.y);
			});

			templateDisplay.src = templateCanvas.toDataURL() as string;
			console.log('template display image render');
		} catch (err: any) {
			console.log('err,', err);
		}
	}

	const handleFileInput = (e: Event) => {
		isTemplateLoaded = true;
		const target = e.target as HTMLInputElement;
		if (!target.files || !target.files[0]) {
			return; //function exit
		}
		const file = target.files[0];

		try {
			const reader = new FileReader();

			// onloadend is fired when a file read has completed
			reader.onloadend = () => {
				if (!templateCanvas) {
					console.log('no template canvas');
					return;
				}
				const ctx = templateCanvas.getContext('2d');

				//newly created image object is assigned
				origImage = new Image();

				//event is fired when a file has been read successfully.
				origImage.onload = () => {
					templateCanvas.width = origImage.width;
					templateCanvas.height = origImage.height;
					origDimensions = { width: origImage.width, height: origImage.height };
					ctx?.drawImage(origImage, 0, 0);

					// display template
					templateDisplay.src = templateCanvas.toDataURL() as string;

					templateDisplay.addEventListener('click', templateDisplayClickHandler);
				};
				// display original image
				origImage.src = reader.result as string;
			};
			reader.readAsDataURL(file);
		} catch (err: any) {
			console.log(err);
		}
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
		ctx.fillText('Null Value', offsetXCanvas, offsetYCanvas);
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
			isCSVfileLoaded = true;
			const file = csvReaderInput.files?.[0];
			if (!file) {
				return;
			}
			const reader = new FileReader(); // asynchronously read file data
			reader.readAsText(file);

			// onloadend is fired when a file read has completed
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
			generated = true;
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
		const FontStyleInput = document.querySelector('input[name="font-input"]') as HTMLInputElement;
		const colorInput = document.querySelector('input[name="color"]') as HTMLInputElement;

		fontSizeInput.value = selectedField.fontSize.toString();
		valueInput.value = selectedField.value;
		demoTextInput.value = selectedField.value;
		FontStyleInput.value = selectedField.fontFamily;
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

<div class="font-semibold flex flex-col justify-start m-16">
	<!-- Template display part -->
	<div class="flex flex-col justify-center">
		<div class=" mb-3 text-2xl text-center">
			Template:
			<!-- {#if !isTemplateLoaded} -->
			<!-- <label for="template-img" class="text-xl">Add template</label> -->
			<!-- {:else} -->
			<!-- <label for="template-img " class="text-xl">Change template</label>
		{/if} -->
		</div>

		<div class="mt-1">
			{#if isTemplateLoaded}
				<img
					class="border preview border-red-600"
					id="display-template"
					bind:this={templateDisplay}
					alt="template-display"
				/>

				<canvas bind:this={templateCanvas} class="hidden" />
			{:else}
				<div
					class="flex items-center justify-center border h-56 font-semibold hover:font-extrabold text-blue-600"
				>
					<label
						class=" text-3xl w-full h-full flex justify-center items-center cursor-pointer"
						for="fileupload"
						>Click To Upload
					</label>
				</div>
			{/if}
		</div>
	</div>

	<div class="flex flex-row justify-between">
		<!-- setting section -->
		<div class="">
			<div>
				<!-- title -->
				<div>
					<h2 class=" underline text-center mt-3 mb-1 p-1">Change field details:</h2>
					{#if fields.length == 0}
						<p class="text-sm font-normal text-green-500 p-1">
							*click somewhere on loaded template to open settings
						</p>
					{/if}
				</div>

				<!-- settings -->
				<div class="mt-3">
					{#if fields.length != 0}
						<div class="  p-2">
							<!-- field select -->

							<div class="flex flex-row items-center">
								<div>Text field :</div>
								<div class="">
									<select on:change={handleSelectChange} name="field-details" class="inputfield">
										{#each fields as field}
											<option value={field.id}>
												{field.value}
											</option>
										{/each}
									</select>
								</div>
							</div>

							<!-- font  -->

							<div class="mt-1">
								<label for="font-size">Font size : </label>
								<input
									type="number"
									name="font-size"
									class="inputfield"
									min="1"
									max="100"
									on:change={setFontSize}
									value={fields[0].fontSize}
								/>
							</div>

							<!-- Value Change -->

							<div class="mt-1">
								<label for="value">Value : </label>
								<input type="text" name="value" class="inputfield" on:change={setValue} />
							</div>

							<!--  text Input -->

							<div class="mt-1">
								<label for="demo-text">Text :</label>
								<input type="text" name="demo-text" class="inputfield" on:change={setDemoText} />
							</div>

							<!-- font style -->

							<div class="flex flex-row items-center mt-1">
								<div>Font Style :</div>
								<div class="">
									<select on:change={handleSelectChange} name="Font-style" class="inputfield">
										{#each fields as field}
											<option value={field.id}>
												{field.fontFamily}
											</option>
										{/each}
									</select>
								</div>
							</div>

							<!-- Color -->
							<div class="border-b">
								<label for="color">Color: </label>
								<input type="color" on:change={setColor} class="ml-2 mt-1 mb-2" name="color" />
							</div>

							<button class="mt-3 bg-gray-700 p-2 rounded-md border">Update data</button>
						</div>
					{/if}
				</div>
			</div>
		</div>

		<!-- Template add section -->
		<div class="mt-4">
			<div class="mb-2 underline">Upload Template :</div>
			<div class="mt-3">
				<input
					type="file"
					id="fileupload"
					class="preview cursor-pointer fileUpload"
					name="template-img"
					accept="image/*"
					on:change={handleFileInput}
				/>
			</div>

			<div class="mt-2">
				<div class="mb-2 underline">Upload CSV file :</div>

				<input
					id="csv-reader"
					class=" cursor-pointer fileUpload"
					type="file"
					bind:this={csvReader}
					accept=".csv"
				/>
			</div>

			<div class="mt-6 border-t">
				{#if fields.length != 0}
					{#if isCSVfileLoaded}
						<p class="text-xs ml-1 mt-2">*generate certificates using CSV data</p>
						<button
							on:click={generateCertificates}
							class="mt-2 bg-gray-700 p-2 w-full rounded-md border">Generate Certificates</button
						>
					{/if}
					{#if generated}
						<button
							on:click={download}
							class="mt-2 bg-green-700 p-2 w-full rounded-md text-white border">Download</button
						>
					{/if}
				{/if}
			</div>
		</div>
	</div>

	<div class="hidden" bind:this={certificatesContainer} />
	<div bind:this={previewsContainer} />
</div>

<style>
	.preview {
		width: 100%;
		height: 100%;
		object-fit: contain;
		max-width: 900px;
		display: block;
	}
	.borderbtm {
		border-bottom: 1px solid white;
	}

	input,
	select {
		background: transparent;
	}

	.inputfield {
		border: 1px solid;
		background-color: darkgrey;
		margin-left: 10px;
		padding-top: 2px;
		outline: none;
		color: black;
		padding: 2px;
	}
	.fileUpload {
		outline: none;
		color: rgb(59, 130, 246);
	}
</style>
