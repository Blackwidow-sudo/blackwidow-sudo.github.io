<script lang="ts">
	import Camera from '$lib/components/Camera.svelte'
	import type { PageData } from './$types'

	let { data }: { data: PageData } = $props()
</script>

<Camera debug />
