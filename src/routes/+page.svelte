<script lang="ts">
	import loam from 'loam';
	import { onMount } from 'svelte';

	let isInit: Boolean = false;
	let meta: any;

	onMount(async () => {
		let gdal = loam.initialize(window.location.origin);

		isInit = await gdal;

		console.log(gdal);
	});

	async function getTiffMeta(tiffFile: File) {
		let d = await loam.open(tiffFile);
		let wkt = await d.wkt();
		let width = await d.width();
		let height = await d.height();
		let numBands = await d.count();
		let transform = await transformTiffAffine(tiffFile);
		console.log(d, wkt, width, height);
		return {
			wkt,
			width,
			height,
			numBands,
			transform
		};
	}
	async function transformTiffAffine(tiffFile: File) {
		let d = await loam.open(tiffFile);
		let tf = await d.transform();

		return tf;
	}

	async function onFileInput(e: Event | any) {
		const file = e.target.files[0];
		const result = await getTiffMeta(file);
		meta = result;
		console.log(result);
	}
</script>

<div>
	<h1>Wrapper</h1>
	{#if isInit != undefined || isInit != null}
		<p>GDAL ready</p>
		<input type="file" on:change={onFileInput} />

		<h2>Metadata</h2>
		{#if meta?.wkt}
			<p>WKT: {meta?.wkt}</p>
		{/if}
		{#if meta?.height && meta?.width}
			<p>height: {meta.height}</p>
			<p>width: {meta.width}</p>
		{/if}
		{#if meta?.numBands}
			<p>{meta.numBands}</p>
		{/if}
		{#if meta?.transform}
			<p>{meta.transform}</p>
		{/if}
	{:else}
		<p>Loading GDAL...</p>
	{/if}
</div>

<style>
	div {
		width: 100%;
		max-width: 100%;
	}
</style>
