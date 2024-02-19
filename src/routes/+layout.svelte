<script lang="ts">

    import Header from "$lib/components/individuels/Header.svelte"
    import { fade  } from 'svelte/transition';
    import Button from "$lib/components/individuels/Button.svelte";
    
    export let data

</script>


<svelte:head>
    <title>ISATI BDE</title>
    <meta name="description" content="">
    <link rel="stylesheet" href="./app.css">

</svelte:head>



<div class="app">

    <Header></Header>
    
    <div style="display:flex;justify-content: space-evenly;">
        <a href={"./mails"}><Button>Editeur Mails</Button></a>
        <a href={"./postes"}><Button>Editeur Postes</Button></a>
    </div>

    {#key data.pathname}
        <div id="content" in:fade={{ duration: 1000}}>
            <slot></slot>
        </div>
    {/key}
    
    <!-- <Footer></Footer> -->
    
</div>

<style>
    div.app{
        display: flex;
        flex-direction: column;
        gap:1em;
    }

    #content {
        overflow:auto;
        display: block;
        position: relative;
        padding: 16px;
        margin:0;
        width:calc( 100% - 32px);
        overflow: overlay;
    }

</style>