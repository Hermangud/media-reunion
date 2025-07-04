<script lang="ts">
	import { onMount } from 'svelte';

	let isMounted = $state(false);
	let { children } = $props();

	onMount(() => {
		document.body.classList.add('bg-dark', 'bg-gradient');
		document.body.setAttribute('data-bs-theme', 'dark');
		isMounted = true;
	});
</script>

<svelte:head>
	<link
		href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
		rel="stylesheet"
		integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
		crossorigin="anonymous"
	/>

	<script
		src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
		integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz"
		crossorigin="anonymous"
	></script>
</svelte:head>

{#if isMounted}
	<div class="container py-4">
		{@render children?.()}
	</div>
{/if}
