<script lang="ts">
    import { page } from '$app/stores';
    let font: string = $page.url.searchParams.get('font') || "Roboto+Mono";
    let size: string = $page.url.searchParams.get('size') || "6em";
    let color: string = $page.url.searchParams.get('color') || "black";
    let text: string | null = $page.url.searchParams.get('text');

    let to = parseInt($page.url.searchParams.get('to') || '0');
    if (to == 0) {
        to = Math.round((Date.now()/1000) + parseInt($page.url.searchParams.get('duration') || '0'));
    }

    function getDuration(): number {
        return Math.round(to - (Date.now()/1000));
    }

    let duration = getDuration();

    // TODO: this is dumb lol
    let seconds: number = 1;
    let minutes: number = 0;
    let hours: number = 0;
    let days: number = 0;
    let weeks: number = 0;
    if (duration >= 10) {
        seconds = 2;
        if (duration >= 60) {
            minutes = 1;
            if (duration >= 600) {
                minutes = 2;
                if (duration >= 3_600) {
                    hours = 1;
                    if (duration >= 36_000) {
                        hours = 2;
                        if (duration >= 86_400) {
                            days = 1;
                            if (duration >= 604_800) {
                                weeks = 1;
                                if (duration >= 6_048_000) {
                                    weeks = 2;
                                }
                            }
                        }
                    }
                }
            }
        }
    }


    function pad(num: number, to: number = 2): string {
        return num.toString().padStart(to, '0');
    }

    let formatted: string = '';
    $: {
        formatted = pad(duration%60, seconds);
        if (minutes > 0) {
            formatted = `${pad(Math.floor(duration/60)%60, minutes)}:${formatted}`;
            if (hours > 0) {
                formatted = `${pad(Math.floor(duration/3600)%24, hours)}:${formatted}`;
                if (days > 0) {
                    formatted = `${pad(Math.floor(duration/86400)%7, days)}:${formatted}`;
                    if (weeks > 0) {
                        formatted = `${pad(Math.floor(duration/604800), weeks)}:${formatted}`;
                    }
                }
            }
        }
    }

    setInterval(() => {
        duration = getDuration();
    }, 1000);
</script>

<svelte:head>
    <link href="https://fonts.googleapis.com/css2?family={font}&display=swap" rel="stylesheet">
    <title>Countdown Timer</title>
    <meta property="og:title" content="Countdown Timer">
    <meta property="og:description" content="A simple search parameter driven countdown timer.">
    <meta name="description" content="A simple search parameter driven countdown timer.">
</svelte:head>

<main style="font-size: {size}; font-family: '{font.replace('+', ' ')}', monospace; color: {color};">
    <div>
        {#if text != null}
            <p>{text}</p>
        {/if}
        {formatted}
    </div>
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        justify-content: center;
        text-align: center;
        height: 100vh;
        width: 100vw;
        margin: 0;
        padding: 0;
    }
</style>
