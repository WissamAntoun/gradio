<script lang="ts">
	import Copy from "./Copy.svelte";
	import Download from "./Download.svelte";

	export let value: string;
	export let language: string;
</script>

<div>
	<Download {value} {language} />
	<Copy {value} />
</div>

<style>
	div {
		display: flex;
		position: absolute;
		top: var(--block-label-margin);
		right: var(--block-label-margin);
		align-items: center;

		z-index: var(--layer-2);
		transition: 150ms;
		box-shadow: var(--shadow-drop);
		border: 1px solid var(--color-border-primary);
		border-top: none;
		border-right: none;
		border-radius: var(--block-label-right-radius);
		background: var(--block-label-background);
		overflow: hidden;
		color: var(--block-label-color);
		font: var(--font-sans);
		font-size: var(--button-small-text-size);
	}
</style>
