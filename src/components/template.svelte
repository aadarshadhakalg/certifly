<script lang="ts">
	import { onMount, createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function loadedInfo(path: string) {
		dispatch('templatestatus', {
			loaded: true,
			imagePath: path
		});
	}

	onMount(() => {
		const templateInput = document.getElementById('template-input') as HTMLInputElement;
		const templatePreview = document.getElementById('template-preview') as HTMLDivElement;

		templateInput.addEventListener('change', (event) => {
			const selectedtemplate = (event.target as HTMLInputElement).files?.[0];

			if (selectedtemplate) {
				const reader = new FileReader();

				reader.onload = (e: ProgressEvent<FileReader>) => {
					const img = document.createElement('img');
					img.src = e.target?.result as string;
					img.alt = 'Uploaded template';
					img.style.maxWidth = '100%';
					img.style.maxHeight = '100%';

					// Clear previous preview
					templatePreview.innerHTML = '';

					// Append the new preview template
					templatePreview.appendChild(img);

					loadedInfo(e.target?.result as string);
				};

				reader.readAsDataURL(selectedtemplate);
			}
		});
	});
</script>

<p>Drop the template here.</p>
<input type="file" id="template-input" accept="image/png, image/jpeg" />
<div id="template-preview" />
