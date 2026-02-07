<script lang="ts">
    import { onMount } from 'svelte';
    import { emoji } from "./emoji";

    type State = 'start' | 'won' | 'lost' | 'playing' | 'paused';

    let state: State = 'start';

    let size = 20;
    let grid = createGrid();
    let maxMatches = grid.length / 2;
    let selected: number[] = []; // grid indices
    let matches: string[] = [];
    let timerId: number | null = null;
    let timeLeft = 60;

    onMount(() => {
        if (typeof window !== 'undefined') {
            document.documentElement.classList.add('no-scroll');
        }
        return () => {
            if (typeof window !== 'undefined') {
                document.documentElement.classList.remove('no-scroll');
            }
        };
    });


    function startTimer() {
        function countDown() {
            state !== 'paused' && (timeLeft -= 1);
        }
        timerId = setInterval(countDown, 1000);
    }

    function createGrid() {
        let cards = new Set<string>();
        let maxSize = size / 2;

        while (cards.size < maxSize) {
            let index = Math.floor(Math.random() * emoji.length);
            cards.add(emoji[index]);
        }

        return shuffle([...cards, ...cards]);

    }
    function shuffle<T>(arr: T[]): T[] {
        const shuffled = [...arr];
        for (let i = shuffled.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
        }
        return shuffled;
    }

    function matchCards() {
        const [i, j] = selected;
        if (grid[i] === grid[j]) {
            matches = matches.concat(grid[i]);
            console.log("direct hit");
            selected = [];
        }
    }

    function gameWon() {
        state = 'won';
        resetGame();
    }

    function gameLost() {
        state = 'lost';
        resetGame();
    }

    function resetGame() {
        grid = createGrid();
        timeLeft = 60;
        timerId && clearInterval(timerId);
        timerId = null;
        maxMatches = grid.length/2;
        selected = [];
        matches = [];
    }
    
    function selectCard(cardIndex: number) {
        if (selected.length === 2) {
            selected = [cardIndex];
        } else {
            selected = selected.concat(cardIndex);
        }
    }

    function canShowEmoji(emoji: string) {
        return matches.includes(emoji) || selected.some(i => grid[i] === emoji);
    }
    
    // Reactive stuff

    $: selected.length === 2 && matchCards();
    $: timeLeft === 0 && gameLost();
    $: if (state === 'playing') {
        // to check for paused
        !timerId && startTimer();
    }
    $: if (maxMatches === matches.length) {
        gameWon();
    }
    $: console.log({state, selected, matches, maxMatches});

    onMount(() => {
        state = 'playing';
    });

</script>

<!-- Removido bloco de início; o jogo começa automaticamente em onMount -->


{#if state === 'playing'}
<div class="game-header">
    <a class="menu" href="/">Voltar ao Menu</a>
    <h1 class="timer" class:pulse={timeLeft <= 10}>{timeLeft}</h1>
</div>
<div class="game-content">
    <div class="cards">
        {#each grid as emoji, cardIndex}

        {@const isSelected = selected.includes(cardIndex)}
        {@const isMatch = matches.includes(emoji)}
        {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(emoji)}
        

        <button 
            class:selected = {isSelected}
            class:match = {isMatch}
            class:flip = {isSelectedOrMatched && !isMatch}
            disabled = {isSelectedOrMatched}
            on:click={() => selectCard(cardIndex)} class="card">
            <div class="back"><img src={"/images/" + emoji} alt={emoji} /></div>
        </button>
        {/each}
    </div>
</div>
{/if}

{#if state === 'lost'}
    <h1>Womp womp! Better luck next time</h1>
    <button on:click={() => state = 'playing'}>
        Jogar denovo
    </button>
{/if}

{#if state === 'won'}
    <h1>Você ganhou!</h1>
    <button on:click={() => state = 'playing'}>
        Jogar denovo
    </button>
{/if}
