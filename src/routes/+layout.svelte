<script lang="ts">
	import '../app.css';
	import { onMount, onDestroy } from 'svelte';
	let CursorComponent: typeof import('$lib/Cursor.svelte').default | null = null;
	onMount(async () => {
		const m = await import('$lib/Cursor.svelte');
		CursorComponent = m.default;
	});

	// theme handling: follow system preference by default, allow manual toggle fallbacks
	let _themeBtn: HTMLElement | null = null;
	let _themeHandler: EventListener | null = null;
	let _prefersMq: MediaQueryList | null = null;
	let _prefersHandler: ((event: MediaQueryListEvent) => void) | null = null;
	let _manualTheme: 'light' | 'dark' | null = null;

	function applyTheme(theme: 'light' | 'dark') {
		document.documentElement.setAttribute('data-theme', theme);
	}

	function systemTheme(): 'light' | 'dark' {
		return _prefersMq && _prefersMq.matches ? 'dark' : 'light';
	}

	function updateToggleState() {
		if (!_themeBtn) return;
		const mode = _manualTheme ?? 'system';
		const titles: Record<'system' | 'light' | 'dark', string> = {
			system: 'Match device theme',
			light: 'Light theme active',
			dark: 'Dark theme active'
		};
		_themeBtn.setAttribute('data-mode', mode);
		_themeBtn.setAttribute('aria-label', `Toggle theme (currently ${titles[mode].toLowerCase()})`);
		_themeBtn.setAttribute('title', `${titles[mode]} — tap to toggle`);
	}

	function syncSystemTheme() {
		applyTheme(systemTheme());
		updateToggleState();
	}
	onMount(() => {
		_themeBtn = document.getElementById('theme-toggle');
		_prefersMq = window.matchMedia('(prefers-color-scheme: dark)');
		syncSystemTheme();
		_prefersHandler = (event) => {
			if (_manualTheme) return;
			applyTheme(event.matches ? 'dark' : 'light');
			updateToggleState();
		};
		_prefersMq.addEventListener('change', _prefersHandler);

		if (_themeBtn) {
			_themeHandler = () => {
				const currentTheme =
					(document.documentElement.getAttribute('data-theme') as 'light' | 'dark' | null) ||
					systemTheme();
				if (_manualTheme) {
					_manualTheme = null;
					syncSystemTheme();
					return;
				}
				const next = currentTheme === 'dark' ? 'light' : 'dark';
				_manualTheme = next;
				applyTheme(next);
				updateToggleState();
			};
			_themeBtn.addEventListener('click', _themeHandler);
			updateToggleState();
		}
	});

	// Portal the large background watermark into document.body so it remains fixed
	// relative to the viewport even if other layout nodes have transforms.
	let _wmEl: HTMLElement | null = null;
	let _scrollHandler: (() => void) | null = null;
	onMount(() => {
		try {
			// remove any previous watermark container if present
			const prev = document.querySelector('.bg-texts-svg');
			if (prev && prev.parentNode) prev.parentNode.removeChild(prev);

			const svgNS = 'http://www.w3.org/2000/svg';
			const svg = document.createElementNS(svgNS, 'svg');
			// when we append to body we want it to scroll, so use class for absolute rules
			svg.setAttribute('class', 'bg-texts-scroll');
			// minimal inline style; detailed layout handled by CSS
			svg.setAttribute('style', 'width:100%; height:100%; pointer-events:none; overflow:visible;');
			svg.setAttribute('aria-hidden', 'true');
			svg.setAttribute('width', '100%');
			svg.setAttribute('height', '100%');
			svg.setAttribute('xmlns', svgNS);

			const t1 = document.createElementNS(svgNS, 'text');
			t1.setAttribute('class', 'bg-word bg-left');
			t1.setAttribute('x', '2%');
			t1.setAttribute('y', '55%');
			t1.setAttribute(
				'style',
				'fill:none; stroke: var(--watermark-stroke); stroke-width:2.6; opacity:0.9; font-weight:800; font-family: Inter, system-ui, sans-serif; font-size:260px;'
			);
			t1.textContent = 'jay';

			const t2 = document.createElementNS(svgNS, 'text');
			t2.setAttribute('class', 'bg-word bg-right');
			t2.setAttribute('x', '75%');
			t2.setAttribute('y', '84%');
			t2.setAttribute(
				'style',
				'fill:none; stroke: var(--watermark-stroke); stroke-width:3.2; opacity:0.9; font-weight:800; font-family: Inter, system-ui, sans-serif; font-size:300px;'
			);
			t2.textContent = 'ax';

			svg.appendChild(t1);
			svg.appendChild(t2);

			document.body.appendChild(svg);
			document.documentElement.setAttribute('data-bg-mounted', 'svg-body');
			_wmEl = svg as unknown as HTMLElement;
		} catch {
			// ignore - non-critical
			_wmEl = null;
		}
		// header scroll handler: toggle data attribute to style header when scrolled
		const headerEl = document.querySelector('header.site-header') as HTMLElement | null;
		if (headerEl) headerEl.setAttribute('data-scrolled', 'false');
		_scrollHandler = () => {
			if (!headerEl) return;
			if (window.scrollY > 8) headerEl.setAttribute('data-scrolled', 'true');
			else headerEl.setAttribute('data-scrolled', 'false');
		};
		window.addEventListener('scroll', _scrollHandler as EventListener, { passive: true });
		// run once to set initial state
		_scrollHandler();
	});

	onDestroy(() => {
		if (_wmEl && _wmEl.parentNode) _wmEl.parentNode.removeChild(_wmEl);
		if (_themeBtn && _themeHandler) _themeBtn.removeEventListener('click', _themeHandler);
		if (_prefersMq && _prefersHandler) _prefersMq.removeEventListener('change', _prefersHandler);
		if (_scrollHandler) window.removeEventListener('scroll', _scrollHandler as EventListener);
	});
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<header class="site-header" data-scrolled="false">
	<div class="header-inner">
		<button
			id="theme-toggle"
			type="button"
			aria-label="Toggle theme"
			class="theme-toggle"
			title="Toggle theme"
		>
			<!-- material-like sun/moon icon (SVG) -->
			<svg
				class="theme-icon"
				width="20"
				height="20"
				viewBox="0 0 24 24"
				fill="none"
				xmlns="http://www.w3.org/2000/svg"
				aria-hidden="true"
			>
				<path
					class="sun"
					d="M12 3v2M12 19v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"
					stroke="currentColor"
					stroke-width="1.5"
					stroke-linecap="round"
					stroke-linejoin="round"
				/>
				<circle class="core" cx="12" cy="12" r="4" stroke="currentColor" stroke-width="1.5" />
			</svg>
		</button>

		<div class="nav-shell">
			<nav class="site-nav" aria-label="Primary">
				<a href="#home" class="nav-link">Home</a>
				<a href="#about" class="nav-link">About</a>
				<a href="#work" class="nav-link">Work</a>
			</nav>
		</div>
		</div>
</header>

<main class="p-6">
	<slot />
</main>

<footer class="border-t border-gray-200 p-4 text-center text-sm text-gray-500">
	© {new Date().getFullYear()} Jay. Built with frowns.
</footer>

{#if CursorComponent}
	<svelte:component this={CursorComponent} />
{/if}
