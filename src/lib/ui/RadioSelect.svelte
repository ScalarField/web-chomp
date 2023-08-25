<script>

    export let name = "Radio buttons";
    export let options = ["one", "two"];
    export let value = options[0];

</script>

<style>

.container {
    display: flex;
    flex-flow: row wrap;
    gap: 0.25rem 1rem;
}

.label-container {
}


</style>

<div class="container">
    {#each options as option}
        <div class="label-container">
            <label>
                <input 
                    class="input-accent"
                    type="radio" 
                    name={name}
                    value={option}
                    bind:group={value} 
                    />
                {option}
            </label>
        </div>
    {/each}
</div>


