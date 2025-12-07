<script lang="ts">
	import './layout.css'
	import favicon from '$lib/assets/favicon.svg'

	let { children } = $props()
</script>

<svelte:head>
	<link
		rel="icon"
		href={favicon} />
</svelte:head>

<header>
	<nav></nav>
</header>
<main class="container mx-auto py-8">
	{@render children()}
</main>
