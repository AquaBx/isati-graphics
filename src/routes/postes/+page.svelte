<script lang="ts">
    import Button from "$lib/components/individuels/Button.svelte";
    import { Template2 } from "$lib/canvas";
    import type { configuration } from "$lib/canvas";

    let title = $state("Grand titre")
    let texte = $state("#Titre\nsalut ceci est une démonstration")
    let variante:string|undefined = $state(undefined)
    let files:FileList|undefined = $state()

    let canvas : HTMLCanvasElement|undefined = $state()

    let config:configuration
    let temp2:Template2

    $effect(() => {
        config ={
            backgroundURL:variante!,
            height:1440,
            width:1440,
            canvas:canvas!
        }
        temp2 = new Template2(config)
    })

    let formatting = (input:string) => {
        if (input.startsWith("#")){
            return {
                fontSize:80,
                fontWeight:700,
                color:"#262626",
                fontFamily:'LeagueSpartan',
                textAlign:"left",
                output:input.slice(1)
            }
        }
        else {
            return {
                fontSize:60,
                fontWeight:500,
                color:"#262626",
                fontFamily:'LeagueSpartan',
                textAlign:"left",
                output:input
            }
        }
    }

    $effect(() => {

        ( async (title,texte,files,variante) => {

            temp2.clear()
            await temp2.drawBackground()

            temp2.drawTexte(title,190,235,'LeagueSpartan',120,"900","#D82B2B","left")

            let y = temp2.drawFormattedTexte(texte,150,config.width-150,235+120+50,formatting)

            if (files) {
                // load une image 

                var fr = new FileReader();
                let image = new Image()

                fr.onload = function () {
                    image.src = fr.result as string
                }

                fr.readAsDataURL(files[0]);

                // on la fit à une dimension précise
                image.onload = function (){
                    let nheight = 500
                    let nwidth = image.width / image.height * nheight

                    temp2.drawImage(image,(config.width-nwidth)/2,y,nwidth,nheight)
                }

            }

        })(title,texte,files,variante)

    })

</script>
<div class="main">

    <div>
        <canvas bind:this={canvas}></canvas>
    </div>
    <form spellcheck="false">

        <label for="titre">Titre</label>
        <input name="titre" bind:value={title}>
        <label for="image">Image</label>
        <input name="image" type="file" bind:files={files}>


        <label for="texte">Contenu</label>
        <textarea name="texte" bind:value={texte} rows="11"  ></textarea>

        <label for="template">Template</label>

        <select name="template" bind:value={variante}>
            <option value="./demo.png">Fond Demo (ne pas utiliser, juste pour debug)</option>
            <option value="./templates/Template_1.png" selected={true}>Fond 1</option>
            <option value="./templates/Template_2.png">Fond 2</option>
            <option value="./templates/Template_3.png">Fond 3</option>
            <option value="./templates/Template_4.png">Fond 4</option>
        </select>

    </form>
    <Button on:click={() => temp2.download()}>Télécharger</Button>

</div>
<style>
    canvas {
        aspect-ratio: 1/1;
        width: 100%;
    }

    .main {
        width : clamp(0px,100%,1000px);
        display: grid;
        margin:auto;
        grid-template-columns: repeat(2,1fr);
        gap:10px;
    }
    
</style>