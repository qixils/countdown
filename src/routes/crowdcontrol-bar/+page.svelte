<script lang="ts">
    import { onMount, onDestroy } from 'svelte'
    import { page } from '$app/stores'

    const whitelistedSources = [
        "twitch-bits",
        //"paypal",
        //"stripe",
    ] as const

    const key: string | null = $page.url.searchParams.get('key')
    let amount: number = parseFloat($page.url.searchParams.get('amount') || '0.00')
    const goal: number = parseFloat($page.url.searchParams.get('goal') || '50.00')
    const split: number = parseFloat($page.url.searchParams.get('split') || '80') / 100
    const title: string | null = $page.url.searchParams.get('title')

    const wssUrl = `wss://pubsub.crowdcontrol.live/`
    let wss: WebSocket | undefined

    function onOpen(event: Event) {
        if (!wss) return

        const subscribe = {
            action: "subscribe",
            data: JSON.stringify({
                key,
                topics: [
                    "prv/self",
                    "pub/self",
                    "overlay/self",
                ],
            }),
        }
        wss.send(JSON.stringify(subscribe))
    }

    function onMessage(event: MessageEvent<any>) {
        const { data } = event
        if (typeof data !== 'string') return
        if (data === 'pong') return

        const message = JSON.parse(data)
        //console.log(message)

        if (message.domain !== 'prv') return
        if (message.type === 'coin-exchange' && whitelistedSources.includes(message.payload.source)) {
            amount += message.payload.amount / 100 * (message.payload.source === 'twitch-bits' ? .8 : split)
        }
        else if (message.type === 'effect-success' && message.payload.payments) {
            amount += message.payload.payments.global.paid / 100 * split
        }
    }

    function onClose(event: Event) {
        wss = undefined
    }

    onMount(() => {
        if (!key) return

        wss = new WebSocket(wssUrl)
        wss.addEventListener('open', onOpen)
        wss.addEventListener('message', onMessage)
        wss.addEventListener('close', onClose)
        wss.addEventListener('error', onClose)
    })

    onDestroy(() => {
        if (!wss) return

        wss.removeEventListener('open', onOpen)
        wss.removeEventListener('message', onMessage)
        wss.removeEventListener('close', onClose)
        wss.removeEventListener('error', onClose)
        wss = undefined
    })

    function money(amt: number) {
        return amt.toLocaleString(undefined, { style: 'currency', currency: 'USD' })
    }

    function percent(amt: number) {
        return amt.toLocaleString(undefined, { style: 'percent', maximumFractionDigits: 0 })
    }
</script>

<svelte:head>
    <link href="https://fonts.googleapis.com/css2?family=Lato&display=swap" rel="stylesheet">
    <title>Crowd Control Goal Bar</title>
    <meta property="og:title" content="Crowd Control Goal Bar">
</svelte:head>

<main>
    {#if !key}
        <p>Please specify your overlay key in the URL</p>
    {:else}
        {#if title}
            <p>{ title }</p>
        {/if}
        <div class="bar" style="--pct: {amount/goal * 100}%">
            <p>{ money(amount) } ({ percent(amount/goal) })</p>
        </div>
        <div class="totals">
            <p>{ money(0) }</p>
            <!-- <p>{ money(amount) }</p> -->
            <p>{ money(goal) }</p>
        </div>
    {/if}
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        text-align: center;
        height: 100vh;
        width: 100vw;
        margin: 0;
        padding: 0;
        font-family: 'Lato', sans-serif;
        font-size: 18pt;
    }

    .bar {
        --pct: 0%;
        width: 100%;
        background: linear-gradient(to right, #5ccc5a 0 var(--pct), white var(--pct) 100%);
        color: black;
        font-weight: bold;
    }

    .bar p {
        filter: drop-shadow(0 0 0.01rem white);
    }

    .totals {
        width: 100%;
        display: flex;
        flex-flow: row nowrap;
        justify-content: space-between;
    }

    p {
        margin: 0.25rem;
    }
</style>
