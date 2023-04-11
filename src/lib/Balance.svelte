<script>
    import { utils } from "ethers";
    /**
     * @type {import('ethers').BigNumber} balance
     */
    export let balance;
    /**
     * @type {string} label
     */
    export let label;
    $: remainder = balance.mod(1e14);
    $: display = utils.formatEther(balance.sub(remainder));
</script>

<div>
    <label for="balance">{label}</label>
    <input name="balance" value={display} readonly />
</div>

<style>
    div {
        max-width: 640px;
        flex-direction: column;
        display: inline-flex;
        margin: 0 auto 5px;
    }
    label {
        font-size: 0.9em;
        text-align: left;
    }
    input {
        text-align: center;
        padding-right: 15px;    
        text-align: right;
        font-family: monospace;
        font-size: 1.5em;
        border-radius: 25px;
    }
</style>
