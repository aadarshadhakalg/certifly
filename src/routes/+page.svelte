<script lang="ts">
	let displayCertificate = '';
	let isTemplateLoaded = false;

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
			displayCertificate = reader.result as string;
		};
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
<img id="display-template" src={displayCertificate} alt="template-display" />

<style>
	#display-template {
		width: 100%;
		height: 100%;
		object-fit: contain;
		max-width: 900px;
		display: block;
	}
</style>
