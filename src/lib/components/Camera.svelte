<script lang="ts">
	import SwapCamera from 'lucide-svelte/icons/refresh-ccw'
	import Camera from 'lucide-svelte/icons/camera'
	import { Button } from '$lib/components/ui/button'
	import { untrack } from 'svelte'

	let video: HTMLVideoElement | undefined = $state()
	let stream: MediaStream | undefined = $state()

	let camera = $derived.by(() => {
		if (stream) {
			const [track] = stream?.getVideoTracks()

			return {
				track,
				settings: track?.getSettings(),
				capabilities: track?.getCapabilities(),
				constraints: track?.getConstraints()
			}
		}

		return {}
	})

	$effect(() => {
		initCamera()
			.then((videoStream) => {
				untrack(() => (stream = videoStream))

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
				width: { ideal: 1920 },
				height: { ideal: 1080 }
			}
		})
	}

	function hasCapability(capability: string) {
		return camera.capabilities && capability in camera.capabilities
	}

	async function onZoom(e: Event) {
		try {
			const zoom = (e.target as HTMLInputElement).valueAsNumber

			await camera.track?.applyConstraints({
				advanced: [{ zoom }]
			})
		} catch (err) {
			console.error(err)
		}
	}

	async function switchCamera(e: Event) {
		try {
			const facingMode = camera.constraints?.facingMode === 'environment' ? 'user' : 'environment'

			const videoStream = await initCamera(facingMode)

			stream?.getTracks().forEach((track) => track.stop())

			stream = videoStream

			video.srcObject = videoStream
		} catch (err) {
			console.error(err)
		}
	}
</script>

<div class="relative mx-auto">
	<div class="absolute flex size-full flex-col items-center justify-between p-4">
		<div class="flex w-full flex-grow items-center justify-end">
			{#if hasCapability('zoom')}
				<input
					type="range"
					class="slider"
					min={camera.capabilities?.zoom?.min}
					max={camera.capabilities?.zoom?.max}
					step={camera.capabilities?.zoom?.step}
					value={camera.settings?.zoom}
					oninput={onZoom} />
			{/if}
		</div>
		<div class="flex w-full items-center justify-center gap-2">
			<Button
				class="rounded-full"
				variant="outline"
				size="icon">
				<Camera />
			</Button>
			<Button
				class="rounded-full"
				variant="outline"
				size="icon"
				onclick={switchCamera}>
				<SwapCamera />
			</Button>
		</div>
	</div>
	<!-- svelte-ignore a11y_media_has_caption -->
	<video
		class="mx-auto"
		width={camera.settings?.width}
		height={camera.settings?.height}
		autoplay
		playsinline
		bind:this={video}></video>
</div>

<style>
	.slider {
		writing-mode: vertical-lr;
		direction: rtl;
		appearance: slider-vertical;
		width: 16px;
		vertical-align: bottom;
	}
</style>
