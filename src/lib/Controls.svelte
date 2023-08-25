<script context="module">

    import { writable } from 'svelte/store';                           

    export const settings = writable({
        rows: 10,
        cols: 10,
        firstPlayer: players.HUMAN
    });

</script>
<script>
    
    import { createEventDispatcher } from 'svelte';
    import { 
        MIN_BOARD_SIZE, 
        MAX_BOARD_SIZE, 
        players 
    } from './const.js';
    import { gameInProgress, gameStates } from './Chomp.svelte';

    import Slider from './ui/Slider.svelte';
    import RadioSelect from './ui/RadioSelect.svelte';

    export let disabled;

    const dispatch = createEventDispatcher();

    const startGame = () => {
        gameInProgress.set(gameStates.IN_PROGRESS);
    }

</script>

<style>

legend {
    font-size: 1.25rem;
    font-weight: bold;
    color: var(--primary-3);
}

.control-container {
    padding: 0px;
    display: flex;
    flex-flow: column nowrap;
    border: none;
}

.control-item {
    padding-left: 1rem;
    border: none;
}

.control-fieldset {
    border-top: 1px solid var(--primary-3);
}

.start-button-container {
    padding-top: 0.25rem;
}

.start-button {
    background-color: var(--primary-1);
    color: var(--text-color-dark);
    border: none;
    padding: 0.25rem 0.75rem;
    border-radius: 5px;
}

.start-button:hover {
    background-color: var(--primary-2);
}

.start-button:disabled {
    background-color: var(--bg-color-2);
}

</style>

<div>
    <form>
        <fieldset disabled={disabled} class="control-container">
            <fieldset class="grid-size control-item control-fieldset">
                <legend>Grid size</legend>
                <Slider 
                    name="Rows"
                    min={MIN_BOARD_SIZE}
                    max={MAX_BOARD_SIZE}
                    step={1}
                    bind:value={$settings.rows}
                />
                <Slider 
                    name="Columns"
                    min={MIN_BOARD_SIZE}
                    max={MAX_BOARD_SIZE}
                    step={1}
                    bind:value={$settings.cols}
                />
            </fieldset>
            <fieldset class="player-select control-item control-fieldset">
                <legend>First player</legend>
                <RadioSelect
                    name="playerone"
                    options={[players.HUMAN, players.CPU]}
                    bind:value={$settings.firstPlayer}
                    />
            </fieldset>
            <div class="start-button-container control-item">
                <button 
                    class="start-button pointer"
                    on:click={()=>{startGame();}}>
                    Start Game
                </button>
            </div>
        </fieldset>
    </form>
</div>
