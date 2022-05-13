<script>
    import Quagga from '@ericblade/quagga2';
    import { onMount, onDestroy } from 'svelte';

    let quagga
    let started = false
    let code = '[scan result]'
    let detected = false

    onMount(init);
    onDestroy(destroy);

    function init() {
        Quagga.init({
            inputStream: {
                name: 'Live',
                type: 'LiveStream',
                target: quagga,
                constraints: {
                    facingMode: 'environment',
                },
            },
            decoder: { readers: ['code_128_reader'] },
        },
        function(error) {
            if (error) {
                console.warn(error);
                return;
            }
            Quagga.onProcessed((data) => {      
                console.log('onProcessed', data)
            });
            Quagga.onDetected((data) => {
                console.log('onDetected', data)
                code = data.codeResult?.code
                flash()
            });
            start();
        }
    )}

    function destroy() {
        Quagga.offProcessed();
        Quagga.offDetected();
    }

    async function start() {
        await Quagga.start();
        started = true;

    }

    async function stop() {
        await Quagga.stop();
        started = false;
    }

    function flash() {
        detected = true
        setTimeout(() => { detected = false }, 500)
    }
</script>
<header class:detected>{code}</header>    
<main>
    <quagga bind:this={quagga} />
</main>
<footer>
    {#if started}
        <button on:click={stop}>stop</button>
    {:else}
        <button on:click={start}>start</button>
    {/if}
</footer>
<style>
    header, footer {
        position: fixed;
        background-color: #3337;
        width: 100vw; 
        height: 80px;
        display: grid;
        place-content: center;
    }
    header { 
        top: 0;
        color: white;
        font-size: 3rem;
        font-family: monospace;
    }
    .detected {
        color: yellowgreen;
        font-weight: bold;
    }
    footer {
        bottom: 0;
    }
    main {
        display: grid;
    }
    quagga {
        width: 100vw;
        height: 100vh;
    }
</style>
