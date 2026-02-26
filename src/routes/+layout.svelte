<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.png';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" type="image/png" href={`${favicon}?v=2`} /></svelte:head>
{@render children()}
