<script>
    import {emoji} from './emoji';

    let state = 'start';
    let size = 20;
    let grid = createGrid();
    let maxMatches = grid.length / 2;
    let selected = [];
    let matches = [];
    let timerId = null;
    let time = 60;

    function startGameTimer() {
        function countdown() {
            state !== 'paused' && (time -= 1);
        }

        timerId = setInterval(countdown, 1000);
    }

    function createGrid() {
        let cards = new Set();
        let maxSize = size / 2;

        while (cards.size < maxSize) {
            const randomIndex = Math.floor(Math.random() * emoji.length);
            cards.add(emoji[randomIndex]);
        }

        return shuffle([...cards, ...cards]);
    }

    function shuffle(array) {
        return array.sort(() => Math.random() - 0.5);
    }

    function selectCard(cardIndex) {
        selected = selected.concat(cardIndex);
    }

    function matchCards() {
        const [first, second] = selected;

        if (grid[first] === grid[second]) {
            matches = matches.concat(grid[first]);
        }

        setTimeout(() => {
            selected = [];
        }, 300);
    }

    function resetGame() {
        timerId && clearInterval(timerId);
        grid = createGrid();
        maxMatches = grid.length / 2;
        selected = [];
        matches = [];
        timerId = null;
        time = 20;
    }

    function gameWon() {
        state = 'won';
        resetGame();
    }

    function gameLost() {
        state = 'lost';
        resetGame();
    }

    function pauseGame(e) {
        if (e.key === 'Escape') {
            switch(state) {
                case 'playing':
                    state = 'paused';
                    break;
                case 'paused':
                    state = 'playing';
                    break;
            }
        }
    }

    $: if (state === 'playing') {
        !timerId && startGameTimer();
    }

    $: selected.length === 2 && matchCards();
    $: maxMatches === matches.length && gameWon();
    $: time === 0 && gameLost();

    $: console.log({state, selected, matches});
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'paused'}
    <h1>Game Paused</h1>
{/if}

{#if state === 'start'}
    <h1>Matching Gayme</h1>
    <button on:click={() => state = 'playing'} class="a">Play</button>
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

            <button class="card" on:click={() => selectCard(cardIndex)} class:selected={isSelected} class:flip={isSelectedOrMatched} disabled={isSelectedOrMatched}>
                <div class="back" class:match>{card}</div>
            </button>
        {/each}
    </div>
{/if}

{#if state === 'lost'}
    <h1>You Lost! 💩</h1>
    <button on:click={() => state = 'playing'} class="b">Play Again</button>
{/if}

{#if state === 'won'}
    <h1>You Win! 🥇</h1>
    <button on:click={() => state = 'playing'} class="b">Play Again</button>
{/if}

<style>
    .cards {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 0.4rem;
    }
    .card {
        height: 140px;
        width: 140px;
        font-size: 4rem;
        background-color: var(--bg-2);
        transition: rotate 0.3s ease-out;
        transform-style: preserve-3d;

        &.selected {
            border: 4px solid var(--border);
        }

        &.flip {
            rotate: y 180deg;
            pointer-events: none;
        }

        & .back {
            position: absolute;
            inset: 0;
            display: grid;
            place-items: center;
            backface-visibility: hidden;
            rotate: y 180deg;
        }

        & .match {
            transition: 0.3s ease-out;
            opacity: 0.4;
        }
    }
    .a {
        margin-left: 10rem;
    }
    .b {
        margin-left: 2rem;
    }
    .matches {
        display: flex;
        gap: 1rem;
        margin-block: 2rem;
        font-size: 3rem;
    }
    .timer {
        transition: color 0.3s ease;
    }
    .pulse {
        color: var(--pulse);
        animation: pulse 1s infinite ease;
    }
    @keyframes pulse {
        to {
            scale: 1.4;
        }
    }
</style>