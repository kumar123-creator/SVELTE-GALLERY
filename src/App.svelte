<script>
	import { onMount } from 'svelte';
	
	let images = [];
	let filteredImages = [];
	let uploadWidget;
	let searchQuery = '';
	let sortAsc = true;
	let isLightboxOpen = false;
	let currentImageIndex = 0;
	let currentPage = 1;
	
	const imagesPerPage = 9;
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
    uploadPreset: 'd4qzm6g2', // Replace with your upload preset name
    maxFileSize: 5242880, // Maximum file size in bytes (5MB in this example)
    maxImageWidth: 4000, // Maximum image width in pixels
    maxImageHeight: 4000, // Maximum image height in pixels
    sources: ['local'], // Only allow local file uploads
    multiple: false, // Only allow single file upload
    resourceType: 'image', // Restrict uploads to image files
    clientAllowedFormats: ['png', 'jpeg'], // Allow PNG and JPEG formats
    maxFiles: 1, // Maximum number of files to upload (1 in this example)
    cropping: false, // Disable image cropping
    theme: 'minimal', // Use the minimal widget theme
    showPoweredBy: false, // Hide the "Powered by Cloudinary" logo
    text: {
      en: {
        local: {
          browse: 'Choose Image', // Text for the file selection button
          main_title: 'Upload an Image', // Text for the main title
          powered_by: '', // Empty text to hide "Powered by Cloudinary" logo
        },
        queue: {
          title: 'Upload Queue', // Text for the upload queue title
          title_uploading_with_counter: 'Uploading {{num}} Images', // Text for the uploading title with counter
          mini_title_uploading_with_counter: '{{num}} Uploading', // Text for the mini uploading title with counter
        },
        notifications: {
          general_error: 'An error occurred', // Text for the general error notification
          invalid_file_extension: 'Invalid file format. Only PNG and JPEG images are allowed.', // Text for the file extension error notification
          max_file_size_exceeded: 'File size exceeds the maximum limit. Please choose a smaller file.', // Text for the file size error notification
          image_dimensions_exceeded: 'Image dimensions exceed the maximum limit. Please choose a smaller image.', // Text for the image dimensions error notification
          max_files_exceeded: 'You can only upload one file at a time.', // Text for the max files exceeded error notification
        },
      },
    },
  },
  (error, result) => {
    if (!error && result && result.event === 'success') {
      console.log('Done! Here is the image info: ', result.info);
      fetchImages();
      uploadWidget.close(); // Close the upload widget after successful upload
    }
  }
);

	  };
	  document.body.appendChild(script);
	
	  fetchImages();
	});
	
  async function fetchImages() {
  const proxyUrl = 'https://cors-anywhere.herokuapp.com/';
  const apiUrl = `https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/resources/image?max_results=100`;

  const response = await fetch(proxyUrl + apiUrl, {
    headers: {
      Authorization: `Basic ${btoa(`${CLOUDINARY_API_KEY}:${CLOUDINARY_API_SECRET}`)}`,
    },
  });

  const data = await response.json();
  images = data.resources;
  updatePagination();
  filterImages();
}

	
	function filterImages() {
	  filteredImages = images
		.filter((image) => image.public_id.toLowerCase().includes(searchQuery.toLowerCase()))
		.slice((currentPage - 1) * imagesPerPage, currentPage * imagesPerPage);
	  sortData();
	}
	
	function sortData() {
	  if (sortAsc) {
		filteredImages = filteredImages.sort((a, b) => a.public_id.localeCompare(b.public_id));
	  } else {
		filteredImages = filteredImages.sort((a, b) => b.public_id.localeCompare(a.public_id));
	  }
	}
	
	function updateSearchQuery(event) {
	  searchQuery = event.target.value;
	  filterImages();
	}
	
	function toggleSortOrder() {
	  sortAsc = !sortAsc;
	  sortData();
	}
	
	function openWidget() {
	  uploadWidget.open();
	}
	
	function updatePagination() {
	  const totalPages = Math.ceil(images.length / imagesPerPage);
	  if (currentPage > totalPages) {
		currentPage = totalPages;
	  }
	}
	
	
	function goToPage(page) {
    currentPage = page;
    filterImages();
  }

	
	function openLightbox(index) {
	  currentImageIndex = index;
	  isLightboxOpen = true;
	
	  // Disable scrolling on the body while the lightbox is open
	  document.body.style.overflow = 'hidden';
	}
	
	function closeLightbox() {
	  isLightboxOpen = false;
	
	  // Enable scrolling on the body after closing the lightbox
	  document.body.style.overflow = 'auto';
	}
	
  </script>

  <main>
	  <h1 class="gallery-heading">SVELTE GALLERY</h1>
	  
	  <div class="upload-button-container">
		<button id="upload_widget" class="upload-button" on:click={openWidget}>
		  Upload image
		</button>
	  </div>

	  <div class="sort-button">
		<button on:click={toggleSortOrder}>Sort</button>
	  </div>
	  
	  <div class="search-bar">
		<input class="search-input" type="text" placeholder="Search by image name" bind:value={searchQuery} on:input={updateSearchQuery} />
		<p class="search-results">Showing <strong>{filteredImages.length}</strong> out of <strong>{images.length}</strong> images</p>
	  </div>
	  
	  <div class="image-grid">
		{#each filteredImages as image, index}
		  <div class="image-container">
			<img src={image.url} alt={image.public_id} on:click={() => openLightbox(index)} />
			<p class="image-name">{image.public_id}</p>
		  </div>
		  {#if (index + 1) % 3 === 0}
			{#if (index + 1) % 9 === 0}
			  <div class="page-break"></div>
			{/if}
		  {/if}
		{/each}
	  </div>
	  
	  <div class="pagination">
		{#each Array.from({ length: Math.ceil(images.length / imagesPerPage) }) as _, index}
		  <button on:click={() => goToPage(index + 1)} class:active={currentPage === index + 1}>{index + 1}</button>
		{/each}
	  </div>
	  
	  {#if isLightboxOpen}
		<div class="lightbox-overlay" on:click={closeLightbox}>
		  <div class="lightbox-container">
			<button class="lightbox-close-button" on:click={closeLightbox}>Close</button>
			<img class="lightbox-image" src={filteredImages[currentImageIndex].url} alt={filteredImages[currentImageIndex].public_id} />
		  </div>
		</div>
	  {/if}
	  </main>
	  
	<style>
        .gallery-heading {
          text-align: center;
          margin-top: 20px;
          font-size: 50px;
          font-style: italic;
          color: blue;
        }

		.upload-button-container {
          display: flex;
          justify-content: flex-start;
          margin: 20px;
        }

        .upload-button {
		   margin: 40px  20px;
		   padding: 10px 20px;
		   background-color: goldenrod;
		   color: white;
		   border: none;
		   border-radius: 4px;
		   cursor: pointer;
		   position: absolute;
		   top: 120px;
		   right: 10px;
		 }

        @media (max-width: 768px) {
        .upload-button-container {
          justify-content: center;
           margin-top: 20px;
          }
        }

		 .image-grid {
		   display: grid;
		   grid-template-columns: repeat(3, minmax(200px, 1fr));
		   grid-gap: 20px;
		 }
	   
		 .image-container {
		   display: flex;
		   flex-direction: column;
		   align-items: center;
		   box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
		   border-radius: 5px;
		   overflow: hidden;
		   background-color: #fff;
		   transition: transform 0.2s ease;
		 }
	   
		 .image-container:hover {
		   transform: translateY(-5px);
		 }
	   
		 img {
		   width: 100%;
		   height: 300px;
		   object-fit: cover;
		   cursor: pointer;
		   transition: opacity 0.2s ease;
		 }
	   
		 img:hover {
		   opacity: 0.8;
		 }
	   
		 .image-name {
		   font-size: 20px;
		   margin-top: 5px;
		   color: purple;
		   font-style: normal;
		   font-weight: bold;
		 }
	   
		 .search-bar {
		   margin-top: 20px;
		   display: flex;
		   align-items: center;
		 }
	   
		 .search-input {
		   padding: 8px;
		   font-size: 16px;
		   border: 1px solid #ccc;
		   border-radius: 4px;
		   flex-grow: 1;
		   margin-right: 10px;
		 }
	   
		 .search-results {
		   font-size: 16px;
		   color: #666;
		 }
	   
		 .search-results strong {
		   color: #333;
		 }
	   
		 .sort-button {
		   margin-top: 10px;
		   text-align: left;
		 }
	   
		 .sort-button button {
		   padding: 8px 16px;
		   font-size: 16px;
		   border: none;
		   border-radius: 4px;
		   background-color: #555;
		   color: white;
		   cursor: pointer;
		 }
	   
		 .pagination {
		   margin-top: 20px;
		   display: flex;
		   justify-content: center;
		 }
	   
		 .pagination button {
		   padding: 8px 16px;
		   font-size: 16px;
		   border: none;
		   border-radius: 4px;
		   background-color: palegreen;
		   color: black;
		   cursor: pointer;
		   margin: 0 5px;
		 }
	   
		 .pagination button.active {
		   background-color: darkgreen;
		   color: white;
		 }
	   
		 .page-break {
		   flex-basis: 100%;
		   height: 0;
		 }
	   
		 .lightbox-overlay {
		   position: fixed;
		   top: 0;
		   left: 0;
		   width: 100%;
		   height: 100%;
		   background-color: rgba(0, 0, 0, 0.7);
		   display: flex;
		   justify-content: center;
		   align-items: center;
		   z-index: 999;
		 }
	   
		 .lightbox-container {
		   position: relative;
		   max-width: 90%;
		   max-height: 90%;
		 }
	   
		 .lightbox-close-button {
		   position: absolute;
		   top: 10px;
		   right: 10px;
		   padding: 8px 16px;
		   font-size: 16px;
		   border: none;
		   border-radius: 4px;
		   background-color: #555;
		   color: red;
		   cursor: pointer;
		 }
	   
		 .lightbox-image {
		   max-width: 100%;
		   max-height: 100%;
		 }
	</style>