<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import Instagram from '@lucide/svelte/icons/instagram';
	import logoImage from '$lib/assets/keroyokan_logo.png?enhanced';
	import logoImageRaw from '$lib/assets/keroyokan_logo.png';
	import cardImage from '$lib/assets/card_backcover.png?enhanced';
	import cardImageUrl from '$lib/assets/card_backcover.png';

	let canTilt = $state(false);
	let isTouchActive = $state(false);
	let cardLoaded = $state(false);
	let tiltX = $state(0);
	let tiltY = $state(0);
	let glowX = $state(50);
	let glowY = $state(50);
	let canReactBackground = $state(false);
	let isBackgroundHovering = $state(false);
	let bgX = $state(50);
	let bgY = $state(50);
	let showLoader = $state(true);
	let floatInnerShiftX = $state(0);
	let floatInnerShiftY = $state(0);

	const maxTilt = 9;

	function updatePointerPosition(event: PointerEvent) {
		const target = event.currentTarget as HTMLElement | null;
		if (!target) return;

		const card = target.querySelector<HTMLElement>('.tilt-card');
		const rect = (card ?? target).getBoundingClientRect();
		const x = Math.min(1, Math.max(0, (event.clientX - rect.left) / rect.width));
		const y = Math.min(1, Math.max(0, (event.clientY - rect.top) / rect.height));

		tiltY = (x - 0.5) * maxTilt * 3;
		tiltX = (0.5 - y) * maxTilt * 3;
		glowX = x * 100;
		glowY = y * 100;
		floatInnerShiftX = (x - 0.5) * 4.4;
		floatInnerShiftY = (y - 0.5) * 3.6;
	}

	function handlePointerMove(event: PointerEvent) {
		const touchTracking = event.pointerType === 'touch' && isTouchActive;
		if (!canTilt && !touchTracking) return;
		updatePointerPosition(event);
	}

	function handlePointerDown(event: PointerEvent) {
		if (event.pointerType !== 'touch') return;

		isTouchActive = true;
		updatePointerPosition(event);

		const target = event.currentTarget as HTMLDivElement | null;
		target?.setPointerCapture(event.pointerId);
	}

	function endTouchInteraction(event: PointerEvent) {
		if (event.pointerType !== 'touch') return;

		const target = event.currentTarget as HTMLDivElement | null;
		if (target?.hasPointerCapture(event.pointerId)) {
			target.releasePointerCapture(event.pointerId);
		}

		isTouchActive = false;
		resetTilt();
	}

	function handlePointerLeave(event: PointerEvent) {
		if (event.pointerType === 'touch') return;
		resetTilt();
	}

	function resetTilt() {
		tiltX = 0;
		tiltY = 0;
		glowX = 50;
		glowY = 50;
		floatInnerShiftX = 0;
		floatInnerShiftY = 0;
	}

	function preventNativeImageActions(event: Event) {
		event.preventDefault();
	}

	function handleBackgroundPointerMove(event: PointerEvent) {
		if (!canReactBackground || event.pointerType !== 'mouse') return;

		const target = event.currentTarget as HTMLElement | null;
		if (!target) return;

		const rect = target.getBoundingClientRect();
		const x = ((event.clientX - rect.left) / rect.width) * 100;
		const y = ((event.clientY - rect.top) / rect.height) * 100;

		bgX = x;
		bgY = y;
		isBackgroundHovering = true;
	}

	function handleBackgroundPointerLeave() {
		isBackgroundHovering = false;
	}

		onMount(() => {
			const hoverMedia = window.matchMedia('(hover: hover) and (pointer: fine)');
			const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
			const loaderOverlay = document.querySelector<HTMLDivElement>('.boot-loader');
			const loaderOrb = loaderOverlay?.querySelector<HTMLDivElement>('.loader-orb') ?? null;
			let introTimeline: gsap.core.Timeline | null = null;
			let loaderLoopTimeline: gsap.core.Timeline | null = null;
			let loaderExitTimeline: gsap.core.Timeline | null = null;
			const splitInstances: Array<{ revert: () => void }> = [];
			let isUnmounted = false;

		const hideLoader = (animate: boolean) => {
			if (!loaderOverlay) {
				showLoader = false;
				return;
			}

			loaderExitTimeline?.kill();
			if (loaderOrb) {
				gsap.killTweensOf(loaderOrb);
			}
			gsap.killTweensOf(loaderOverlay);

			if (!animate) {
				if (loaderOrb) {
					gsap.set(loaderOrb, { clearProps: 'transform,opacity' });
				}
				gsap.set(loaderOverlay, { autoAlpha: 0 });
				showLoader = false;
				return;
			}

			loaderExitTimeline = gsap.timeline({
				onComplete: () => {
					showLoader = false;
				}
			});

			if (loaderOrb) {
				loaderExitTimeline.to(loaderOrb, {
					y: '110vh',
					rotation: 15,
					scale: 0.9,
					duration: 0.7,
					ease: 'power3.in'
				});
			}

			loaderExitTimeline.to(
				loaderOverlay,
				{
					autoAlpha: 0,
					duration: 0.38,
					ease: 'power2.out'
				},
				loaderOrb ? '-=0.18' : 0
			);
		};

		const createIntroTimeline = (splitWords: Element[] | null) => {
			introTimeline?.kill();
			introTimeline = gsap.timeline({ defaults: { duration: 0.85, ease: 'power2.out' }, paused: true });

			introTimeline.from('[data-reveal="logo"]', { autoAlpha: 0, y: 20, scale: 0.97 });
			introTimeline.from('[data-reveal="kicker"]', { autoAlpha: 0, y: 16 }, '-=0.5');

			if (splitWords?.length) {
				introTimeline.from(
					splitWords,
					{
						rotationX: -100,
						transformOrigin: '50% 50% -50px',
						opacity: 0,
						duration: 0.8,
						ease: 'power3.out',
						stagger: 0.08
					},
					'-=0.45'
				);
			} else {
				introTimeline.from('[data-reveal="headline-main"]', { autoAlpha: 0, y: 24 }, '-=0.45');
			}

			introTimeline.from('[data-reveal="card"]', { autoAlpha: 0, y: 28, scale: 0.95 }, '-=0.45');
			introTimeline.from('[data-reveal="social"]', { autoAlpha: 0, y: 12 }, '-=0.08');
			introTimeline.from('[data-reveal="soon"]', { autoAlpha: 0, y: 16 }, '-=0.12');
			return introTimeline;
		};

		const updateTiltAvailability = () => {
			canTilt = hoverMedia.matches;
			canReactBackground = hoverMedia.matches;
			if (!canReactBackground) {
				isBackgroundHovering = false;
			}
			if (!canTilt && !isTouchActive) resetTilt();
		};

		updateTiltAvailability();
		hoverMedia.addEventListener('change', updateTiltAvailability);

		if (loaderOverlay) {
			gsap.set(loaderOverlay, { autoAlpha: 1 });
		}

		if (reduceMotion.matches) {
			hideLoader(false);
			return () => {
				hoverMedia.removeEventListener('change', updateTiltAvailability);
			};
		}

		if (loaderOverlay) {
			if (loaderOrb) {
				gsap.set(loaderOrb, { scale: 0.96 });
			}

			loaderLoopTimeline = gsap.timeline({ repeat: -1, defaults: { ease: 'sine.inOut' } });
			loaderLoopTimeline
				.to(loaderOverlay.querySelector('.loader-ring'), { rotation: 360, duration: 2.4, ease: 'none' }, 0)
				.to(loaderOrb, { scale: 1.06, duration: 0.72, yoyo: true, repeat: 1 }, 0)
				.to(
					loaderOverlay.querySelector('.loader-core'),
					{ scale: 1.18, opacity: 0.98, duration: 0.72, yoyo: true, repeat: 1 },
					0
				)
				.to(loaderOverlay.querySelector('.loader-sheen'), { xPercent: 120, opacity: 0.72, duration: 1.1 }, 0)
				.set(loaderOverlay.querySelector('.loader-sheen'), { xPercent: -130, opacity: 0.1 }, 1.1);
		}

		void import('gsap/SplitText')
			.then(({ SplitText }) => {
				if (isUnmounted) return;

				gsap.registerPlugin(SplitText);

				const headlineMain = document.querySelector<HTMLElement>('.headline-main');
				if (headlineMain) {
					const splitHeadline = new SplitText(headlineMain, { type: 'words' });
					splitInstances.push(splitHeadline);
					createIntroTimeline(splitHeadline.words);
				} else {
					createIntroTimeline(null);
				}

				loaderLoopTimeline?.kill();
				hideLoader(true);
				introTimeline?.play(0);
			})
			.catch(() => {
				if (isUnmounted) return;

				createIntroTimeline(null);
				loaderLoopTimeline?.kill();
				hideLoader(true);
				introTimeline?.play(0);
			});

		return () => {
			isUnmounted = true;
			introTimeline?.kill();
			loaderLoopTimeline?.kill();
			loaderExitTimeline?.kill();
			for (const splitInstance of splitInstances) {
				splitInstance.revert();
			}
			if (loaderOverlay) {
				gsap.killTweensOf(loaderOverlay);
				gsap.killTweensOf(loaderOverlay.querySelector('.loader-orb'));
				gsap.killTweensOf(loaderOverlay.querySelector('.loader-ring'));
				gsap.killTweensOf(loaderOverlay.querySelector('.loader-core'));
				gsap.killTweensOf(loaderOverlay.querySelector('.loader-sheen'));
			}
			hoverMedia.removeEventListener('change', updateTiltAvailability);
		};
	});
