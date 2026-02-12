<script lang="ts">
	import './layout.css';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href="/favicon_keroyokan.png" />
	<title>Keroyokan Multisemesta | all realities, in one deck</title>
	<meta
		name="description"
		content="all realities, in one deck. soon."
	/>
	<meta property="og:title" content="Keroyokan Multisemesta | all realities, in one deck" />
	<meta
		property="og:description"
		content="all realities, in one deck. soon."
	/>
	<meta property="og:type" content="website" />
	<meta property="og:image" content="/og-image.jpg" />
	<meta property="og:image:type" content="image/jpeg" />
	<meta property="og:image:width" content="800" />
	<meta property="og:image:height" content="600" />
	<meta property="og:image:alt" content="Keroyokan Multisemesta - teaser image" />
	<meta property="og:site_name" content="Keroyokan Multisemesta" />
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="Keroyokan Multisemesta | all realities, in one deck" />
	<meta
		name="twitter:description"
		content="all realities, in one deck. soon."
	/>
	<meta name="twitter:image" content="/og-image.jpg" />
	<meta name="twitter:image:alt" content="Keroyokan Multisemesta - teaser image" />
</svelte:head>
{@render children()}
