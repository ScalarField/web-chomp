<script context="module">

    import { writable } from 'svelte/store';

    export const gameInProgress = writable(0);
    export const gameStates = {
        BEFORE_START: 0,
        IN_PROGRESS: 1,
        GAME_ENDED: 2
    }

</script>

<script>

    import { tick } from 'svelte';
    import { fade, fly } from 'svelte/transition';
    import { players } from './const.js';

    export let rows;
    export let cols;
    export let firstPlayer;

    export let serverURL;

    const TILE_DELAY = 100;
    const FADE_DURATION = 100;
    const CPU_TURN_DELAY = 500;

    let xOffset = 0;
    let yOffset = 0;

    $: maxDim = Math.max(rows, cols);
    $: CELL_SIZE = 90/maxDim;
    $: CELL_SPACING = 10/(maxDim+1);
    $: BORDER_RADIUS = CELL_SPACING;
    $: if(maxDim === cols){
        xOffset = CELL_SPACING;
        yOffset = 0.5 * (100 - rows*CELL_SIZE - (rows-1)*CELL_SPACING);
    } else if(maxDim === rows){
        yOffset = CELL_SPACING;
        xOffset = 0.5 * (100 - cols*CELL_SIZE - (cols-1)*CELL_SPACING);
    }

    let startRow = 0;
    let startCol = 0;
    let transitioning = 0;

    let chompBoard;
    let lastMoveInfo;

    const initBoard = (rows, cols) => {
        chompBoard = Array.from(
            {length: rows}, 
            (v) => Array.from(
                {length: cols},
                (w) => ({ active: true, delay: 0})
            )
        );
        lastMoveInfo = {
            delay: 0,
            lastPlayer: firstPlayer
        }
    };

    const randomMove = (posString) => {

        const col = Math.floor(Math.random()*(posString.length-1));
        const colSize = parseInt(posString.charAt(col), 16);

        let row = 0;
        if(col < posString.length-1){
            row = Math.floor(Math.random()*colSize);
        } else if(colSize > 1) {
            row = 1+Math.floor(Math.random()*(colSize-1));
        } else {
            row = 0;
        }

        return {row, col: posString.length-col-1};

    };

    const getNextMove = async (row, col) => {
        let position = Array.from({length: 15}, (v) => 0);
        for(let r = 0; r < rows; r++){
            for(let c = 0; c < cols; c++){
                if(chompBoard[r][c].active 
                    && (r < row || c < col)){
                    position[14-c]++;
                }
            }
        }

        const posString = position
            .map((n) => n.toString(16))
            .join("")
            .replace(/[0]+/g, "");

        const url = `${serverURL}0x${posString}`;
        let useRandomMove = false;
        const response = await fetch(url)
            .then((res) => res.json())
            .then((res) => res.response)
            .catch((err) => { useRandomMove = true; });

        if(response?.code !== 0){ 
            useRandomMove = true;
        }
        
        if(useRandomMove){
            return randomMove(posString);
        }

        return { row: response.row, col: response.col }
        
    };

    const playMove = (row, col, cpuRow, cpuCol) => {
        let tileDelay = 0;
        for(let r = row; r < rows; r++){
            for(let c = col; c < cols; c++){
                if(chompBoard[r][c].active){

                    chompBoard[r][c].active = false;

                    const delay = (r+c-row-col) * TILE_DELAY;
                    if(delay > tileDelay){
                        tileDelay = delay;
                    }

                    transitioning++;
                    chompBoard[r][c].delay = delay;
                }
            }
        }

        if( cpuRow === -1 || cpuCol === -1){ 
            return {
                delay: tileDelay + FADE_DURATION,
                lastPlayer: players.PLAYER
            };
        }

        const offset = tileDelay + FADE_DURATION + CPU_TURN_DELAY;
        for(let r = cpuRow; r < rows; r++){
            for(let c = cpuCol; c < cols; c++){
                if(chompBoard[r][c].active){

                    chompBoard[r][c].active = false;

                    const delay = offset + (r+c-cpuRow-cpuCol)*TILE_DELAY;
                    if(delay > tileDelay){
                        tileDelay = delay;
                    }

                    transitioning++;
                    chompBoard[r][c].delay = delay;

                }
            }
        }

        return {
            delay: tileDelay + FADE_DURATION,
            lastPlayer: players.CPU
        };
    };

    const gameStateChange = async (gameInProgress) => {
        switch(gameInProgress){
            case gameStates.BEFORE_START:
                initBoard(rows, cols);
                break;
            case gameStates.IN_PROGRESS:
                initBoard(rows, cols);
                if(firstPlayer == players.CPU){
                    const {row, col} = await getNextMove(rows, cols);
                    playMove(row, col, -1, -1);
                }
                break;  
            default: break;
        }
    };

    const handleTurn = async (row, col) => {
        if($gameInProgress === gameStates.IN_PROGRESS
            && chompBoard[row][col].active 
            && transitioning === 0){

            let cpuMoveRow = -1;
            let cpuMoveCol = -1;

            if(row > 0 || col > 0){
                ({ 
                    row: cpuMoveRow, 
                    col: cpuMoveCol 
                } = await getNextMove(row, col));
            }

            lastMoveInfo = playMove(row, col, cpuMoveRow, cpuMoveCol);

            if( (row == 0 && col == 0) 
                || (cpuMoveRow == 0 && cpuMoveCol == 0) ){
                gameInProgress.set(gameStates.GAME_ENDED); 
            }
        }
    };

    $: initBoard(rows, cols);
    $: gameStateChange($gameInProgress);

