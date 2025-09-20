<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	export let userId = '479395702401662988';
	export let clock: string | null = null;

	interface LanyardAssets {
		large_image?: string;
		small_image?: string;
	}

	interface LanyardActivity {
		type: number;
		name: string;
		details?: string;
		assets?: LanyardAssets;
		application_id?: string;
	}

	interface LanyardDiscordUser {
		id: string;
		username: string;
		avatar: string | null;
		discriminator: string;
	}

	interface LanyardSpotify {
		album_art_url?: string;
	}

	interface LanyardData {
		discord_status?: string;
		discord_user?: LanyardDiscordUser;
		activities?: LanyardActivity[];
		spotify?: LanyardSpotify;
	}

	interface LanyardApiResponse {
		data?: LanyardData;
	}

	let status: string | null = null;
	let activity: string | null = null;
	let avatarUrl: string | null = null;
	let statusImageUrl: string | null = null;
	let handle = `@${userId}`;
	let intervalId: number | null = null;

	function formatStatus(value: string | null): string {
		if (!value) return 'Offline';
		return value.charAt(0).toUpperCase() + value.slice(1);
	}

	function buildAvatarUrl(discordUser: LanyardDiscordUser | undefined): string | null {
		if (discordUser?.avatar) {
			const format = discordUser.avatar.startsWith('a_') ? 'gif' : 'png';
			return `https://cdn.discordapp.com/avatars/${discordUser.id ?? userId}/${discordUser.avatar}.${format}?size=128`;
		}
		const discriminator = discordUser?.discriminator;
		const fallbackIndex = discriminator && discriminator !== '0' ? Number(discriminator) % 5 : 0;
		return `https://cdn.discordapp.com/embed/avatars/${fallbackIndex}.png`;
	}

	function resolveActivityImage(
		presence: LanyardActivity | undefined,
		data: LanyardData | undefined
	): string | null {
		if (!presence) return null;

		if (presence.name === 'Spotify' && data?.spotify?.album_art_url) {
			return data.spotify.album_art_url;
		}

		const assets = presence.assets;
		if (!assets) return null;

		const candidate = assets.large_image || assets.small_image;
		if (!candidate) return null;

		if (candidate.startsWith('mp:')) {
			return `https://media.discordapp.net/${candidate.slice(3)}`;
		}
		if (candidate.startsWith('spotify:')) {
			return `https://i.scdn.co/image/${candidate.slice(8)}`;
		}
		if (candidate.startsWith('http')) {
			return candidate;
		}
		if (presence.application_id) {
			return `https://cdn.discordapp.com/app-assets/${presence.application_id}/${candidate}.png`;
		}
		return null;
	}

	async function fetchLanyard() {
		try {
			const res = await fetch(`https://api.lanyard.rest/v1/users/${userId}`);
			if (!res.ok) return;
			const json = (await res.json()) as LanyardApiResponse;
			const data = json.data;
			if (!data) return;
			status = data.discord_status ?? null;
			const activities = data.activities ?? [];
			const presence = activities.find((activity) => activity.type === 0) ?? activities[0];
			activity = presence
				? `${presence.name}${presence.details ? ` — ${presence.details}` : ''}`
				: null;
			avatarUrl = buildAvatarUrl(data.discord_user) ?? avatarUrl;
			statusImageUrl = resolveActivityImage(presence, data) ?? avatarUrl;
			handle = data.discord_user?.username ? `@${data.discord_user.username}` : handle;
		} catch {
			status = null;
			activity = null;
			statusImageUrl = avatarUrl;
		}
	}

	onMount(() => {
		fetchLanyard();
		intervalId = window.setInterval(fetchLanyard, 30000);
	});

	onDestroy(() => {
		if (intervalId) clearInterval(intervalId);
	});
</script>

<div class="ds" aria-live="polite">
	<div class="thumb">
		{#if statusImageUrl}
			<img src={statusImageUrl} alt={activity ?? 'Discord avatar'} loading="lazy" />
		{:else}
			<div class="thumb-fallback" aria-hidden="true"></div>
		{/if}
	</div>
	<div class="details">
		<div class="handle">{handle}</div>
		{#if activity}
			<div class="line activity">{activity}</div>
		{:else}
			<div class="line">{formatStatus(status)}</div>
		{/if}
		{#if clock}
			<div class="clock">{clock}</div>
		{/if}
	</div>
</div>

<style>
	.ds {
		display: flex;
		gap: 22px;
		align-items: center;
		font-family: 'JetBrains Mono', ui-monospace, SFMono-Regular, Menlo, monospace;
		color: #5c4335;
		padding: 0;
		border: none;
		background: none;
		box-shadow: none;
		min-width: 0;
	}
	.thumb {
		width: 96px;
		height: 96px;
		border-radius: 26px;
		overflow: hidden;
		box-shadow: 0 10px 28px rgba(30, 41, 59, 0.22);
		background: linear-gradient(135deg, rgba(148, 163, 184, 0.25), rgba(148, 163, 184, 0.08));
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}
	.thumb img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		display: block;
	}
	.thumb-fallback {
		width: 60%;
		height: 60%;
		border-radius: 24px;
		background: rgba(2, 6, 23, 0.12);
	}
	.details {
		display: flex;
		flex-direction: column;
		gap: 6px;
		flex: 1;
		min-width: 0;
	}
	.handle {
		font-size: 22px;
		font-weight: 700;
		letter-spacing: 0.03em;
		color: #44271c;
	}
	.line {
		font-size: 16px;
		letter-spacing: 0.01em;
		color: rgba(92, 67, 53, 0.82);
		line-height: 1.38;
	}
	.line.activity {
		color: rgba(92, 67, 53, 0.75);
		max-width: 280px;
		word-break: break-word;
	}
	.clock {
		font-size: 0.95rem;
		font-variant-numeric: tabular-nums;
		color: rgba(76, 56, 44, 0.7);
		text-transform: uppercase;
		letter-spacing: 0.12em;
	}

	@media (max-width: 600px) {
		.ds {
			gap: 16px;
		}
		.thumb {
			width: 72px;
			height: 72px;
			border-radius: 18px;
		}
		.handle {
			font-size: 20px;
		}
		.line {
			font-size: 0.95rem;
		}
		.clock {
			font-size: 0.85rem;
			letter-spacing: 0.14em;
		}
	}
</style>
