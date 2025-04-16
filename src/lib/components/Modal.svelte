<script lang="ts">
	import { onMount } from 'svelte';

	export let isOpen = false; // Проп для управления видимостью

	function close() {
		isOpen = false;
	}

	onMount(() => {
		if (isOpen) document.body.style.overflow = 'hidden';
		return () => (document.body.style.overflow = '');
	});

	let comment = '';

	//Телефон

	let phone: string = '';
	let lastRawValue: string = '';
	let lastCursorPos: number = 0;

	function formatPhone(value: string): string {
		// Удаляем все нецифровые символы
		const numbers = value.replace(/\D/g, '');

		// Если ничего не введено - возвращаем пустую строку
		if (numbers === '') return '';

		// Определяем начало номера
		let prefix = '+7 ';
		let mainNumbers = numbers;

		if (numbers.startsWith('8')) {
			mainNumbers = numbers.slice(1);
		} else if (!numbers.startsWith('7')) {
			mainNumbers = numbers;
		} else {
			mainNumbers = numbers.slice(1);
		}

		// Форматируем основную часть
		let formatted = '';
		if (mainNumbers.length > 0) {
			formatted += `(${mainNumbers.slice(0, 3)}`;
			if (mainNumbers.length > 3) {
				formatted += `) ${mainNumbers.slice(3, 6)}`;
				if (mainNumbers.length > 6) {
					formatted += `-${mainNumbers.slice(6, 8)}`;
					if (mainNumbers.length > 8) {
						formatted += `-${mainNumbers.slice(8, 10)}`;
					}
				}
			}
		}

		return prefix + formatted;
	}

	function handleInputTel(e: Event) {
		const target = e.target as HTMLInputElement;
		const cursorPos = target.selectionStart || 0;
		const inputValue = target.value;

		// Сохраняем raw-значение (только цифры) для сравнения
		const currentRawValue = inputValue.replace(/\D/g, '');

		// Определяем тип изменения
		const isAdding = currentRawValue.length > lastRawValue.length;
		const isDeleting = currentRawValue.length < lastRawValue.length;

		// Форматируем новое значение
		const formattedValue = formatPhone(inputValue);

		// Вычисляем новую позицию курсора
		let newCursorPos = cursorPos;

		if (isAdding) {
			// При добавлении цифры двигаем курсор вперед
			const addedChars = formattedValue.length - phone.length;
			newCursorPos = cursorPos + addedChars;
		} else if (isDeleting) {
			// При удалении стараемся сохранить позицию
			if (cursorPos <= 3) {
				// Если удаляем в начале - очищаем поле
				phone = '';
				lastRawValue = '';
				lastCursorPos = 0;
				return;
			}
			newCursorPos = cursorPos;
		}

		// Обновляем состояние
		phone = formattedValue;
		lastRawValue = currentRawValue;
		lastCursorPos = newCursorPos;

		// Устанавливаем курсор
		setTimeout(() => {
			if (target) {
				target.selectionStart = newCursorPos;
				target.selectionEnd = newCursorPos;
			}
		}, 0);
	}

	function handleKeyDownTel(e: KeyboardEvent) {
		// Разрешаем только цифры и управляющие клавиши
		if (!/[0-9]|Backspace|Delete|Tab|ArrowLeft|ArrowRight/.test(e.key)) {
			e.preventDefault();
		}
	}

	//Имя

	let name: string = '';

	function handleInputName(e: Event) {
		const target = e.target as HTMLInputElement;
		name = target.value;

		const spliter = name.split(' ');
		console.log(name);

		// Удаляем пробел в начале
		if (name.charAt(0) === ' ') {
			name = name.slice(1);
		} else if (name.charAt(0) !== ' ' && spliter.length < 4) {
			// Делаем до 4 слов, потому что последний индекс становится пустым сразу после того как мы вводим пробел

			// Переводим первую букву каждого слова в верхний регистр

			for (let i = 0; i < spliter.length; i++) {
				if (spliter[i].charAt(0) === ' ') {
					spliter[i] = spliter[i].slice(1);
				} else {
					spliter[i] = spliter[i].charAt(0).toUpperCase() + spliter[i].slice(1);
				}
			}

			name = spliter.join(' ');
		} else {
			// Удаляем пробел в конце
			name = name.slice(0, -1);
			spliter.splice(spliter.length - 1, 1);
		}
		console.log(spliter);
	}
