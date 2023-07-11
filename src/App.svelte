<script>
	import { onMount } from 'svelte';

	let images = [];
	let uploadWidget;

	const CLOUDINARY_CLOUD_NAME = 'dxhvg0tuo';
	const CLOUDINARY_API_KEY = '525634432671769';
	const CLOUDINARY_API_SECRET = 'YYeeaBzlUjWylZNKUbIO9in-HGE';

	
	onMount(() => {
		const script = document.createElement('script');
		script.src = 'https://upload-widget.cloudinary.com/global/all.js';
		script.async = true;
		script.onload = () => {
			uploadWidget = cloudinary.createUploadWidget(
				{
					cloudName: CLOUDINARY_CLOUD_NAME,
					uploadPreset: 'ml_default' // Replace with your upload preset name
				},
				(error, result) => {
					if (!error && result && result.event === 'success') {
						console.log('Done! Here is the image info: ', result.info);
						fetchImages();
					}
				}
			);
		};
		document.body.appendChild(script);

		fetchImages();
	});

	async function fetchImages() {
		const response = await fetch(`https://cors-anywhere.herokuapp.com/https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/resources/image?max_results=100`, {
			headers: {
				Authorization: `Basic ${btoa(`${CLOUDINARY_API_KEY}:${CLOUDINARY_API_SECRET}`)}`
			}
		});

		const data = await response.json();
		images = data.resources;
	}

	fetchImages();

	
	function openWidget() {
		uploadWidget.open();
	}

</script>

<style>
	.cloudinary-button {
		margin: 10px;
		padding: 10px 20px;
		background-color: #4CAF50;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		position: absolute;
		top: 10px;
		right: 10px;
	}

	.image-container {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
		grid-gap: 10px;
		margin-top: 20px;
	}

	.image-item {
		width: 100%;
		height: auto;
		overflow: hidden;
	}

	.image-item img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}
</style>

<div class="image-container">
	{#each images as image}
	<div class="image-item">
		<img src={image.url} alt={image.public_id} />
	</div>
	{/each}
</div>

<button id="upload_widget" class="cloudinary-button" on:click={openWidget}>
	Upload files
</button>
