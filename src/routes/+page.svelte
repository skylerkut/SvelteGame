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
            matches == matches.concat(grid[first]);
        }
        //Clear after 300 ms when 2 cards are selected
        setTimeout(() => selected=[], 300); 
    }

    function gameWon() {
        currentState = 'won';
    }

    $: selected.length === 2 && matchCards();
    $: maxMatches === matches.length && gameWon();
    $: console.log({State, selected, matches});

</script>

{#if currentState === 'start'}
    <h1>Matching Game</h1>
    <button on:click={() => currentState = 'playing'}>Play</button>
{/if}

{#if currentState === 'playing'}
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
            <button class="card" 
                    class:selectedCard={isSelected} 
                    disabled={isSelectedOrMatched}
                    on:click={() => selectCard(cardIndex)}>
                <div class:match>{card}</div>
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
     
        &.selectedCard {
            border: 4px solid var(--border);    
        }

        .match {
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

</style>



