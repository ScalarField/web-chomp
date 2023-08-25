<script>

    export let name = "Value";
    export let min = 1;
    export let max = 10;
    export let step = 1;
    export let value = 1;

</script>

<style>

.labeltext {
    display: block;
}

.slider {
    display: block;
}

</style>

<div>
    <label>
        <span class="labeltext">{name}: {value}</span>
        <input 
            class="slider input-accent"
            type="range" 
            min={min}
            max={max}
            bind:value={value}
            />
    </label>
</div>
