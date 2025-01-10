<script lang="ts">
	import '../app.css'
	import { i18n } from '$lib/i18n'
	import { ParaglideJS } from '@inlang/paraglide-sveltekit'

	let { children } = $props()
</script>

<header>
	<nav></nav>
</header>
<main class="container mx-auto py-8">
	<ParaglideJS {i18n}>
		{@render children()}
	</ParaglideJS>
</main>
