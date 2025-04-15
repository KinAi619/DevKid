<script>
	import { onMount } from 'svelte';

	export let isOpen = false; // Проп для управления видимостью

	function close() {
		isOpen = false;
	}

	onMount(() => {
		if (isOpen) document.body.style.overflow = 'hidden';
		return () => (document.body.style.overflow = '');
	});
</script>

{#if isOpen}
	<div class="modal-overlay" on:click={close}>
		<div class="modal-content" on:click|stopPropagation>
			<button on:click={close} class="close-btn">×</button>
			<slot />
			<p>qwrfnqwiopqw</p>
		</div>
	</div>
{/if}

<style>
	.modal-overlay {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.5);
		display: grid;
		place-items: center;
		z-index: 1000;
	}

	.modal-content {
		background: white;
		padding: 2rem;
		border-radius: 8px;
		max-width: 80%;
		position: relative;
	}

	.close-btn {
		position: absolute;
		top: 10px;
		right: 10px;
		background: none;
		border: none;
		font-size: 1.5rem;
		cursor: pointer;
	}
</style>
