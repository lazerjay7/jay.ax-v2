<script lang="ts">
	export let tip: string;
	export let placement: 'top' | 'bottom' = 'top';

	let open = false;

	function show() {
		open = true;
	}

	function hide() {
		open = false;
	}
</script>

<span class="tooltip" data-state={open ? 'open' : 'closed'} data-placement={placement}>
	<span
		class="tooltip-trigger"
		tabindex="0"
		aria-label={tip}
		on:mouseenter={show}
		on:mouseleave={hide}
		on:focus={show}
		on:blur={hide}
		on:touchstart|preventDefault={show}
		on:touchend={hide}
	>
		<slot />
	</span>
	{#if tip}
		<span class="tooltip-bubble" role="tooltip" aria-hidden={!open}>
			{tip}
		</span>
	{/if}
</span>

<style>
	.tooltip {
		position: relative;
		display: inline-flex;
		align-items: center;
	}

	.tooltip-trigger {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		outline: none;
	}

	.tooltip-trigger:focus-visible {
		outline: 2px solid color-mix(in srgb, var(--accent) 60%, transparent 40%);
		outline-offset: 4px;
		border-radius: inherit;
	}

	.tooltip-bubble {
		position: absolute;
		left: 50%;
		pointer-events: none;
		background: color-mix(in srgb, var(--tooltip-bg, #111827) 90%, rgba(255, 255, 255, 0.1));
		color: var(--tooltip-text, #f9fafb);
		padding: 0.35rem 0.55rem;
		border-radius: 0.55rem;
		font-size: 0.72rem;
		line-height: 1.1;
		box-shadow: 0 0.75rem 2rem rgba(15, 23, 42, 0.35);
		opacity: 0;
		transform: translate3d(-50%, 6px, 0) scale(0.95);
		transition:
			opacity 120ms ease,
			transform 150ms cubic-bezier(0.22, 1, 0.36, 1);
		z-index: 40;
		white-space: nowrap;
	}

	.tooltip[data-placement='top'] .tooltip-bubble {
		bottom: calc(100% + 10px);
	}

	.tooltip[data-placement='bottom'] .tooltip-bubble {
		top: calc(100% + 10px);
	}

	.tooltip[data-state='open'] .tooltip-bubble {
		opacity: 1;
		transform: translate3d(-50%, 0, 0) scale(1);
	}

	.tooltip[data-placement='top'] .tooltip-bubble::after,
	.tooltip[data-placement='bottom'] .tooltip-bubble::after {
		content: '';
		position: absolute;
		left: 50%;
		transform: translateX(-50%);
		border: 6px solid transparent;
	}

	.tooltip[data-placement='top'] .tooltip-bubble::after {
		top: 100%;
		border-top-color: color-mix(in srgb, var(--tooltip-bg, #111827) 90%, rgba(255, 255, 255, 0.1));
	}

	.tooltip[data-placement='bottom'] .tooltip-bubble::after {
		bottom: 100%;
		border-bottom-color: color-mix(in srgb, var(--tooltip-bg, #111827) 90%, rgba(255, 255, 255, 0.1));
	}
</style>
