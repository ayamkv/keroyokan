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
			.from('[data-reveal="copy"]', { autoAlpha: 0, y: 14 }, '-=0.55')
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
		<h1 id="coming-soon-title" data-reveal="headline">Keroyokan lands soon.</h1>
		<p data-reveal="copy" class="lead">
			A calmer hub for coordinated work is taking shape. We are polishing the first public drop with
			focus on flow, clarity, and momentum.
		</p>

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
				<div aria-hidden="true" class="glow"></div>
			</div>
		</div>
	</section>
</main>

<style>
	:global(body) {
		background:
			radial-gradient(circle at 16% 14%, rgba(244, 183, 120, 0.26), transparent 44%),
			radial-gradient(circle at 85% 10%, rgba(110, 160, 150, 0.2), transparent 42%),
			radial-gradient(circle at 50% 120%, rgba(166, 132, 88, 0.18), transparent 58%),
			linear-gradient(158deg, #f6efe3 0%, #efe4d4 52%, #e6d8c4 100%);
	}

	.landing {
		min-height: 100svh;
		display: grid;
		place-items: center;
		padding: clamp(1.25rem, 2.6vw, 2.5rem);
		color: #2a2016;
	}

	.hero {
		width: min(68rem, 100%);
		display: grid;
		justify-items: center;
		text-align: center;
		gap: clamp(0.9rem, 2vw, 1.2rem);
	}

	.logo {
		width: clamp(5.5rem, 11vw, 7.25rem);
		height: auto;
		filter: drop-shadow(0 16px 20px rgba(67, 43, 23, 0.25));
	}

	.kicker {
		margin: 0;
		font-size: 0.85rem;
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: #755436;
	}

	h1 {
		margin: 0;
		max-width: 18ch;
		font-size: clamp(2rem, 6.8vw, 4.4rem);
		line-height: 1.05;
		text-wrap: balance;
		font-weight: 700;
	}

	.lead {
		margin: 0;
		max-width: 60ch;
		font-size: clamp(1rem, 1.8vw, 1.2rem);
		line-height: 1.6;
		color: #4a3625;
		text-wrap: pretty;
	}

	.card-wrap {
		width: min(100%, 34rem);
		margin-top: clamp(0.6rem, 2vw, 1rem);
		perspective: 1100px;
	}

	.tilt-card {
		position: relative;
		border-radius: 1.35rem;
		overflow: hidden;
		transform-style: preserve-3d;
		transform: rotateX(var(--tilt-x, 0deg)) rotateY(var(--tilt-y, 0deg));
		transition: transform 220ms ease-out;
		outline: 1px solid rgba(118, 86, 56, 0.22);
		box-shadow:
			0 24px 42px rgba(70, 45, 24, 0.24),
			0 6px 16px rgba(105, 74, 46, 0.16);
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

	.glow {
		position: absolute;
		inset: 0;
		pointer-events: none;
		background:
			radial-gradient(circle at var(--glow-x, 50%) var(--glow-y, 50%), rgba(255, 236, 190, 0.26), transparent 36%),
			linear-gradient(118deg, rgba(146, 181, 164, 0.14), rgba(208, 150, 90, 0.11) 55%, rgba(87, 64, 42, 0.08));
		mix-blend-mode: soft-light;
	}

	@media (max-width: 42rem) {
		.hero {
			align-content: center;
		}

		.lead {
			max-width: 34ch;
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.tilt-card {
			transition: none;
			transform: none;
		}
	}
</style>
