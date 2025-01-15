<script lang="ts">
	import SwapCamera from 'lucide-svelte/icons/refresh-ccw'
	import Camera from 'lucide-svelte/icons/camera'
	import { browser } from '$app/environment'
	import { Button } from '$lib/components/ui/button'
	import { onDestroy, onMount, untrack } from 'svelte'

	type FacingMode = 'environment' | 'user'

	interface Props {
		debug?: boolean
	}

	let { debug }: Props = $props()

	let video: HTMLVideoElement | undefined = $state()
	let stream: MediaStream | null = $state(null)

	let track = $derived.by(() => {
		const [track] = stream?.getVideoTracks() ?? []

		return track || null
	})
	let settings = $derived(track?.getSettings() ?? {})
	let capabilities = $derived(track?.getCapabilities() ?? {})
	let constraints = $derived(track?.getConstraints() ?? {})

	let debugJson = $derived.by(() => {
		const cameraAttributes = {
			label: track?.label,
			settings,
			capabilities,
			constraints
		}

		return JSON.stringify(cameraAttributes, undefined, 4)
	})

	let hasCapability = $derived.by(() => (capability: string) => {
		return capabilities && capability in capabilities
	})

	$effect(() => {
		const videoEl = untrack(() => video)

		if (videoEl && stream) {
			videoEl.srcObject = stream
		}
	})

	onMount(() => {
		browser && startCamera('environment')
	})

	onDestroy(() => {
		stopCamera()
	})

	$inspect({ stream, track, settings, capabilities, constraints })

	async function onZoom(e: Event) {
		try {
			const zoom = (e.target as HTMLInputElement).valueAsNumber

			await track?.applyConstraints({
				advanced: [{ zoom }]
			})
		} catch (err) {
			console.error(err)
		}
	}

	async function startCamera(facingMode: FacingMode = 'environment') {
		if (stream instanceof MediaStream) {
			stopCamera()
		}

		try {
			stream = await window.navigator.mediaDevices.getUserMedia({
				audio: false,
				video: {
					facingMode,
					width: { min: 800, max: 1920 }
				}
			})
		} catch (err) {
			console.error(err)
		}
	}

	function stopCamera() {
		stream?.getTracks().forEach((track) => track.stop())

		stream = null
	}

	async function switchCamera(e: Event) {
		const facingMode = constraints?.facingMode === 'environment' ? 'user' : 'environment'

		await startCamera(facingMode)
	}
</script>

<div class="relative mx-auto w-full">
	<div class="absolute flex size-full flex-col items-center justify-between p-4">
		<div class="flex w-full flex-grow items-center justify-end">
			{#if hasCapability('zoom')}
				<input
					type="range"
					class="slider"
					min={capabilities?.zoom?.min}
					max={capabilities?.zoom?.max}
					step={capabilities?.zoom?.step}
					value={settings?.zoom}
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
		class="mx-auto w-full"
		autoplay
		playsinline
		bind:this={video}></video>
</div>
{#if debug}
	<pre class="mt-8 rounded-md bg-slate-800 p-4 text-slate-300">
		{debugJson}
	</pre>
{/if}

<style>
	.slider {
		writing-mode: vertical-lr;
		direction: rtl;
		appearance: slider-vertical;
		width: 16px;
		vertical-align: bottom;
	}
</style>
