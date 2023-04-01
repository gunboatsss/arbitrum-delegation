<script>
    export let title = "SNX Ambassadors: Arbitrum Delegation";

    import { ethers } from "ethers";
    import { ArbitrumTokenABI, TokenDistributorABI } from "$lib/abi.js";
    import {
        chainId,
        contracts,
        defaultEvmStores,
        signerAddress,
        provider
    } from "svelte-ethers-store";
    import { onMount } from "svelte";
    const ARBITRUM_ONE_CHAINID = 42161;
    const AMBASSADORS_ADDRESS = "0xf18E9799ef294079ec3C312f8940741f0A03d952";
    const unsub = 
    defaultEvmStores.attachContract(
        "ArbitrumToken",
        "0x912CE59144191C1204E64559FE8253a0e49E6548",
        ArbitrumTokenABI,
        false
    );
    $: signedIn = $signerAddress !== undefined;
    function reset() {
        defaultEvmStores.setProvider(
            new ethers.providers.InfuraProvider(ARBITRUM_ONE_CHAINID),
            null
        );
    }
    async function getVotes() {
        let power = await $contracts.ArbitrumToken.getVotes(
            AMBASSADORS_ADDRESS
        );
        return ethers.utils.formatEther(power);
    }
    onMount(() => {
        reset();
    });
</script>

<svelte:head>
    <title>{title}</title>
</svelte:head>

<div class="wallet">
    {#if !signedIn}
        <button
            on:click={() => {
                defaultEvmStores.setProvider();
            }}>Connect wallet</button
        >
    {:else}
        <span
            >{$signerAddress}
            <button
                on:click={() => {
                    reset();
                }}>Disconnect</button
            ></span
        >
    {/if}
</div>
<div class="balance">
    <h1>{title}</h1>
    {#if signedIn}
        {#key $chainId}
            {#if $chainId !== ARBITRUM_ONE_CHAINID}
                <span>Please connect to Arbitrum One to continue</span>
            {:else}
                <span
                    >Your balance: {#await $contracts.ArbitrumToken.balanceOf($signerAddress)}...{:then bal}{ethers.utils.formatEther(
                            bal
                        )}{/await}</span
                >
                {#await $contracts.ArbitrumToken.delegates($signerAddress)}
                    <span>...</span>
                {:then delegate}
                    {#if delegate === AMBASSADORS_ADDRESS}
                        <span
                            >You are delegated to SNX Ambassadors, Thank you for
                            your support!</span
                        >
                    {:else}
                        <span>
                            You are delegated to: {delegate}
                            <button
                                on:click={() => {
                                    $contracts.ArbitrumToken.delegate(
                                        AMBASSADORS_ADDRESS
                                    );
                                }}>Delegate to Ambassadors</button
                            >
                        </span>
                    {/if}
                {/await}
            {/if}
        {/key}
    {:else}
        <span>Connect your wallet to continue</span>
    {/if}
    <p>
        Current Ambassadors Power:
        {#await getVotes()}
            ...
        {:then power}
            {power}
        {/await}
    </p>
</div>
