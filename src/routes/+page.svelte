<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let canTilt = $state(false);
	let tiltX = $state(0);
	let tiltY = $state(0);
	let glowX = $state(50);
	let glowY = $state(50);

	const maxTilt = 9;

	function handlePointerMove(event: PointerEvent) {
		if (!canTilt) return;

		const target = event.currentTarget as HTMLDivElement | null;
		if (!target) return;

		const rect = target.getBoundingClientRect();
		const x = (event.clientX - rect.left) / rect.width;
		const y = (event.clientY - rect.top) / rect.height;

		tiltY = (x - 0.5) * maxTilt * 2;
		tiltX = (0.5 - y) * maxTilt * 2;
		glowX = x * 100;
		glowY = y * 100;
	}

	function resetTilt() {
		tiltX = 0;
		tiltY = 0;
		glowX = 50;
		glowY = 50;
	}

	onMount(() => {
		const hoverMedia = window.matchMedia('(hover: hover) and (pointer: fine)');
		const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

		const updateTiltAvailability = () => {
			canTilt = hoverMedia.matches;
			if (!canTilt) resetTilt();
		};

		updateTiltAvailability();
		hoverMedia.addEventListener('change', updateTiltAvailability);

		if (reduceMotion.matches) {
			return () => hoverMedia.removeEventListener('change', updateTiltAvailability);
		}

		const timeline = gsap.timeline({ defaults: { duration: 0.85, ease: 'power2.out' } });
		timeline
			.from('[data-reveal="logo"]', { autoAlpha: 0, y: 20, scale: 0.97 })
			.from('[data-reveal="headline"]', { autoAlpha: 0, y: 24 }, '-=0.45')
			.from('[data-reveal="card"]', { autoAlpha: 0, y: 28, scale: 0.95 }, '-=0.5');

		return () => {
			timeline.kill();
			hoverMedia.removeEventListener('change', updateTiltAvailability);
		};
	});
</script>

<main class="landing">
	<section class="hero" aria-labelledby="coming-soon-title">
		<img
			data-reveal="logo"
			class="logo"
			src="/keroyokan_logo.png"
			alt="Keroyokan logo"
			width="228"
			height="228"
		/>

		<p class="kicker">Phase -1</p>
		<h1 id="coming-soon-title" data-reveal="headline">
			<span class="headline-main">all realities, in one deck</span>
			<span class="headline-secondary">soon</span>
		</h1>

		<div data-reveal="card" class="card-wrap">
			<div
				class="tilt-card"
				class:tilt-enabled={canTilt}
				role="img"
				aria-label="Interactive launch artwork preview"
				onpointermove={handlePointerMove}
				onpointerleave={resetTilt}
				style={`--tilt-x:${tiltX}deg; --tilt-y:${tiltY}deg; --glow-x:${glowX}%; --glow-y:${glowY}%;`}
			>
				<img
					class="cover"
					src="/card_backcover.png"
					alt="Decorative card artwork previewing the Keroyokan launch"
				/>
				<img
					aria-hidden="true"
					class="cover holo-dup"
					src="/card_backcover.png"
					alt=""
				/>
				<div aria-hidden="true" class="glow"></div>
				<div aria-hidden="true" class="holo"></div>
			</div>
		</div>
	</section>
</main>

