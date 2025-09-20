<!-- src/routes/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import DiscordStatus from '$lib/DiscordStatus.svelte';

	// NOTE: showing a fixed "owner" timezone rather than the visitor's local time.
	const OWNER_TIMEZONE = 'Europe/London';

	const timeFmt = new Intl.DateTimeFormat('en-US', {
		hour: 'numeric',
		minute: '2-digit',
		second: '2-digit',
		hour12: true,
		timeZone: OWNER_TIMEZONE
	});

	let time = timeFmt.format(new Date());
	let interval: number;

	function tick() {
		const now = new Date();
		time = timeFmt.format(now);
	}

	onMount(() => {
		interval = window.setInterval(tick, 1000);
		tick();
		return () => clearInterval(interval);
	});

	function hookSeeMore() {
		// attach click handlers after DOM ready
		const btn = document.querySelector('.see-more');
		if (btn)
			btn.addEventListener('click', (e) => {
				const t = (e.currentTarget as HTMLElement).getAttribute('data-target');
				if (t) {
					const el = document.querySelector(t);
					if (el) el.scrollIntoView({ behavior: 'smooth' });
				}
			});
	}

	onMount(() => {
		hookSeeMore();
	});
</script>

<section id="home" class="page-hero py-20" role="presentation">
	<div class="card">
		<div class="hero-row">
			<div class="hero-left">
				<div class="name-wrap mb-4">
					<div class="slashes" aria-hidden="true">//</div>
					<h1 class="inline text-6xl font-bold">jay</h1>
				</div>
				<p class="hero-tagline">
					insert text here but it has to be short because thats how its styled
				</p>

				<div class="hero-cta">
					<button class="discover" data-target="#work">
						<div class="tab">/</div>
						<div class="label">Discover more ↓</div>
					</button>
				</div>

				<div class="status-wrap">
					<DiscordStatus userId="479395702401662988" clock={time} />
				</div>
			</div>
			<div class="hero-right">
				<div class="avatar">
					<!-- placeholder avatar image: replace with your image -->
					<div
						style="width:100%;height:100%;background:linear-gradient(135deg,#e2dcd6,#cfc9c3);"
					></div>
				</div>
				<div class="about-card">
					<h3>About me</h3>
					<p>
						this one will be longer er than the one over there because this one will be more
						detailed with hover reactive shit
					</p>
				</div>
			</div>
		</div>
	</div>
</section>

<section id="about" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">About</h2>
		<p class="mb-6 text-gray-300">glazebox goes here maybe? and some other about shite</p>
	</div>
</section>

<section id="work" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">Work</h2>
		<p class="mb-6 text-gray-300">github? or maybe a blog..</p>
	</div>
</section>

<section id="contact" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">Contact</h2>
		<p class="mb-6 text-gray-300">Email: hello@jay.ax</p>
	</div>
</section>
