<script>
    
    import { players, CHOMP_URL } from '$lib/const.js';
    import Controls, { settings } from '$lib/Controls.svelte';
    import Chomp, { gameInProgress } from '$lib/Chomp.svelte';

</script>

<style>

header {
    text-align: center;
}

.rules {
    width: 80vw;
    margin: 0.5rem auto;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--text-color-light);
}

main {
}

.content {
    width: 80vw;
    margin: 0.5rem auto;
    display: flex;
    flex-flow: row wrap;
    justify-content: space-evenly;
    gap: 1rem;
    padding: 0.5rem 0rem 1rem 0rem;
    border-bottom: 1px solid var(--text-color-light);
}

.chomp-container {
    min-width: 18rem;
    max-width: min(50vw, 80vh);
    flex: 3;
}

.controls-container {
    min-width: 15rem;
    flex: 2;
}

</style>

<header> 
    <h1>Chomp</h1>
    <p class="rules">
        On your turn, click a square to remove it, as well as everything below and to the right of it. If you remove the top left square, you lose!
    </p>
</header>
<main>

    <div class="content">
        <div class="chomp-container">
            <Chomp 
                bind:rows={$settings.rows}
                bind:cols={$settings.cols}
                bind:firstPlayer={$settings.firstPlayer}
                serverURL={CHOMP_URL}
                />
        </div>

        <div class="controls-container">
            <Controls bind:disabled={$gameInProgress}/>
        </div>
    </div>

</main>
