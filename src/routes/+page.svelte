<script lang="ts">
	import Certifly from './certifly.svelte';
</script>

<div class="flex flex-row justify-between items-center mt-5 ml-10 mr-10">
	<div class="flex flex-row text-4xl font-bold items-center">
		<p class="text-green-600 font-extrabold text-5xl">C</p>
		<h1 class="">ertifly</h1>
	</div>
	<div class="flex font-semibold cursor-pointer hover:text-green-500 mt-3">
		<p class="borderbtm">How to use</p>
		<p class="ml-1">?</p>
	</div>
</div>

<Certifly />

<style>
	.borderbtm {
		border-bottom: 1px solid;
	}
</style>
