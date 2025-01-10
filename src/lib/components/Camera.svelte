<script lang="ts">
	import { untrack } from 'svelte'

	let {} = $props()

	let video: HTMLVideoElement | undefined = $state()
	let stream: MediaStream | undefined = $state()

	let track: MediaStreamTrack | undefined = $derived(stream?.getVideoTracks()[0])
	let settings: MediaTrackSettings | undefined = $derived(track?.getSettings())
	let capabilities: MediaTrackCapabilities | undefined = $derived(track?.getCapabilities())

	$effect(() => {
		initCamera()
			.then((videoStream) => {
				stream = videoStream

				if (video) {
					video.srcObject = videoStream
				}
			})
			.catch(console.error)

		return () => {
			stream?.getTracks().forEach((track) => track.stop())
		}
	})

	function initCamera(facingMode = 'environment') {
		return navigator.mediaDevices.getUserMedia({
			audio: false,
			video: {
				facingMode,
				width: { ideal: 9999 },
				height: { ideal: 9999 }
			}
		})
	}

	function hasCapability(capability: string) {
		return capabilities && capability in capabilities
	}

	$inspect(capabilities, settings)
</script>

<div class="container relative mx-auto w-full p-4">
	<div class="absolute flex flex-col items-end">
		<div class="flex gap-2">
			<button class="rounded-full p-4">+</button>
		</div>
	</div>
	<!-- svelte-ignore a11y_media_has_caption -->
	<video
		class="mx-auto"
		width={settings?.width}
		height={settings?.height}
		autoplay
		playsinline
		bind:this={video}></video>
</div>