<style>
	:global(html) {
		color-scheme: dark;
	}

	:global(body) {
		background:
			radial-gradient(circle at 14% 12%, rgb(253 0 92 / 24%), transparent 44%),
			radial-gradient(circle at 84% 10%, rgb(254 204 57 / 18%), transparent 41%),
			radial-gradient(circle at 48% 118%, rgb(253 0 92 / 13%), transparent 58%),
			linear-gradient(165deg, #030915 0%, #071224 48%, #0b1c31 100%);
	}

	.landing {
		min-height: 100svh;
		display: grid;
		place-items: center;
		padding: clamp(1rem, 2.1vw, 1.8rem);
		color: #ecf2ff;
		font-family: 'Geist', sans-serif;
	}

	.hero {
		width: min(30rem, 100%);
		display: grid;
		justify-items: center;
		text-align: center;
		gap: clamp(0.45rem, 1.5vw, 0.85rem);
	}

	.logo {
		width: clamp(3.5rem, 8vw, 5rem);
		height: auto;
		filter: drop-shadow(0 14px 18px rgba(25, 36, 68, 0.34));
	}

	.kicker {
		margin: 0;
		font-size: 0.7rem;
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: rgba(218, 229, 255, 0.75);
		font-family: 'Geist Mono', monospace;
	}

	h1 {
		margin: 0;
		display: grid;
		gap: 0.3rem;
		justify-items: center;
		line-height: 1;
		text-wrap: balance;
	}

	.headline-main {
		display: block;
		max-width: 18ch;
		font-size: clamp(1.45rem, 4.1vw, 2.15rem);
		font-weight: 600;
		font-family: 'Geist Pixel', monospace;
		letter-spacing: -0.1em;
	}

	.headline-secondary {
		display: block;
		font-family: 'Geist Pixel', monospace;
		font-size: clamp(0.82rem, 1.7vw, 1rem);
		letter-spacing: 0.14em;
		color: rgba(202, 217, 255, 0.86);
		text-transform: lowercase;
	}

	.card-wrap {
		width: min(100%, 12.2rem);
		margin-top: clamp(0.45rem, 1.4vw, 0.75rem);
		perspective: 1100px;
	}

	.tilt-card {
		position: relative;
		border-radius: 0.9rem;
		overflow: hidden;
		transform-style: preserve-3d;
		transform: translateZ(0) rotateX(var(--tilt-x, 0deg)) rotateY(var(--tilt-y, 0deg))
			scale(var(--card-scale, 1));
		transition: transform 220ms ease-out, box-shadow 220ms ease-out;
		outline: 1px solid rgba(117, 178, 235, 0.34);
		box-shadow:
			0 16px 28px rgba(4, 8, 20, 0.52),
			0 4px 10px rgba(17, 42, 84, 0.35);
	}

	@media (hover: hover) and (pointer: fine) {
		.tilt-card:hover {
			--card-scale: 1.025;
			box-shadow:
				0 20px 34px rgba(6, 11, 26, 0.62),
				0 7px 16px rgba(30, 52, 103, 0.36);
		}
	}

	.tilt-enabled {
		cursor: grab;
	}

	.tilt-enabled:active {
		cursor: grabbing;
	}

	.cover {
		display: block;
		width: 100%;
		height: auto;
		object-fit: cover;
	}

	.holo-dup {
		position: absolute;
		inset: 0;
		height: 100%;
		opacity: 0.18;
		mix-blend-mode: screen;
		filter: hue-rotate(24deg) saturate(1.18);
		pointer-events: none;
	}

	.glow {
		position: absolute;
		inset: 0;
		pointer-events: none;
		background:
			radial-gradient(circle at var(--glow-x, 50%) var(--glow-y, 50%), rgba(214, 240, 255, 0.28), transparent 42%),
			linear-gradient(118deg, rgba(105, 206, 255, 0.16), rgba(255, 193, 104, 0.08) 56%, rgba(73, 141, 235, 0.16));
		mix-blend-mode: soft-light;
	}

	.holo {
		position: absolute;
		inset: 0;
		pointer-events: none;
		opacity: 0.24;
		background:
			conic-gradient(
				from 145deg at 24% 22%,
				rgba(107, 225, 255, 0.2),
				rgba(255, 190, 94, 0.14),
				rgba(80, 127, 228, 0.2),
				rgba(107, 225, 255, 0.2)
			),
			linear-gradient(125deg, rgba(255, 252, 242, 0.05), rgba(255, 255, 255, 0) 44%);
		mix-blend-mode: screen;
	}

	.holo::after {
		content: '';
		position: absolute;
		inset: -22%;
		background: linear-gradient(
			110deg,
			rgba(255, 255, 255, 0) 36%,
			rgba(122, 225, 255, 0.14) 48%,
			rgba(255, 191, 86, 0.12) 52%,
			rgba(255, 255, 255, 0) 66%
		);
		opacity: 0.16;
		mix-blend-mode: screen;
		transform: translate3d(-20%, 0, 0) rotate(-8deg);
		will-change: transform, opacity;
		animation: holo-shimmer 8.5s ease-in-out infinite;
	}

	@keyframes holo-shimmer {
		0%,
		100% {
			transform: translate3d(-22%, 0, 0) rotate(-8deg);
			opacity: 0.1;
		}

		50% {
			transform: translate3d(20%, 0, 0) rotate(-8deg);
			opacity: 0.2;
		}
	}

	@media (max-width: 42rem) {
		.hero {
			align-content: center;
			width: min(25rem, 100%);
		}

		.card-wrap {
			width: min(100%, 11.4rem);
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.tilt-card {
			transition: none;
			transform: none;
		}

		.holo {
			opacity: 0.2;
		}

		.holo::after {
			animation: none;
			opacity: 0.08;
			transform: translate3d(0, 0, 0) rotate(-8deg);
		}
	}
</style>
