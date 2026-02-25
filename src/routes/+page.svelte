<script lang="ts">
	import { onMount, tick } from 'svelte';

	const WORDS = [
		'the',
		'be',
		'to',
		'of',
		'and',
		'a',
		'in',
		'that',
		'have',
		'it',
		'for',
		'not',
		'on',
		'with',
		'he',
		'as',
		'you',
		'do',
		'at',
		'this',
		'but',
		'his',
		'by',
		'from',
		'they',
		'we',
		'say',
		'her',
		'she',
		'or',
		'an',
		'will',
		'my',
		'one',
		'all',
		'would',
		'there',
		'their',
		'what',
		'so',
		'up',
		'out',
		'if',
		'about',
		'who',
		'get',
		'which',
		'go',
		'me',
		'when',
		'make',
		'can',
		'like',
		'time',
		'no',
		'just',
		'him',
		'know',
		'take',
		'people',
		'into',
		'year',
		'your',
		'good',
		'some',
		'could',
		'them',
		'see',
		'other',
		'than',
		'then',
		'now',
		'look',
		'only',
		'come',
		'its',
		'over',
		'think',
		'also',
		'back',
		'after',
		'use',
		'two',
		'how',
		'our',
		'work',
		'first',
		'well',
		'way',
		'even',
		'new',
		'want',
		'because',
		'any',
		'these',
		'give',
		'day',
		'most',
		'us',
		'great',
		'between',
		'need',
		'large',
		'often',
		'hand',
		'high',
		'place',
		'hold',
		'turn',
		'help',
		'move',
		'play',
		'small',
		'number',
		'off',
		'always',
		'point',
		'world',
		'near',
		'build',
		'self',
		'earth',
		'head',
		'stand',
		'own',
		'page',
		'should',
		'country',
		'found',
		'answer',
		'school',
		'grow',
		'study',
		'still',
		'learn',
		'plant',
		'food',
		'sun',
		'four',
		'state',
		'keep',
		'eye',
		'never',
		'last',
		'let',
		'thought',
		'city',
		'tree',
		'cross',
		'farm',
		'hard',
		'start',
		'might',
		'story',
		'far',
		'sea',
		'draw',
		'left',
		'late',
		'run',
		'while',
		'press',
		'close',
		'night',
		'real',
		'life',
		'few',
		'north',
		'open',
		'seem',
		'together',
		'next',
		'white',
		'children',
		'begin',
		'got',
		'walk',
		'example',
		'paper',
		'group',
		'music',
		'those',
		'both',
		'mark',
		'book',
		'letter',
		'until',
		'mile',
		'river',
		'car',
		'feet',
		'care',
		'second',
		'enough',
		'plain',
		'girl',
		'young',
		'ready',
		'above',
		'ever',
		'red',
		'list',
		'though',
		'feel',
		'talk',
		'bird',
		'soon',
		'body',
		'dog',
		'family',
		'direct',
		'leave',
		'song',
		'measure',
		'door',
		'product',
		'black',
		'short',
		'class',
		'wind',
		'question',
		'happen',
		'complete',
		'ship',
		'area',
		'half',
		'rock',
		'order',
		'fire',
		'south',
		'problem',
		'piece',
		'told',
		'knew',
		'pass',
		'since',
		'top',
		'whole',
		'king',
		'space',
		'heard',
		'best',
		'hour',
		'better',
		'true',
		'during',
		'hundred',
		'five',
		'remember',
		'step',
		'early',
		'west',
		'ground',
		'interest',
		'reach',
		'fast',
		'listen',
		'six',
		'table',
		'travel',
		'less',
		'morning',
		'ten',
		'simple',
		'several',
		'toward',
		'war',
		'lay',
		'against',
		'pattern',
		'slow',
		'center',
		'love',
		'person',
		'money',
		'serve',
		'appear',
		'road',
		'map',
		'rain',
		'rule',
		'pull',
		'cold',
		'notice',
		'voice',
		'power',
		'town',
		'fine',
		'drive',
		'lead',
		'cry',
		'dark',
		'machine',
		'note',
		'wait',
		'plan',
		'figure',
		'star',
		'box',
		'field',
		'rest',
		'able',
		'done',
		'stood',
		'under',
		'along',
		'sound',
		'something',
		'show'
	];

	type Mode = 'time' | 'words';
	type TestState = 'idle' | 'running' | 'finished';
	type CharState = 'correct' | 'wrong' | null;
	type FloatingChar = { id: number; char: string; x: number; y: number };

	// Settings
	let mode = $state<Mode>('time');
	let timeLimit = $state(30);
	let wordLimit = $state(30);

	// Test state
	let testState = $state<TestState>('idle');
	let words = $state<string[]>([]);
	let charStates = $state<CharState[][]>([]);
	let typedWords = $state<string[]>([]);
	let currentWordIndex = $state(0);
	let currentInput = $state('');
	let timeRemaining = $state(30);
	let startTime = 0;
	let timerInterval: number | null = null;

	// Results
	let wpm = $state(0);
	let accuracy = $state(0);
	let displayWpm = $state(0);
	let displayAccuracy = $state(0);

	// DOM refs
	let inputEl: HTMLInputElement;
	let wordsEl = $state<HTMLElement | null>(null);
	let wordEls = $state<(HTMLElement | null)[]>([]);
	let isFocused = $state(false);
	let floatingChars = $state<FloatingChar[]>([]);
	let nextFloatId = 0;
	let prevInputLen = 0;

	// Derived
	let currentWord = $derived(words[currentWordIndex] ?? '');
	let extraChars = $derived(
		currentInput.length > currentWord.length ? currentInput.slice(currentWord.length).split('') : []
	);

	// Helpers
	function shuffle<T>(arr: T[]): T[] {
		const a = [...arr];
		for (let i = a.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[a[i], a[j]] = [a[j], a[i]];
		}
		return a;
	}

	function generateWords(count: number): string[] {
		const result: string[] = [];
		while (result.length < count) result.push(...shuffle(WORDS));
		return result.slice(0, count);
	}

	// Test lifecycle
	function initTest() {
		if (timerInterval !== null) {
			clearInterval(timerInterval);
			timerInterval = null;
		}
		const count = mode === 'words' ? wordLimit : 120;
		const newWords = generateWords(count);
		words = newWords;
		charStates = newWords.map((w) => new Array(w.length).fill(null));
		currentWordIndex = 0;
		currentInput = '';
		if (inputEl) inputEl.value = '';
		testState = 'idle';
		timeRemaining = timeLimit;
		wpm = 0;
		accuracy = 0;
		displayWpm = 0;
		displayAccuracy = 0;
		typedWords = new Array(newWords.length).fill('');
		wordEls = new Array(newWords.length).fill(null);
		floatingChars = [];
		prevInputLen = 0;
		tick().then(() => {
			if (wordsEl) wordsEl.scrollTop = 0;
			inputEl?.focus();
		});
	}

	function startTest() {
		testState = 'running';
		startTime = Date.now();
		if (mode === 'time') {
			timerInterval = window.setInterval(() => {
				timeRemaining = Math.max(0, timeRemaining - 1);
				if (timeRemaining <= 0) endTest();
			}, 1000);
		}
	}

	function endTest() {
		if (timerInterval !== null) {
			clearInterval(timerInterval);
			timerInterval = null;
		}
		computeResults();
		testState = 'finished';
	}

	function computeResults() {
		let totalChars = 0,
			correctChars = 0,
			correctWords = 0;
		for (let i = 0; i < currentWordIndex; i++) {
			const states = charStates[i];
			let allCorrect = states.length > 0;
			for (const s of states) {
				if (s !== null) {
					totalChars++;
					if (s === 'correct') correctChars++;
					else allCorrect = false;
				} else {
					allCorrect = false;
				}
			}
			if (allCorrect) correctWords++;
		}
		const elapsed = (Date.now() - startTime) / 1000;
		const minutes = mode === 'time' ? timeLimit / 60 : elapsed / 60;
		wpm = minutes > 0 ? Math.round(correctWords / minutes) : 0;
		accuracy = totalChars > 0 ? Math.round((correctChars / totalChars) * 100) : 100;
	}

	async function spawnFloat(char: string, charIndex: number, wordIndex: number) {
		await tick();
		const wordEl = wordEls[wordIndex];
		if (!wordEl) return;
		const charEls = wordEl.querySelectorAll<HTMLElement>('.char');
		const charEl = charEls[charIndex];
		if (!charEl) return;
		const rect = charEl.getBoundingClientRect();
		const id = nextFloatId++;
		floatingChars = [...floatingChars, { id, char, x: rect.left, y: rect.top }];
		setTimeout(() => {
			floatingChars = floatingChars.filter((f) => f.id !== id);
		}, 700);
	}

	function updateCurrentCharStates() {
		const word = currentWord;
		if (!word) return;
		const states: CharState[] = new Array(word.length).fill(null);
		for (let j = 0; j < Math.min(currentInput.length, word.length); j++) {
			states[j] = currentInput[j] === word[j] ? 'correct' : 'wrong';
		}
		charStates[currentWordIndex] = states;

		// Spawn float for newly typed wrong char (only on forward typing)
		if (currentInput.length > prevInputLen) {
			const idx = currentInput.length - 1;
			if (idx < word.length) {
				if (states[idx] === 'wrong') {
					spawnFloat(currentInput[idx], idx, currentWordIndex);
				}
			} else {
				// Extra char typed beyond word length — always wrong
				spawnFloat(currentInput[idx], idx, currentWordIndex);
			}
		}
		prevInputLen = currentInput.length;

		// Auto-finish: last word in words mode typed correctly without needing space
		if (
			mode === 'words' &&
			currentWordIndex === wordLimit - 1 &&
			currentInput.length === word.length &&
			states.every((s) => s === 'correct')
		) {
			submitWord();
		}
	}

	function submitWord() {
		const word = currentWord;
		if (!word) return;
		const states: CharState[] = new Array(word.length).fill('wrong');
		for (let j = 0; j < word.length; j++) {
			if (j < currentInput.length) {
				states[j] = currentInput[j] === word[j] ? 'correct' : 'wrong';
			}
		}
		charStates[currentWordIndex] = states;
		typedWords[currentWordIndex] = currentInput;
		currentWordIndex++;
		currentInput = '';
		prevInputLen = 0;
		if (inputEl) inputEl.value = '';
		if (mode === 'words' && currentWordIndex >= wordLimit) {
			endTest();
			return;
		}
		tick().then(scrollToCurrent);
	}

	function scrollToCurrent() {
		const el = wordEls[currentWordIndex];
		if (!el || !wordsEl) return;
		const lineH = getLineHeight();
		wordsEl.scrollTop = Math.max(0, el.offsetTop - lineH);
	}

	function getLineHeight(): number {
		for (let i = 0; i + 1 < wordEls.length; i++) {
			const a = wordEls[i],
				b = wordEls[i + 1];
			if (a && b && b.offsetTop > a.offsetTop) return b.offsetTop - a.offsetTop;
		}
		return 48;
	}

	// Input handlers
	function handleKeydown(e: KeyboardEvent) {
		if (testState === 'finished') {
			if (e.key === 'Tab' || e.key === 'Enter') {
				e.preventDefault();
				initTest();
			}
			return;
		}
		if (e.key === ' ') {
			e.preventDefault();
			if (currentInput.length > 0) submitWord();
		} else if (e.key === 'Backspace' && currentInput === '' && currentWordIndex > 0) {
			e.preventDefault();
			currentWordIndex--;
			const restored = typedWords[currentWordIndex] ?? '';
			currentInput = restored;
			prevInputLen = restored.length;
			if (inputEl) inputEl.value = restored;
			updateCurrentCharStates();
			tick().then(scrollToCurrent);
		} else if (e.key === 'Tab') {
			e.preventDefault();
			initTest();
		}
	}

	function handleInput(e: Event) {
		if (testState === 'finished') return;
		const raw = (e.target as HTMLInputElement).value;
		// Strip spaces — space submission is handled in keydown.
		// Some browsers fire the input event anyway despite preventDefault.
		const val = raw.replace(/\s/g, '');
		if (raw !== val && inputEl) inputEl.value = val;
		currentInput = val;
		if (testState === 'idle' && val.length > 0) startTest();
		updateCurrentCharStates();
	}

	// Lifecycle
	onMount(initTest);

	// Animate results numbers on finish
	$effect(() => {
		if (testState !== 'finished') return;
		const targetWpm = wpm,
			targetAcc = accuracy;
		let frame = 0;
		const total = 50;
		const id = setInterval(() => {
			frame++;
			const t = Math.min(frame / total, 1);
			const ease = 1 - Math.pow(1 - t, 3);
			displayWpm = Math.round(targetWpm * ease);
			displayAccuracy = Math.round(targetAcc * ease);
			if (frame >= total) clearInterval(id);
		}, 16);
		return () => clearInterval(id);
	});