</script>

{#if showLoader}
	<div class="boot-loader" aria-hidden="true">
		<div class="loader-orb" role="presentation">
			<div class="loader-ring"></div>
			<div class="loader-core"></div>
			<div class="loader-sheen"></div>
		</div>
	</div>
{/if}

<main
	class="landing"
	onpointermove={handleBackgroundPointerMove}
	onpointerleave={handleBackgroundPointerLeave}
	style={`--bg-x:${bgX}%; --bg-y:${bgY}%; --bg-reactive-opacity:${canReactBackground && isBackgroundHovering
		? 1
		: 0};`}
>
	<section class="hero" aria-labelledby="coming-soon-title">
		<div class="logo-wrap">
			<enhanced:img
				data-reveal="logo"
				class="logo"
				src={logoImage}
				alt="Keroyokan logo"
				loading="eager"
				fetchpriority="high"
				decoding="async"
				sizes="(max-width: 42rem) 4rem, 5rem"
			/>
		</div>

		<p class="kicker" data-reveal="kicker">Phase -1</p>
		<h1 id="coming-soon-title">
			<span class="headline-main" data-reveal="headline-main">all realities, in one deck</span>
			<span class="headline-secondary" data-reveal="soon">soon</span>
		</h1>

		<div
			data-reveal="card"
			class="card-wrap"
			class:tilt-enabled={canTilt}
			class:is-touch-active={isTouchActive}
			role="group"
			aria-label="Interactive launch card stage"
			oncontextmenu={preventNativeImageActions}
			ondragstart={preventNativeImageActions}
			onpointerdown={handlePointerDown}
			onpointermove={handlePointerMove}
			onpointerup={endTouchInteraction}
			onpointercancel={endTouchInteraction}
			onpointerleave={handlePointerLeave}
			style={`--glow-x:${glowX}%; --glow-y:${glowY}%;`}
		>
			<div class="card-tilt-stage" style={`--tilt-x:${tiltX}deg; --tilt-y:${tiltY}deg;`}>
				<div aria-hidden="true" class="floating-logo-layer">
					<div
						class="floating-logo-inner"
						style={`--float-inner-shift-x:${floatInnerShiftX}px; --float-inner-shift-y:${floatInnerShiftY}px;`}
					>
						<div class="floating-logo-shell">
							<enhanced:img
								class="floating-logo floating-logo-base"
								src={logoImage}
								alt=""
								loading="lazy"
								decoding="async"
								sizes="(max-width: 42rem) 12.1rem, 16.5rem"
								draggable="false"
							/>
							<img
								class="floating-logo floating-logo-holo"
								src={logoImageRaw}
								alt=""
								aria-hidden="true"
								loading="lazy"
								decoding="async"
								draggable="false"
							/>
							<img
								class="floating-logo floating-logo-glint"
								src={logoImageRaw}
								alt=""
								aria-hidden="true"
								loading="lazy"
								decoding="async"
								draggable="false"
							/>
						</div>
					</div>
				</div>
				<div
					class="tilt-card"
					class:is-active={isTouchActive}
					role="img"
					aria-label="Interactive launch artwork preview"
				>
					<enhanced:img
						class="cover image-fade"
						class:is-loaded={cardLoaded}
						src={cardImage}
						alt="Decorative card artwork previewing the Keroyokan launch"
						loading="lazy"
						decoding="async"
						sizes="(max-width: 42rem) 11.4rem, 12.2rem"
						draggable="false"
						onload={() => {
							cardLoaded = true;
						}}
					/>
					<img aria-hidden="true" class="cover holo-dup" src={cardImageUrl} alt="" draggable="false" />
					<img aria-hidden="true" class="cover holo-dup-2" src={cardImageUrl} alt="" draggable="false" />
					<div aria-hidden="true" class="glow"></div>
					<div aria-hidden="true" class="holo"></div>
				</div>
			</div>
		</div>

		<a
			data-reveal="social"
			class="social-link"
			href="https://instagram.com/keroyokanmultisemesta"
			target="_blank"
			rel="noreferrer noopener"
			aria-label="Follow Keroyokan Multisemesta on Instagram"
		>
			<Instagram size={15} strokeWidth={1.8} />
			<span>keroyokanmultisemesta</span>
		</a>
	</section>
</main>

<style>
	:global(html) {
		color-scheme: dark;
	}

	:global(body) {
		background:
			radial-gradient(circle at 48% 118%, rgba(206 110 0 / 0.16), transparent 30%),
			radial-gradient(circle, rgba(184 199 229 / 0.03) 0.8px, transparent 1px),
			linear-gradient(165deg, #000000 0%, #140A18 48%, #04070C 100%);
		background-size: auto, 20px 20px, auto;
		background-position: center, 0 0, center;
	}

	.landing {
		min-height: 100svh;
		position: relative;
		overflow: hidden;
		display: grid;
		place-items: center;
		padding: clamp(1rem, 2.1vw, 1.8rem);
		color: #ecf2ff;
		font-family: 'Geist', sans-serif;
	}

	.landing::before {
		content: '';
		position: absolute;
		inset: -12%;
		z-index: 0;
		pointer-events: none;
		opacity: calc(var(--bg-reactive-opacity, 0) * 0.5);
		background-image: radial-gradient(circle, rgba(212, 227, 255, 0.42) 1px, transparent 1px);
		background-size: 20px 20px;
		mask-image: radial-gradient(
			13rem circle at var(--bg-x, 50%) var(--bg-y, 50%),
			rgba(0, 0, 0, 1) 0%,
			rgba(0, 0, 0, 0.8) 45%,
			transparent 85%
		);
		-webkit-mask-image: radial-gradient(
			13rem circle at var(--bg-x, 50%) var(--bg-y, 50%),
			rgba(0, 0, 0, 1) 0%,
			rgba(0, 0, 0, 0.8) 45%,
			transparent 85%
		);
		transition: opacity 180ms ease-out;
	}

	.boot-loader {
		position: fixed;
		inset: 0;
		z-index: 40;
		display: grid;
		place-items: center;
		overflow: hidden;
		pointer-events: none;
		background-color: #03070f;
		background:
			radial-gradient(circle at 50% 56%, rgba(18 14 44 / 0.24), transparent 100%),
			linear-gradient(165deg, #040310 0%, #03070f 100%);
	}

	.loader-orb {
		position: relative;
		width: 5.2rem;
		height: 5.2rem;
		display: grid;
		place-items: center;
		will-change: transform, opacity;
	}

	.loader-ring,
	.loader-core,
	.loader-sheen {
		position: absolute;
		border-radius: 999px;
	}

	.loader-ring {
		inset: 0;
		border: 1px solid rgba(255, 216, 124, 0.56);
		box-shadow:
			0 0 0.24rem rgba(255, 214, 120, 0.4),
			0 0 1.5rem rgba(255, 186, 62, 0.36);
	}

	.loader-core {
		inset: 1.35rem;
		background: radial-gradient(circle at 35% 30%, rgba(255, 248, 214, 0.98), rgba(255, 193, 68, 0.64));
		box-shadow: 0 0 1.75rem rgba(255, 193, 77, 0.5);
	}

	.loader-sheen {
		inset: 0.24rem;
		opacity: 0;
		background: linear-gradient(110deg, transparent 0%, rgba(255, 244, 204, 0.72) 48%, transparent 0%);
		transform: translateX(-130%);
	}

	.hero {
		--floating-logo-unit: clamp(2rem, 2.2vw, 1.452rem);
		--floating-logo-size: clamp(8rem, calc(var(--floating-logo-unit) * 16), 12rem);
		--floating-logo-top: clamp(3.5rem, calc(6rem - var(--floating-logo-unit)), 8rem);
		width: min(30rem, 100%);
		position: relative;
		z-index: 1;
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

	.logo-wrap {
		will-change: opacity, transform;
	}

	.logo :global(img) {
		width: 100%;
		height: auto;
		display: block;
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

	.headline-main :global(.word) {
		display: inline-block;
		transform-style: preserve-3d;
		will-change: transform, opacity;
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
		position: relative;
		--floating-logo-active: 0;
		--floating-logo-scale: 1;
		--holo2-opacity: 0.04;
		--holo2-active-opacity: 0.8;
		--holo2-hue: -44deg;
		--holo2-sat: 2;
	}

	.card-tilt-stage {
		position: relative;
		transform-style: preserve-3d;
		transform: rotateX(var(--tilt-x, 0deg)) rotateY(var(--tilt-y, 0deg));
		transition: transform 220ms ease-out;
		will-change: transform;
	}

	.floating-logo-layer {
		position: absolute;
		left: 50%;
		top: var(--floating-logo-top);
		z-index: 3;
		pointer-events: none;
		transform: translate3d(-50%, 0, 12px);
	}

	.floating-logo-inner {
		transform: translate3d(var(--float-inner-shift-x, 0px), var(--float-inner-shift-y, 0px), 0);
		transition: transform 260ms ease-out;
		will-change: transform;
	}

	.floating-logo-shell {
		width: var(--floating-logo-size);
		position: relative;
		isolation: isolate;
		border-radius: 1rem;
		transform: scale(var(--floating-logo-scale));
		transition: transform 260ms ease-out, filter 260ms ease-out;
		filter: drop-shadow(0 7px 14px rgba(4, 10, 22, 0.5));
		will-change: transform, filter;
	}

	.floating-logo {
		position: absolute;
		inset: 0;
		width: 100%;
		height: auto;
		display: block;
		max-width: none;
		pointer-events: none;
	}

	.floating-logo-base {
		position: relative;
		z-index: 1;
		opacity: calc(0.94 + (var(--floating-logo-active) * 0.04));
		filter:
			drop-shadow(0 0 1.1rem rgba(106, 175, 255, 0.2))
			hue-rotate(calc(var(--floating-logo-active) * -10deg))
			saturate(calc(1 + (var(--floating-logo-active) * 0.42)));
		transition: filter 260ms ease-out, opacity 260ms ease-out;
		will-change: filter, opacity;
	}

	.floating-logo-holo {
		z-index: 2;
		opacity: calc(0.06 + (var(--floating-logo-active) * 0.24));
		mix-blend-mode: color-dodge;
		filter: hue-rotate(-140deg) saturate(2.2);
		-webkit-mask-image: radial-gradient(
			circle at var(--glow-x, 50%) var(--glow-y, 50%),
			rgba(0, 0, 0, 0.95) 0%,
			rgba(0, 0, 0, 0.7) 25%,
			transparent 72%
		);
		mask-image: radial-gradient(
			circle at var(--glow-x, 50%) var(--glow-y, 50%),
			rgba(0, 0, 0, 0.95) 0%,
			rgba(0, 0, 0, 0.7) 25%,
			transparent 72%
		);
		transition: opacity 260ms ease-out, filter 260ms ease-out;
	}

	.floating-logo-glint {
		z-index: 3;
		opacity: calc(0.04 + (var(--floating-logo-active) * 0.14));
		mix-blend-mode: screen;
		filter: saturate(1.5) brightness(1.08);
		-webkit-mask-image: linear-gradient(
			108deg,
			rgba(0, 0, 0, 0) 18%,
			rgba(0, 0, 0, 0.95) 42%,
			rgba(0, 0, 0, 0.88) 50%,
			rgba(0, 0, 0, 0) 74%
		);
		mask-image: linear-gradient(
			108deg,
			rgba(0, 0, 0, 0) 18%,
			rgba(0, 0, 0, 0.95) 42%,
			rgba(0, 0, 0, 0.88) 50%,
			rgba(0, 0, 0, 0) 74%
		);
		-webkit-mask-size: 230% 100%;
		mask-size: 230% 100%;
		-webkit-mask-position: -145% 0;
		mask-position: -145% 0;
		animation: edge-glint-sweep 7.6s ease-in-out infinite;
		transition: opacity 260ms ease-out;
		will-change: opacity, mask-position;
	}

	.card-wrap.is-touch-active {
		--floating-logo-active: 1;
		--floating-logo-scale: 1.045;
	}

	.card-wrap.is-touch-active .floating-logo-shell {
		filter: drop-shadow(0 10px 18px rgba(5, 11, 24, 0.44)) drop-shadow(0 0 1rem rgba(94, 149, 255, 0.2));
	}

	.card-wrap.is-touch-active .floating-logo-glint {
		animation-play-state: paused;
	}

	.social-link {
		margin-top: 0.9rem;
		display: inline-flex;
		align-items: center;
		gap: 0.42rem;
		color: rgba(219, 229, 255, 0.82);
		text-decoration: none;
		font-family: 'Geist Mono', monospace;
		font-size: 0.72rem;
		letter-spacing: 0.02em;
		padding: 0.32rem 0.55rem;
		border-radius: 999px;
		outline: 1px solid rgba(111, 146, 203, 0.32);
		background: rgba(8, 14, 27, 0.24);
		transition: color 180ms ease-out, background-color 180ms ease-out, outline-color 180ms ease-out;
	}

	.social-link:hover {
		color: rgba(236, 243, 255, 0.96);
		background: rgba(14, 24, 42, 0.44);
		outline-color: rgba(140, 180, 240, 0.52);
	}

	.social-link:focus-visible {
		outline: 1px solid rgba(180, 211, 255, 0.92);
		outline-offset: 2px;
	}

	.tilt-card {
		position: relative;
		border-radius: 0.6rem;
		overflow: hidden;
		transform-style: preserve-3d;
		transform: translateZ(0) scale(var(--card-scale, 1));
		transition: transform 220ms ease-out, box-shadow 220ms ease-out;
		touch-action: none;
		outline: 0.4px solid rgba(117, 178, 235, 0.34);
		box-shadow:
			0 16px 28px rgba(4, 8, 20, 0.52),
			0 4px 10px rgba(17, 42, 84, 0.35);
	}

	.tilt-card::before {
		content: '';
		position: absolute;
		inset: 0;
		padding: 1px;
		border-radius: inherit;
		pointer-events: none;
		opacity: 0.22;
		background:
			linear-gradient(
				105deg,
				rgba(255, 255, 255, 0) 18%,
				rgba(191, 230, 255, 0.45) 42%,
				rgba(255, 211, 154, 0.33) 50%,
				rgba(255, 255, 255, 0) 74%
			)
			0 0 / 230% 100% no-repeat;
		mix-blend-mode: screen;
		-webkit-mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
		mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
		-webkit-mask-composite: xor;
		mask-composite: exclude;
		animation: edge-glint-sweep 7.6s ease-in-out infinite;
		will-change: background-position, opacity;
	}

	.tilt-card.is-active {
		--card-scale: 1.025;
		box-shadow:
			0 20px 34px rgba(6, 11, 26, 0.62),
			0 7px 16px rgba(30, 52, 103, 0.36);
	}

	.tilt-card.is-active::before {
		opacity: 0.1;
		animation-play-state: paused;
	}

	.tilt-card.is-active .holo-dup {
		opacity: 1;
		filter: hue-rotate(-150deg) saturate(2.8);
	}

	.tilt-card.is-active .holo-dup-2 {
		opacity: var(--holo2-active-opacity);
		filter: hue-rotate(var(--holo2-hue)) saturate(calc(var(--holo2-sat) + 0.45)) brightness(1.07);
	}

	@media (hover: hover) and (pointer: fine) {
		.card-wrap:hover {
			--floating-logo-active: 1;
			--floating-logo-scale: 1.045;
		}

		.card-wrap:hover .floating-logo-shell {
			filter: drop-shadow(0 10px 18px rgba(5, 11, 24, 0.44)) drop-shadow(0 0 1rem rgba(94, 149, 255, 0.2));
		}

		.card-wrap:hover .floating-logo-glint {
			animation-play-state: paused;
		}

		.card-wrap:hover .tilt-card {
			--card-scale: 1.045;
			box-shadow:
				0 20px 34px rgba(6, 11, 26, 0.62),
				0 7px 16px rgba(30, 52, 103, 0.36);
		}

		.card-wrap:hover .tilt-card::before {
			opacity: 0.1;
			animation-play-state: paused;
		}

		.card-wrap:hover .tilt-card .holo-dup {
			opacity: 1;
			filter: hue-rotate(-150deg) saturate(2.8);
		}

		.card-wrap:hover .tilt-card .holo-dup-2 {
			opacity: var(--holo2-active-opacity);
			filter: hue-rotate(var(--holo2-hue)) saturate(calc(var(--holo2-sat) + 0.45)) brightness(1.07);
		}

		.tilt-enabled {
			cursor: grab;
		}

		.tilt-enabled:active {
			cursor: grabbing;
		}
	}

	.card-wrap,
	.card-wrap :global(img),
	.floating-logo,
	.cover,
	.holo-dup,
	.holo-dup-2 {
		-webkit-touch-callout: none;
		-webkit-user-drag: none;
		user-select: none;
	}

	.cover {
		display: block;
		width: 100%;
		height: auto;
		object-fit: cover;
	}

	.cover :global(img) {
		display: block;
		width: 100%;
		height: auto;
		object-fit: cover;
	}

	.image-fade {
		opacity: 0;
		transition: opacity 420ms ease-out;
	}

	.image-fade :global(img) {
		filter: blur(0.5rem);
		transition: filter 420ms ease-out;
	}

	.image-fade.is-loaded {
		opacity: 1;
	}

	.image-fade.is-loaded :global(img) {
		filter: blur(0);
	}

	.holo-dup {
		position: absolute;
		inset: 0;
		height: 100%;
		opacity: 0;
		mix-blend-mode: hard-light;
		filter: hue-rotate(-150deg) saturate(200);
		transition: opacity 240ms ease-out, filter 240ms ease-out;
		will-change: opacity, filter;
		-webkit-mask-image: radial-gradient(
			circle at var(--glow-x, 50%) var(--glow-y, 50%),
			rgba(0, 0, 0, 0.95) 0%,
			rgba(0, 0, 0, 0.72) 24%,
			transparent 100%
		);
		mask-image: radial-gradient(
			circle at var(--glow-x, 50%) var(--glow-y, 50%),
			rgba(0, 0, 0, 0.95) 0%,
			rgba(0, 0, 0, 0.72) 24%,
			transparent 100%
		);
		pointer-events: none;
	}

	.holo-dup-2 {
		position: absolute;
		inset: 0;
		height: 100%;
		opacity: var(--holo2-opacity);
		mix-blend-mode: screen;
		filter: hue-rotate(var(--holo2-hue)) saturate(200) brightness(1.6) contrast(1.8);
		transform: translate3d(-0.7%, 0.5%, 0);
		transition:
			opacity 280ms ease-out,
			filter 280ms ease-out,
			transform 280ms ease-out;
		will-change: opacity, filter, transform;
		-webkit-mask-image: radial-gradient(
			circle at calc(var(--glow-x, 50%) + 6%) calc(var(--glow-y, 50%) - 5%),
			rgba(0, 0, 0, 0.88) 0%,
			rgba(0, 0, 0, 0.56) 20%,
			transparent 100%
		);
		mask-image: radial-gradient(
			circle at calc(var(--glow-x, 50%) + 6%) calc(var(--glow-y, 50%) - 5%),
			rgba(0, 0, 0, 0.88) 0%,
			rgba(0, 0, 0, 0.56) 20%,
			transparent 100%
		);
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
			linear-gradient(125deg, rgba(255, 252, 242, 0.05), rgba(255, 255, 255, 0) 40%);
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
		opacity: 0.40;
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

	@keyframes edge-glint-sweep {
		0%,
		100% {
			background-position: -145% 0;
			-webkit-mask-position: -145% 0;
			mask-position: -145% 0;
		}

		50% {
			background-position: 145% 0;
			-webkit-mask-position: 145% 0;
			mask-position: 145% 0;
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

	@media (hover: none), (pointer: coarse) {
		.landing::before {
			display: none;
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.boot-loader {
			transition: none;
		}

		.image-fade {
			transition: none;
		}

		.image-fade :global(img) {
			transition: none;
		}

		.tilt-card {
			transition: none;
			transform: none;
		}

		.card-tilt-stage {
			transition: none;
			transform: none;
		}

		.holo-dup {
			transition: none;
		}

		.holo-dup-2 {
			transition: none;
		}

		.holo {
			opacity: 0.2;
		}

		.holo::after {
			animation: none;
			opacity: 0.08;
			transform: translate3d(0, 0, 0) rotate(-8deg);
		}

		.tilt-card::before {
			animation: none;
			opacity: 0;
		}

		.logo-wrap {
			transition: none;
			transform: none;
		}

		.floating-logo-inner {
			transition: none;
			transform: none;
		}

		.floating-logo,
		.floating-logo-shell {
			transition: none;
		}

		.floating-logo-glint {
			animation: none;
			opacity: 0.08;
		}
	}
</style>
