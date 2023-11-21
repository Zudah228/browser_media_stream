<script lang="ts">
	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';

	let mediaRecorder: MediaRecorder | undefined;

	const startRecord = async () => {
		try {
			const devices = await navigator.mediaDevices.enumerateDevices()
			
			const device = devices[0]

			console.dir(device)

			const stream = await navigator.mediaDevices.getUserMedia({
				audio: true,
				peerIdentity: device.deviceId
			});

			const chunks: Blob[] = [];

			mediaRecorder = new MediaRecorder(stream);

			mediaRecorder.ondataavailable = (event) => {
				chunks.push(event.data);
			}

			mediaRecorder.onstart = async (_) => {
				console.trace('onstart media record')
			}

			mediaRecorder.onstop = async (_) => {
				console.trace('onstop media record')

				const blob = new Blob(chunks, { type: 'audio/ogg; codecs=opus' })
				const a = document.createElement('a')
				a.href = window.URL.createObjectURL(blob)
				a.download = 'sample.ogg'
				a.click()
			}

			mediaRecorder?.start()
		} catch (e) {
			console.error(e)
		}
	}

	const stopRecord = () => {
		try {
			mediaRecorder?.stop();
			mediaRecorder = undefined;
		} catch (e) {
			console.error(e)
		}
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<h1>
		<span class="welcome">
			<picture>
				<source srcset={welcome} type="image/webp" />
				<img src={welcome_fallback} alt="Welcome" />
			</picture>
		</span>

		to your new<br />SvelteKit app
	</h1>

	<div>
		<button on:click={startRecord}>Record</button>
		<button on:click={stopRecord} disabled={mediaRecorder == null}>Stop</button>
	</div>
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		display: block;
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
</style>