</script>

<style>

    svg {
        margin: auto auto;
    }

    .svg-text {
        fill: var(--primary-3);
    }

    .button-rect {
        fill: var(--primary-2);
        cursor: pointer;
    }

    .button-rect:hover {
        fill: var(--primary-1);
    }

    .win-backdrop {
        fill: var(--bg-color-3);
    }

    .cell:hover {
        opacity: 0.8;
    }

</style>

<div class="container">
    <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
        {#each chompBoard as row, r}
        {#each chompBoard[r] as item, c}
        {#if item.active}
            <rect 
                on:click={ () => handleTurn(r, c) }
                x={ c * (CELL_SPACING + CELL_SIZE) + xOffset }
                y={ r * (CELL_SPACING + CELL_SIZE) + yOffset }
                width={ CELL_SIZE }
                height={ CELL_SIZE }
                rx={ BORDER_RADIUS }
                fill={`rgb(${200+55*r/rows}, ${200+55*c/cols}, ${255-55*c/cols})`}
                class="cell" 
                out:fade={{
                    delay: item.delay,
                    duration: 500
                }}
                on:outroend={ () => --transitioning }
                />
        {/if}
        {/each}
        {/each}
        {#if $gameInProgress === gameStates.GAME_ENDED}
            <g
                class="gameOverBox"
                in:fly={{
                    duration: 500,
                    delay: lastMoveInfo.delay + 1000,
                    y: 50
                }}
                out:fade={{ duration: 100 }}
                >
                <rect 
                    x="10" 
                    y="10" 
                    rx="10"
                    width="80" 
                    height="60" 
                    class="win-backdrop"/>
                <text
                    class="svg-text"
                    text-anchor="middle"
                    x="50"
                    y="30"
                    font-size="10"
                    >
                    You {
                        lastMoveInfo.lastPlayer === players.PLAYER
                        ? "lost!"
                        : "won!"
                    }
                </text>
                <g 
                    on:click={
                        () => gameInProgress.set(gameStates.IN_PROGRESS)
                    } 
                    class="button-rect"
                    >
                    <rect
                        x="20"
                        y="40"
                        rx="5"
                        width="20"
                        height="20"
                        />
                    <image 
                        href="restart.svg"
                        x="20"
                        y="40"
                        width="20"
                        height="20"
                        />
                </g>
                <g 
                    on:click={
                        () => gameInProgress.set(gameStates.BEFORE_START)
                    } 
                    class="button-rect"
                    >
                    <rect
                        x="60"
                        y="40"
                        rx="5"
                        width="20"
                        height="20"
                        />
                    <image
                        href="edit.svg"
                        x="62.5"
                        y="42.5"
                        width="15"
                        height="15"
                        />
                </g>
            </g>
        {/if}
    </svg>
</div>
