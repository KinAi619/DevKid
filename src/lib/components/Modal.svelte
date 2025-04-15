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
	<div on:mousedown={close} on:touchstart={close} tabindex="0" role="button" class="modal-overlay">
		<div
			class="modal-content"
			on:mousedown|stopPropagation
			on:touchstart|stopPropagation
			tabindex="0"
			role="button"
		>
			<button type="button" on:click={close} class="close-btn" aria-label="Закрыть модальное окно"
				><svg
					width="16"
					height="16"
					viewBox="0 0 16 16"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
				>
					<path d="M1 15L15 1.25071" stroke="#3579F4" stroke-width="2" stroke-linecap="round" />
					<path d="M1 1L15 14.7493" stroke="#3579F4" stroke-width="2" stroke-linecap="round" />
				</svg>
			</button>
			<slot />
			<h3 class="modal__header">
				Оставьте свои контакты для связи и получите консультацию от наших специалистов
			</h3>
			<p class="modal__header_descr">
				Оставьте заявку, и мы рассчитаем самый благоприятный график для лекции.
			</p>

			<form action="#">
				<label for="input_name">Введите ваше имя</label>
				<input placeholder="Введите ваше имя" class="modal__input" type="text" id="input_name" />
			</form>
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
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 1000;
	}

	.modal-content {
		background: white;
		padding: 100px;
		max-width: 80%;
		position: relative;
	}

	.close-btn {
		position: absolute;
		top: 60px;
		right: 50px;
		background: #3578f40c;
		width: 60px;
		height: 60px;
		border: none;
		border-radius: 50%;
		font-size: 1.5rem;
		cursor: pointer;
		transition: all 0.2s ease-in-out;
	}

	.close-btn:hover {
		background: #3578f41a;
	}
</style>
