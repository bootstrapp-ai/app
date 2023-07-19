<script>
	import { onMount } from 'svelte';
	let message = '';
	let loading = false;
	let fileUrl = '';
  
	const loader = `<div class="loader">Loading...</div>`;
  
	async function submitForm(e) {
	  e.preventDefault();
	  loading = true;
	  const encodedMessage = encodeURIComponent(message);
	  const response = await fetch(`https://ed34rxgjw9.execute-api.sa-east-1.amazonaws.com/copywrite?message=${encodedMessage}`);
	  const data = await response.json();
	  fileUrl = data.file;
	  loading = false;
	}
	let loadingMessages = [
		'Setting up your project...',
		'Creating some content...',
		'Choosing the perfect color scheme...',
		'Getting ready to launch...',
		'Almost there...',
	];
	let currentLoadingMessageIndex = 0;

	// Create an interval that changes the loading message every few seconds
	onMount(() => {
		const intervalId = setInterval(() => {
			currentLoadingMessageIndex = (currentLoadingMessageIndex + 1) % loadingMessages.length;
		}, 2000); // change message every 2 seconds

		return () => {
			clearInterval(intervalId); // clear interval on unmount
		}
	});

	// Redirect when fileUrl changes.
	$: if (fileUrl) {
	  window.location.href = fileUrl;
	}
  </script>
  
  <div class="container h-full w-full mx-auto flex justify-center items-center">
	{#if !loading}
	  <div class="space-y-10 w-full text-center flex flex-col items-center">
		<h2 class="h2">Ready to launch your project?</h2>
		<form class="w-full md:w-4/5" on:submit={submitForm}>
		  <div class="mb-4 border border-gray-200 rounded-lg bg-gray-50 dark:bg-gray-700 dark:border-gray-600">
			<div class="flex items-center justify-between px-3 py-2 border-b dark:border-gray-600">
			  <div class="flex flex-wrap items-center divide-gray-200 sm:divide-x dark:divide-gray-600">
				<input type="checkbox" checked class="m-2" />
				<span>Express</span>
			  </div>
			</div>
			<div class="px-4 m-1 py-2 bg-white rounded-b-lg dark:bg-gray-800">
			  <textarea id="editor" rows="8" bind:value={message} class="w-full h-full block px-0 text-sm text-gray-800 bg-white border-0 dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400" placeholder="Write about your project" required></textarea>
			</div>
		  </div>
		  <div class="flex justify-center space-x-2">
			<button class="btn variant-filled" type="submit">Launch for Free</button>
		  </div>
		</form>
	  </div>
	{:else}
	<h2 class="h3 loading-message m-20">{loadingMessages[currentLoadingMessageIndex]}</h2>
	<div class="loader-container">
	<div class="loader"></div>	
	</div>
	{/if}
  </div>
  
  <style>
	.loader-container {
	  display: flex;
	  align-items: center;
	  justify-content: center;
	  height: 100%;
	}
	.loader {
	  border: 16px solid #f3f3f3;
	  border-top: 16px solid #3498db;
	  border-radius: 50%;
	  width: 120px;
	  height: 120px;
	  animation: spin 2s linear infinite;
	}

	.loading-message {
		font-size: 1.2em;
		margin-top: 20px;
		text-align: center;
  	}
	@keyframes spin {
	  0% { transform: rotate(0deg); }
	  100% { transform: rotate(360deg); }
	}
  </style>
  