</script>

{#if isOpen}
	<div on:mousedown={close} on:touchstart={close} tabindex="0" role="button" class="modal_overlay">
		<div
			class="modal_content"
			on:mousedown|stopPropagation
			on:touchstart|stopPropagation
			tabindex="0"
			role="button"
		>
			<button type="button" on:click={close} class="close_btn" aria-label="Закрыть модальное окно">
				<svg
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

			<form class="modal__form" action="#">
				<div class="modal__input_group">
					<label class="modal__label" for="modal__input_name">Введите ваше имя</label>
					<input
						required
						placeholder="Введите ваше имя"
						class="modal__input"
						type="text"
						id="modal__input_name"
						bind:value={name}
						on:input={handleInputName}
					/>
				</div>
				<div class="modal__input_group">
					<label class="modal__label" for="modal__input_phone">Введите номер телефона</label>
					<input
						required
						placeholder="+7 (___) ___-__-__"
						class="modal__input"
						type="tel"
						id="modal__input_phone"
						bind:value={phone}
						on:input={handleInputTel}
						on:keydown={handleKeyDownTel}
						maxlength="18"
					/>
				</div>
				<div class="modal__comment_field">
					<label class="modal__label modal__textarea_label" for="modal__comment">
						* Ваш Комментарий &nbsp; <em class="modal__label__hint">(не обязательно)</em>
					</label>
					<textarea
						class="modal__input"
						name="modal__comment"
						id="modal__comment"
						bind:value={comment}
						placeholder="Комментарий..."
						aria-describedby="comment-hint"
						rows="4"
					>
					</textarea>
				</div>

				<div class="modal__btn_group">
					<button class="btn modal__btn" type="submit"> Попробовать бесплатно </button>
					<p class="modal__btn_descr">
						Нажимая кнопку вы соглашаетесь с политикой конфиденциальности
					</p>
				</div>
			</form>
		</div>
	</div>
{/if}

<style>
	.modal_overlay {
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

	.modal_content {
		background: white;
		padding: 100px;
		max-width: 80%;
		position: relative;
		font-family: Mulish;
	}

	.close_btn {
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

	.close_btn:hover {
		background: #3578f41a;
	}

	.modal__header {
		font-weight: 600;
		font-size: 25px;
		max-width: 80%;
		margin-bottom: 25px;
	}

	.modal__header_descr {
		font-size: 18px;
		color: #1e1e1e69;
		max-width: 350px;
		margin-bottom: 40px;
	}

	.modal__form {
		display: flex;
		flex-wrap: wrap;
	}

	.modal__input_group {
		margin-bottom: 60px;
		margin-right: 20px;
		display: flex;
		flex-direction: column;
		width: calc((100% - 20px) / 2);
	}

	.modal__input_group:nth-child(2) {
		margin-right: 0;
	}

	.modal__label {
		font-weight: 600;
		font-size: 22px;
		margin-bottom: 15px;
	}

	.modal__input {
		border: 1px solid transparent;
		outline: none;
		font-size: 18px;
		border-radius: 10px;
		padding: 25px 30px;
		background: #fafafa;
	}

	.modal__input:hover {
		border: 1px solid #4766f141;
		background: #3578f40c;
	}

	.modal__input:focus {
		border: 1px solid #6d6d6d41;
		background: #3578f40c;
	}

	#modal__comment {
		resize: none;
		width: 100%;
	}

	.modal__comment_field {
		display: flex;
		flex-direction: column;
		width: 100%;
		margin-bottom: 40px;
	}

	.modal__label__hint {
		font-size: 14px;
		color: #1e1e1e80;
	}
	.modal__btn_group {
		display: flex;
		align-items: center;
	}

	.modal__btn {
		font-size: 18px;
		font-weight: 700;
		font-family: Mulish;
	}

	.modal__btn_descr {
		font-size: 14px;
		color: #1e1e1e69;
		max-width: 350px;
		margin-left: 20px;
	}
</style>
