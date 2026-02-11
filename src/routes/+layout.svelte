<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<title>Keroyokan | Coming Soon</title>
	<meta
		name="description"
		content="Keroyokan is preparing a new collaborative experience. Phase -1 is landing soon."
	/>
	<meta property="og:title" content="Keroyokan | Coming Soon" />
	<meta
		property="og:description"
		content="Keroyokan is preparing a new collaborative experience. Phase -1 is landing soon."
	/>
</svelte:head>
{@render children()}
