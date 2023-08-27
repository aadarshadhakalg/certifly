<script lang="ts">
	let isTemplateLoaded = false;
	let origDimensions: { width: number; height: number } = { width: 0, height: 0 };

	let templateCanvas: HTMLCanvasElement;
	let templateDisplay: HTMLImageElement;

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
		ctx.font = '14px Cursive';
		ctx.fillText('Hello World', offsetXCanvas, offsetYCanvas);
		templateDisplay.src = templateCanvas.toDataURL() as string;
	};
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

<style>
	#display-template {
		width: 100%;
		height: 100%;
		object-fit: contain;
		max-width: 900px;
		display: block;
	}
</style>
