<script>
    import { emoji } from "./emoji";
    import "./app.css"

    const State =  'start' | 'playing' | 'paused' | 'won' | 'lost'; 

    let currentState = 'start'; 
    let size = 20;
    let grid = createGrid();
    let maxMatches =  grid.length / 2;
    let selected = [];
    let matches = [];

    let timerID = null;
    let time = 60;

    function startGameTimer() {
        function countDown(){
            currentState != 'paused' && (time -= 1);
        }
        timerID = setInterval(countDown, 1000);
    }

    function createGrid() {
        let cards = new Set();
        let maxSize = size / 2; //doubles of each card

        //Add emojis to cards 
        while(cards.size < maxSize){
            const randomIndex = Math.floor(Math.random() * emoji.length);
            cards.add(emoji[randomIndex]);
        }
        //Return new deck of cards
        return shuffle([...cards, ...cards]);
    }

    function shuffle(items){
        return items.sort(() => Math.random() - 0.5);
    }

    function selectCard(cardIndex){
        selected = selected.concat(cardIndex); //Dont need to spread because its more expensive
    }

    function matchCards(){
        const [firstCard, secondCard] = selected;

        if(grid[firstCard] == grid[secondCard]){
            matches = matches.concat(grid[firstCard]);
        }
        //Clear after 300 ms when 2 cards are selected
        setTimeout(() => selected = [], 300); 
    }

    function resetGame() {
        timerID && clearInterval(timerID);
        grid = createGrid();
        maxMatches = grid.length / 2;
        selected = [];
        matches = [];
        timerID = null;
        time = 20;
    }

    function gameWon() {
        currentState = 'won';
        resetGame();
    }

    function gameLost() {
        currentState = 'lost';
        resetGame();
    }

    function pauseGame(e) {
        if(e.key === 'Escape'){
            switch(currentState){
                case 'playing': 
                    currentState = 'paused';
                    break;
                case 'paused':
                    currentState = 'playing';
                    break;
            }
        }
    }

    $: if (currentState === 'playing'){
        !timerID && startGameTimer();
    }

    $: selected.length === 2 && matchCards();
    $: maxMatches === matches.length && gameWon();
    $: time === 0 && gameLost();

    $: console.log(currentState, selected, matches);

</script>

<svelte:window on:keydown={pauseGame}></svelte:window>

{#if currentState === 'paused'}
    <h1>Game Paused</h1>
{/if}

{#if currentState === 'start'}
    <h1>Matching Game</h1>
    <button on:click={() => currentState = 'playing'}>Play</button>
{/if}

{#if currentState === 'playing'}
<h1 class="timer" class:pulse={ time<=10 }>{time}</h1>

    <div class="matches">
        {#each matches as card}
            <div>{card}</div>    
        {/each}
    </div>
    
    <div class="cards">
        {#each grid as card, cardIndex}
            <!--Constants-->
            {@const isSelected = selected.includes(cardIndex)}
            {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
            {@const match = matches.includes(card)}

            <button class="card" 
                    class:flip={isSelectedOrMatched}
                    class:selectedCard={isSelected} 
                    disabled={isSelectedOrMatched}
                    on:click={() => selectCard(cardIndex)}>
                <div class="back" class:match>{card}</div>
            </button>
        {/each}
    </div>
{/if}

{#if currentState === 'lost'}
    <h1>You Lost</h1>
    <button on:click={() => currentState = 'playing'}>Play Again</button>
{/if}

{#if currentState === 'won'}
    <h1>You Win!</h1>
    <button on:click={() => currentState = 'playing'}>Play Again</button>  
{/if}
<!--Boolean states can cause impossible states, use explict states-->

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

        &.selectedCard {
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
            place-content: center;
            backface-visibility: hidden;
            rotate: y 180deg;
        }

        & .match {
            transition: opacity 0.3s ease-out;
            opacity: 0.4;
        }
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



