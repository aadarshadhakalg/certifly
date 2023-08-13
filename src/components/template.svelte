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

		templateInput.addEventListener('change', (event) => {
			const selectedtemplate = (event.target as HTMLInputElement).files?.[0];

			if (selectedtemplate) {
				const reader = new FileReader();

				reader.onload = (e: ProgressEvent<FileReader>) => {
					loadedInfo(e.target?.result as string);
				};

				reader.readAsDataURL(selectedtemplate);
			}
		});
	});
</script>

<p>Drop the template here.</p>
<input type="file" id="template-input" accept="image/png, image/jpeg" />
