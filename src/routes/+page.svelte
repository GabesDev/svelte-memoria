<script lang="ts">
	import { emoji } from './emoji'

	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

	let state: State = 'start'
	let size: number = 20
	let grid: string[] = createGrid()

	let maxMatches: number = grid.length / 2
	let selected: number[] = []
	let matches: string[] = []

	let timerId: number | null = null
	let time: number = 50

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1)
		}
		timerId = setInterval(countdown, 1000)
	}

	function createGrid() {
		let cards: Set<string> = new Set<string>()
		let maxSize = size / 2

		while (cards.size < maxSize) {
			const randomIndex: number = Math.floor(Math.random() * emoji.length)
			cards.add(emoji[randomIndex])
		}

		return shuffle([...cards, ...cards])
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5)
	}

	function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex)
	}

	function matchCards() {
		const [first, second] = selected

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first])
		}

		setTimeout(() => (selected = []), 300)
	}

	function resetGame() {
		timerId && clearInterval(timerId)
		grid = createGrid()
		maxMatches = grid.length / 2
		selected = []
		matches = []
		timerId = null
		time = 50
	}

	function gameWon() {
		state = 'won'
		resetGame()
	}

	function gameLost() {
		state = 'lost'
		resetGame()
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused'
					break
				case 'paused':
					state = 'playing'
					break
			}
		}
	}

	$: if (state == 'playing') {
		!timerId && startGameTimer()
	}

	$: selected.length === 2 && matchCards()
	$: maxMatches === matches.length && gameWon()
	$: time === 0 && gameLost()
</script>

<svelte:window on:keydown={pauseGame} />

<a href="https://gabes.dev/" class="link">Gabes.dev</a>

{#if state == 'paused'}
	<h1>Jogo pausado</h1>
{/if}

{#if state === 'start'}
	<h1>Jogo da memória</h1>
	<button on:click={() => (state = 'playing')}>Jogar</button>
{/if}

{#if state === 'playing'}
	<h1 class="timer" class:pulse={time <= 10}>{time}</h1>
	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
	</div>
	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}
			<button
				class:selected={isSelected}
				class:flip={isSelectedOrMatched}
				disabled={isSelectedOrMatched}
				class="card"
				on:click={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>{card}</div>
			</button>
		{/each}
	</div>
{/if}

{#if state === 'lost'}
	<h1>Você perdeu! 💩</h1>
	<button on:click={() => (state = 'playing')}>Jogar novamente</button>
{/if}

{#if state === 'won'}
	<h1>Você ganhou! 🎉</h1>
	<button on:click={() => (state = 'playing')}>Jogar novamente</button>
{/if}

<style>
	.cards {
		display: grid;
		gap: 0.4rem;
		grid-template-columns: repeat(5, 1fr);
		width: 85vw;
		max-width: 750px;
	}

	.link {
		position: absolute;
		top: 0;
		right: 0;
		width: max-content;
		margin: 2rem;
		padding: 0.5rem;
		background: none;
		border: 2px solid var(--border);
		border-radius: 8px;
		font-size: clamp(1rem, 3.5vw, 1.5rem);
		font-weight: 900;
		color: inherit;
		text-decoration: none;
	}

	.card {
		aspect-ratio: 1 / 1;
		transform-style: preserve-3d;
		width: 100%;
		transition: rotate 0.3s ease-out;
		background-color: var(--bg-2);
		font-size: clamp(2rem, 7vw, 4rem);
		
		&.selected {
			border: 4px solid var(--border);
		}

		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			inset: 0;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
			display: grid;
			position: absolute;
		}

		& .match {
			opacity: 0.4;
			transition: opacity 0.3 ease-out;
		}
	}

	.matches {
		margin-block: 2rem;
		display: flex;
		gap: 1rem;
		font-size: clamp(1.5rem, 4.75vw, 3rem);
	}

	.timer {
		transition: color 0.3s ease;
	}

	.pulse {
		animation: pulse 1s infinite ease;
		color: var(--pulse);
	}

	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}
</style>
