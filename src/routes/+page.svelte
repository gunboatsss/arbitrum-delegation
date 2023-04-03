<script>
    export let title = "SNX Ambassadors: Arbitrum Delegation";
    //external dependency
    import { ethers } from "ethers";
    import {
        chainId,
        contracts,
        defaultEvmStores,
        signerAddress,
        provider,
    } from "svelte-ethers-store";
    import {
        EthereumClient,
        w3mConnectors,
        w3mProvider,
    } from "@web3modal/ethereum";
    import { Web3Modal } from "@web3modal/html";
    import { configureChains, createClient, watchSigner, getProvider } from "@wagmi/core";
    import { arbitrum } from "@wagmi/core/chains";
    import { PUBLIC_WALLETCONNECT_CLOUD_ID as projectId } from "$env/static/public";
    const chains = [arbitrum];
    const { provider: providerConfig } = configureChains(chains, [
        w3mProvider({ projectId }),
    ]);
    const wagmiClient = createClient({
        autoConnect: true,
        connectors: w3mConnectors({ projectId, version: 1, chains }),
        provider: providerConfig,
    });
    const ethereumClient = new EthereumClient(wagmiClient, chains);
    const web3modal = new Web3Modal({ projectId }, ethereumClient);
    import { onMount } from "svelte";
    import { derived } from "svelte/store";
    //internal dependency
    import { ArbitrumTokenABI } from "$lib/abi.js";
    import Balance from "$lib/Balance.svelte";
    //constant
    const AMBASSADORS_ADDRESS = "0xf18E9799ef294079ec3C312f8940741f0A03d952";
    defaultEvmStores.attachContract(
        "ArbitrumToken",
        "0x912CE59144191C1204E64559FE8253a0e49E6548",
        ArbitrumTokenABI,
        false
    );
    let initalized = false;
    let correctChain = false, signedIn = false;
    chainId.subscribe(value => {
        correctChain = (value === arbitrum.id) && (value !== undefined)
    });
    signerAddress.subscribe(value => {
        signedIn = value !== undefined
        console.log($signerAddress);
    });
    onMount(() => {
        const unwatch = watchSigner({}, async (signer) => {
            if (signer !== null) {
                console.debug(signer);
                defaultEvmStores.setProvider(signer.provider);
            }
        });
        initalized = true;
    });
</script>

<svelte:head>
    <title>{title}</title>
</svelte:head>

<header class="wallet">
    <span><w3m-core-button /></span>
</header>
<div id="body" class="balance">
    <h1>{title}</h1>
    {#if signedIn && initalized}
        {#key correctChain}
            {#if !correctChain}
                <span>Please connect to Arbitrum One to continue</span>
            {:else}
                {#await $contracts.ArbitrumToken.balanceOf($signerAddress)}
                    <span>Loading Balance...</span>
                {:then balance}
                    <Balance label="Your Balance:" {balance} />
                {/await}
                <div class="action">
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
                </div>
            {/if}
        {/key}
    {:else}
        <span>Connect your wallet to continue</span>
    {/if}
    {#if initalized && correctChain}
            {#await $contracts.ArbitrumToken.getVotes(AMBASSADORS_ADDRESS)}
                ...
            {:then balance}
                <Balance label="Current Voting Power:" {balance} />
            {/await}
    {/if}
</div>
