<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	let x = 0;
	let y = 0;
	let cx = 0;
	let cy = 0;
	let scale = 1;
	let hue = 210;
	let ringScale = 1;

	let raf: number;
	let audioCtx: AudioContext | null = null;

	type ExtendedWindow = Window & typeof globalThis & { webkitAudioContext?: typeof AudioContext };

	function getAudioContextConstructor(): typeof AudioContext | undefined {
		if (typeof window === 'undefined') return undefined;
		const win = window as ExtendedWindow;
		return win.AudioContext ?? win.webkitAudioContext;
	}

	function createTone(duration = 0.06, freq = 800, vol = 0.015) {
		try {
			if (typeof window === 'undefined') return;
			const Ctor = getAudioContextConstructor();
			if (!Ctor) return;
			if (!audioCtx) audioCtx = new Ctor();
			const ctx = audioCtx;
			const now = ctx.currentTime;
			const o = ctx.createOscillator();
			const g = ctx.createGain();
			o.type = 'sine';
			o.frequency.value = freq;
			g.gain.value = vol;
			o.connect(g);
			g.connect(ctx.destination);
			o.start(now);
			g.gain.exponentialRampToValueAtTime(0.0001, now + duration);
			o.stop(now + duration + 0.02);
		} catch {
			// ignore audio failures on unsupported platforms
		}
	}

	function onMove(e: MouseEvent) {
		x = e.clientX;
		y = e.clientY;
		const target = e.target instanceof Element ? e.target : null;
		if (target?.closest('a')) {
			scale = 1.6; // bigger when hovering links
			hue = 40; // warm accent on hover
		} else {
			scale = 1;
			hue = 210;
		}
	}

	function onDown() {
		createTone(0.04, 520, 0.03);
		// brief expansion animation
		ringScale = 1.4;
		setTimeout(() => (ringScale = 1), 140);
	}

	onMount(() => {
		if (typeof window === 'undefined') return;
		window.addEventListener('mousemove', onMove);
		window.addEventListener('mousedown', onDown);

		function loop() {
			cx += (x - cx) * 0.14;
			cy += (y - cy) * 0.14;
			raf = requestAnimationFrame(loop) as unknown as number;
		}

		loop();
	});

	onDestroy(() => {
		window.removeEventListener('mousemove', onMove);
		window.removeEventListener('mousedown', onDown);
		if (typeof raf !== 'undefined') cancelAnimationFrame(raf);
		if (audioCtx && audioCtx.state !== 'closed') {
			try {
				audioCtx.close();
			} catch {
				// ignore close failures
			}
		}
	});
</script>

<div
	class="cursor"
	aria-hidden="true"
	style="transform: translate3d({cx}px, {cy}px, 0) translate(-50%, -50%) scale({scale}); --h: {hue}; --rs: {ringScale};"
>
	<div class="ring"></div>
</div>

<style>
	/* keep minimal component-scoped fallback styles; main CSS in app.css */
	.cursor {
		position: fixed;
		top: 0;
		left: 0;
		width: 46px;
		height: 46px;
		border-radius: 9999px;
		pointer-events: none;
		z-index: 9999;
		transition: transform 120ms cubic-bezier(0.2, 0.9, 0.2, 1);
		transform-origin: center center;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.ring {
		width: 36px;
		height: 36px;
		border-radius: 9999px;
		border: 2px solid hsla(var(--h) 85% 55% / 0.95);
		box-shadow: 0 12px 32px rgba(2, 6, 23, 0.25);
		transform: scale(var(--rs, 1));
		transition:
			transform 140ms cubic-bezier(0.2, 0.9, 0.2, 1),
			border-color 160ms ease;
	}
</style>
