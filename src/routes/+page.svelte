<script>
	import { Auth } from 'aws-amplify';
	import { onMount } from 'svelte';
	let username = '';  
 	let psw = '';  
 	let loggedIn = false;
 	let registering = false;
 	let code = ''; // validation code

	let message = '';
	let loading = false;
	let fileUrl = '';
  
	const loader = `<div class="loader">Loading...</div>`;
    const signIn = async (username, password) => {
		try {
		console.log("signIn requested");
		console.log(username, password);
		const user = await Auth.signIn(username, password);
		console.log(user);
		loggedIn = true;
		return user;
		} catch (error) {
		console.error(error);
		}
	};

	async function signOut() {
		try {
		await Auth.signOut();
		console.log('User logged out');
		loggedIn = false;
		// Redirect or perform other actions on successful logout
		} catch (error) {
		console.log('Error:', error);
		// Handle logout error
		}
  	}
	
	const signUp = async (username, password) => {
	try {
	console.log("Signup requested");
	console.log(username, password);
	const { user } = await Auth.signUp({
		username, 
		password, 
		autoSignIn: { // optional - enables auto sign in after user is confirmed
					enabled: true,
					}
	});
	
	console.log(user);
	registering = true;
	return user;
	} catch (error) {
	console.log('error signing up:', error);
	}
	};
	
	async function confirmSignUp(username, code) {
		try {
			await Auth.confirmSignUp(username, code);
		loggedIn = true;
		} catch (error) {
			console.log('error confirming sign up', error);
		}
	}
	async function resendConfirmationCode(username) {
		try {
			await Auth.resendSignUp(username);
			console.log('code resent successfully');
		} catch (err) {
			console.log('error resending code: ', err);
		}
	}

	const getCurrentUser = async () => {
		try {
					console.log("getCurrentUser clicked");
		const user = await Auth.currentAuthenticatedUser({
			bypassCache: false,
		});
		console.log(user);
		console.info(JSON.stringify(user));
		return user;
		} catch (error) {
		console.error(error);
		}
	};

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
  {#if loggedIn}
	<div>
		<p>Welcome, you are logged in!</p>
		<button on:click={() => getCurrentUser()}>GetUser</button>
		<button on:click={signOut}>Logout</button>
	</div>
	
	{:else}
		{#if registering}
		<label for="code"><b>Code</b></label>
		<input type="text" placeholder="Enter code" name="code" required bind:value={code} />
		<button on:click={() => confirmSignUp(username, code)}>Verify code</button>
		<button on:click={() => resendConfirmationCode(username)}>Resend code</button>
		
		{:else}
	<div>
	<label for="uname"><b>Username</b></label>
	<input type="text" placeholder="Enter Username" name="uname" required bind:value={username} />
	
	<label for="psw"><b>Password</b></label>
	<input type="password" placeholder="Enter Password" name="psw" required bind:value={psw} />
	
 	 <button type="submit" on:click={() => signIn(username, psw)}>Login</button>
	 <button type="submit" on:click={() => signUp(username, psw)}>Register</button>
   </div>
   {/if}
 {/if}
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
  