<!-- src/routes/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import DiscordStatus from '$lib/DiscordStatus.svelte';
	import Tooltip from '$lib/Tooltip.svelte';

	// NOTE: showing a fixed "owner" timezone rather than the visitor's local time.
	const OWNER_TIMEZONE = 'Europe/London';
	const BIRTHDATE = new Date('2007-04-27T00:00:00Z');
	const MS_IN_YEAR = 31_536_000_000;

	const timeFmt = new Intl.DateTimeFormat('en-US', {
		hour: 'numeric',
		minute: '2-digit',
		second: '2-digit',
		hour12: true,
		timeZone: OWNER_TIMEZONE
	});

	let time = timeFmt.format(new Date());
	let timeTicker: number;
	let age = computeAge();
	let ageTicker: number;

	function computeAge() {
		return (Date.now() - BIRTHDATE.getTime()) / MS_IN_YEAR;
	}

	$: wholeYears = Math.floor(age);
	$: preciseAge = age.toFixed(9);

	function tick() {
		const now = new Date();
		time = timeFmt.format(now);
	}

	onMount(() => {
		timeTicker = window.setInterval(tick, 1000);
		ageTicker = window.setInterval(() => {
			age = computeAge();
		}, 1000);
		age = computeAge();
		tick();
		return () => {
			if (timeTicker) window.clearInterval(timeTicker);
			if (ageTicker) window.clearInterval(ageTicker);
		};
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
					Full stack developer and car enthusiast. 
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
				<div class="avatar" aria-hidden="true">
					<!-- placeholder avatar image: replace with your image -->
				</div>
				<div class="about-card">
					<p class="about-card-label">About Me</p>
					<p class="about-card-body">
						Hey there, I'm{' '}
						<Tooltip tip="Taylor works too, but Jay is how I'm known online.">
							<span class="about-chip">Jay</span>
						</Tooltip>{' '}
						— an{' '}
						<Tooltip tip={`precisely ${preciseAge} years old`}>
							<span class="about-chip">{wholeYears}</span>
						</Tooltip>{' '}
						year-old{' '}
						<Tooltip tip="TypeScript, SvelteKit, Go, Postgres, and more.">
							<span class="about-chip">full stack developer</span>
						</Tooltip>{' '}
						anchored in the UK. Most days I'm deep in{' '}
						<Tooltip tip="From enterprise racks to homelab clusters.">
							<span class="about-chip">network infrastructure</span>
						</Tooltip>{' '}
						and building fast, resilient experiences for the web. When I'm not coding,
						I like to tinker with cars, too. I have a passion for older cars from the late 80s to mid 2000s.
					</p>
					</div>
				</div>
			</div>
		</div>
</section>

<section id="about" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">about the site</h2>
		<p class="mb-6 text-gray-300">This section is a placeholder for now.</p>
		<p class="mb-6 text-gray-500">Inspiration behind this site came from afn.im, I encourage you to check it out! I know it looks very similar right now but there are some big changes I have planned, especially down here. I have written the codebase --mostly-- from scratch with a little help from the open source community, chatbox gpt :(, and some basic googling.</p>
	</div>
</section>

<section id="work" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">???</h2>
		<p class="mb-6 text-gray-300">github? or maybe a blog..</p>
	</div>
</section>

<section id="contact" class="py-16">
	<div class="card mx-auto max-w-3xl">
		<h2 class="mb-4 text-3xl font-semibold">"Contact"</h2>
		<p class="mb-6 text-gray-300">Email: jay@lazerjay.dev, Discord: lazerjay7</p>
	</div>
</section>
