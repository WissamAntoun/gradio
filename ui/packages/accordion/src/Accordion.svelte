<script lang="ts">
	export let label: string = "";
	export let open: boolean = true;
</script>

<div on:click={() => (open = !open)} class="label-wrap" class:open>
	<span>{label}</span>
	<span style:transform={open ? "rotate(0)" : "rotate(90deg)"} class="icon">
		▼
	</span>
</div>
{#if open}
	<slot />
{/if}

<style>
	span {
		font-weight: var(--section-text-weight);
		font-size: var(--section-text-size);
	}
	.label-wrap {
		display: flex;
		justify-content: space-between;
		cursor: pointer;
		width: var(--size-full);
	}
	.label-wrap.open {
		margin-bottom: var(--size-2);
	}

	.icon {
		transition: 150ms;
	}
</style>