</script>

<div class="page">
	<div class="container">
		<!-- Header -->
		<div class="header">
			<span class="logo">type<span class="dot">.</span></span>
			<nav class="settings" aria-label="Test settings">
				<div class="btn-group">
					<button
						class="s-btn"
						class:active={mode === 'time'}
						onclick={() => {
							mode = 'time';
							initTest();
						}}>time</button
					>
					<button
						class="s-btn"
						class:active={mode === 'words'}
						onclick={() => {
							mode = 'words';
							initTest();
						}}>words</button
					>
				</div>
				<span class="sep" aria-hidden="true">/</span>
				<div class="btn-group">
					{#if mode === 'time'}
						{#each [10, 30, 60] as t (t)}
							<button
								class="s-btn"
								class:active={timeLimit === t}
								onclick={() => {
									timeLimit = t;
									initTest();
								}}>{t}</button
							>
						{/each}
					{:else}
						{#each [10, 30, 60] as w (w)}
							<button
								class="s-btn"
								class:active={wordLimit === w}
								onclick={() => {
									wordLimit = w;
									initTest();
								}}>{w}</button
							>
						{/each}
					{/if}
				</div>
			</nav>
		</div>

		<!-- Hidden capture input -->
		<input
			bind:this={inputEl}
			class="ghost-input"
			type="text"
			autocomplete="off"
			autocorrect="off"
			autocapitalize="off"
			spellcheck={false}
			tabindex="-1"
			oninput={handleInput}
			onkeydown={handleKeydown}
			onfocus={() => (isFocused = true)}
			onblur={() => (isFocused = false)}
		/>

		{#if testState !== 'finished'}
			<!-- Typing area -->
			<div class="typing-section">
				<div
					class="words-wrap"
					class:blurred={!isFocused}
					onclick={() => inputEl?.focus()}
					role="button"
					tabindex="0"
					aria-label="Click to start typing"
					onkeypress={() => {}}
				>
					<div class="words-viewport" bind:this={wordsEl}>
						{#each words as word, i (i)}
							{@const isActive = i === currentWordIndex}
							{@const isDone = i < currentWordIndex}
							{@const doneExtras = isDone ? (typedWords[i] ?? '').slice(word.length).split('') : []}
							<span
								class="word"
								class:word-active={isActive}
								class:word-done={isDone}
								bind:this={wordEls[i]}
							>
								{#each word.split('') as char, j (j)}
									{@const state = charStates[i]?.[j] ?? null}
									{@const isCursorBefore = isActive && j === currentInput.length}
									{@const isCursorAfter =
										isActive && j === word.length - 1 && currentInput.length === word.length}
									<span
										class="char"
										class:char-correct={state === 'correct'}
										class:char-wrong={state === 'wrong'}
										class:char-cursor={isCursorBefore}
										class:char-cursor-end={isCursorAfter}>{char}</span
									>
								{/each}
								{#if isActive}
									{#each extraChars as ec, k (k)}
										<span
											class="char char-wrong"
											class:char-cursor-end={k === extraChars.length - 1}>{ec}</span
										>
									{/each}
								{:else if doneExtras.length > 0}
									{#each doneExtras as ec, k (k)}
										<span class="char char-wrong">{ec}</span>
									{/each}
								{/if}
							</span>
						{/each}
					</div>
					{#if !isFocused}
						<div class="focus-hint">click to focus</div>
					{/if}
				</div>

				<!-- Progress / timer -->
				<div class="status-bar">
					{#if mode === 'time'}
						<span class="counter" class:counter-warn={timeRemaining <= 5 && testState === 'running'}
							>{timeRemaining}</span
						>
					{:else}
						<span class="counter"
							>{currentWordIndex}<span class="counter-dim">/{wordLimit}</span></span
						>
					{/if}
				</div>

				<div class="hint">tab — restart</div>
			</div>
		{:else}
			<!-- Results screen -->
			<div class="results">
				<div class="results-row">
					<div class="stat">
						<div class="stat-num">{displayWpm}</div>
						<div class="stat-label">wpm</div>
					</div>
					<div class="stat-divider"></div>
					<div class="stat">
						<div class="stat-num">
							{displayAccuracy}<span class="stat-unit">%</span>
						</div>
						<div class="stat-label">acc</div>
					</div>
				</div>
				<button class="retry-btn" onclick={initTest}>try again</button>
				<div class="hint">or press enter</div>
			</div>
		{/if}
	</div>
	{#each floatingChars as fc (fc.id)}
		<span class="float-char" style:left="{fc.x}px" style:top="{fc.y}px">{fc.char}</span>
	{/each}
</div>

<style>
	/* Base */
	:global(*) {
		box-sizing: border-box;
	}

	:global(body) {
		background: #0e0c0a;
		color: #c4b7a4;
		font-family: 'JetBrains Mono', 'Courier New', monospace;
		margin: 0;
		padding: 0;
	}

	/* Page layout */
	.page {
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 2rem;
	}

	.container {
		width: 100%;
		max-width: 900px;
		display: flex;
		flex-direction: column;
		gap: 2.75rem;
	}

	/* Header */
	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		flex-wrap: wrap;
		gap: 1rem;
	}

	.logo {
		font-size: 1.35rem;
		font-weight: 600;
		letter-spacing: -0.03em;
		color: #6e6258;
		user-select: none;
	}

	.dot {
		color: #e89820;
	}

	/* Settings nav */
	.settings {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.btn-group {
		display: flex;
		align-items: center;
		gap: 0.1rem;
	}

	.s-btn {
		background: none;
		border: none;
		cursor: pointer;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.8rem;
		color: #4a4038;
		padding: 0.3rem 0.55rem;
		border-radius: 3px;
		transition:
			color 0.15s,
			background 0.15s;
		letter-spacing: 0.02em;
	}

	.s-btn:hover {
		color: #c4b7a4;
		background: #1c1a17;
	}

	.s-btn.active {
		color: #e89820;
	}

	.sep {
		color: #2d2820;
		font-size: 0.9rem;
		user-select: none;
	}

	/* Ghost input */
	.ghost-input {
		position: absolute;
		opacity: 0;
		pointer-events: none;
		width: 1px;
		height: 1px;
		padding: 0;
		border: none;
		outline: none;
	}

	/* Typing section */
	.typing-section {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	.words-wrap {
		position: relative;
		width: 100%;
		cursor: text;
		transition: opacity 0.25s;
	}

	.words-wrap.blurred {
		opacity: 0.3;
		filter: blur(3px);
	}

	.focus-hint {
		position: absolute;
		inset: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 0.8rem;
		color: #6e6258;
		pointer-events: none;
		letter-spacing: 0.06em;
	}

	/* Words viewport */
	.words-viewport {
		position: relative;
		width: 100%;
		height: 7.5rem;
		overflow-y: scroll;
		scrollbar-width: none;
		line-height: 2.5rem;
		user-select: none;
	}

	.words-viewport::-webkit-scrollbar {
		display: none;
	}

	/* Word rendering */
	.word {
		display: inline-flex;
		align-items: baseline;
		font-size: 1.4rem;
		margin-right: 0.65rem;
		color: #6b5e52;
	}

	.word-done {
		color: #3d342c;
	}

	.word-active .char:not(.char-correct):not(.char-wrong) {
		color: #6b5e52;
	}

	.char-correct {
		color: #5daa52 !important;
	}

	.char-wrong {
		color: #c24242 !important;
	}

	/* Cursor bar before the next character */
	.char-cursor {
		position: relative;
	}

	.char-cursor::before {
		content: '';
		position: absolute;
		left: -1px;
		top: 15%;
		height: 70%;
		width: 2px;
		background: #e89820;
		border-radius: 1px;
		animation: blink 1.1s ease-in-out infinite;
	}

	/* Cursor after last typed char. Absolute so it takes no layout space */
	.char-cursor-end {
		position: relative;
	}

	.char-cursor-end::after {
		content: '';
		position: absolute;
		right: -2px;
		top: 15%;
		height: 70%;
		width: 2px;
		background: #e89820;
		border-radius: 1px;
		animation: blink 1.1s ease-in-out infinite;
	}

	@keyframes blink {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0;
		}
	}

	/* Status bar */
	.status-bar {
		min-height: 2rem;
		display: flex;
		align-items: center;
	}

	.counter {
		font-size: 1.75rem;
		font-weight: 500;
		letter-spacing: -0.03em;
		color: #e89820;
		text-shadow: 0 0 24px color-mix(in srgb, #e89820 35%, transparent);
		transition: color 0.3s;
	}

	.counter-warn {
		color: #c24242;
		text-shadow: 0 0 24px color-mix(in srgb, #c24242 40%, transparent);
	}

	.counter-dim {
		color: #3d342c;
		font-size: 1.25rem;
	}

	/* Hint text */
	.hint {
		font-size: 0.72rem;
		color: #6b5e52;
		letter-spacing: 0.08em;
	}

	/* Results screen */
	.results {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		gap: 2.5rem;
		animation: slideUp 0.4s cubic-bezier(0.22, 1, 0.36, 1);
	}

	@keyframes slideUp {
		from {
			opacity: 0;
			transform: translateY(16px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.results-row {
		display: flex;
		align-items: center;
		gap: 3rem;
	}

	.stat {
		display: flex;
		flex-direction: column;
		gap: 0.4rem;
	}

	.stat-num {
		font-size: 5rem;
		font-weight: 600;
		line-height: 1;
		letter-spacing: -0.05em;
		color: #ede0c8;
	}

	.stat-unit {
		font-size: 2.5rem;
		font-weight: 300;
		color: #6e6258;
		letter-spacing: 0;
	}

	.stat-label {
		font-size: 0.8rem;
		color: #6e6258;
		letter-spacing: 0.1em;
	}

	.stat-divider {
		width: 1px;
		height: 4.5rem;
		background: #2d2820;
		flex-shrink: 0;
	}

	.retry-btn {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.8rem;
		background: none;
		border: 1px solid #2d2820;
		color: #6e6258;
		cursor: pointer;
		padding: 0.6rem 1.6rem;
		border-radius: 3px;
		letter-spacing: 0.05em;
		transition:
			border-color 0.15s,
			color 0.15s;
	}

	.retry-btn:hover {
		border-color: #e89820;
		color: #e89820;
	}

	/* Wrong letter float */
	.float-char {
		position: fixed;
		pointer-events: none;
		font-family: 'JetBrains Mono', 'Courier New', monospace;
		font-size: 1.4rem;
		color: #c24242;
		animation: wrongFloat 0.65s cubic-bezier(0.22, 1, 0.36, 1) forwards;
		z-index: 9999;
		user-select: none;
	}

	@keyframes wrongFloat {
		0% {
			transform: translateY(0) translateX(0) scale(1);
			opacity: 1;
		}
		15% {
			transform: translateY(-8px) translateX(5px) scale(1.2);
			opacity: 1;
		}
		35% {
			transform: translateY(-22px) translateX(-7px) scale(1.05);
			opacity: 0.85;
		}
		55% {
			transform: translateY(-36px) translateX(5px);
			opacity: 0.55;
		}
		75% {
			transform: translateY(-46px) translateX(-3px);
			opacity: 0.25;
		}
		100% {
			transform: translateY(-58px) translateX(0);
			opacity: 0;
		}
	}

	/* Responsive */
	@media (max-width: 480px) {
		.page {
			padding: 1.25rem;
			align-items: flex-start;
			padding-top: 3rem;
		}

		.word {
			font-size: 1.2rem;
		}

		.words-viewport {
			height: 6.75rem;
			line-height: 2.25rem;
		}

		.stat-num {
			font-size: 3.5rem;
		}

		.header {
			flex-direction: column;
			align-items: flex-start;
		}
	}
</style>
