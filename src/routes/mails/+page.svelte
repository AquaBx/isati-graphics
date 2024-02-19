<script lang="ts">
    import Button from "$lib/components/individuels/Button.svelte";
import { onMount } from "svelte";

    let rôle = $state("Responsable Audiovisuel")
    let personnes = $state("DAMIEN VAILLAND")

    let canvas : HTMLCanvasElement
    let ctx : CanvasRenderingContext2D;

    let template: Template

    onMount(()=>{

        ctx = canvas.getContext('2d')!;
        canvas.height = 1204
        canvas.width = 4815

    })

    class Template {

        private offsetTop:number
        private offsetLeft:number
        private background:Promise<HTMLImageElement>
        private image:Promise<HTMLImageElement>|undefined
        private title:string
        private texte:string
        private ctx:CanvasRenderingContext2D
        private canvas:HTMLCanvasElement
        private padding : number

        private imageHeight = 500

        constructor(backgroundURL:string,padding:number,imageFiles:FileList|undefined,offsetLeft:number,offsetTop:number,texte:string,title:string){
            this.canvas = document.createElement("canvas")
            this.canvas.height = 1204
            this.canvas.width = 4815
            this.ctx = this.canvas.getContext('2d')!

            this.texte = texte
            this.title = title
            this.padding = padding

            this.offsetLeft = offsetLeft
            this.offsetTop = offsetTop

            let background = new Image()
            background.width = 4815
            background.height = 1204
            background.src = backgroundURL;

            this.background = new Promise((resolve,reject) =>{
                background.onload = () => {
                    resolve(background)
                }
            })
        }

        private async drawBackground(){
            let bg = await this.background
            this.ctx.drawImage(bg, 0, 0, 4815, 1204);
        }

        private drawTitle() {
            this.ctx.font = "900 150px Nanami";
            this.ctx.textAlign = "right";
            this.ctx.fillStyle = "#D82B2B"
            
            this.ctx.fillText(this.title,this.offsetLeft-100,this.offsetTop)
            this.offsetTop += 150
        }


        private drawTexte(){
            this.ctx.textAlign = "right";

            let p = this.texte.split("\n")

            let fontSize = 225
            let fontWeight = 800
            let color = "#262626"
            this.ctx.fillStyle = color
            this.ctx.font = `${fontWeight} ${fontSize}px NanamiBlack`;

            for (let texte of p){

                
                this.ctx.fillText(texte, this.offsetLeft, this.offsetTop);
                this.offsetTop += fontSize

            }
        }

        async draw(destctx : CanvasRenderingContext2D) {
           
            await this.drawBackground()

            this.offsetTop = (1204 - (150 + (this.texte.split("\n").length-1)*225))/2

            this.drawTexte()
            this.drawTitle()
            
            destctx.beginPath()
            destctx.drawImage(this.canvas, 0, 0);


        }

    }


    $effect(() => {

        template = new Template("./templates/signature_template.png",50,undefined,4000,235,personnes,rôle);template.draw(ctx)

    })

    var download = function(){
        var link = document.createElement('a');
        link.download = 'mail.png';
        link.href = canvas.toDataURL()
        link.click();
    }

</script>
<div class="main">

    <div>
        <canvas bind:this={canvas}></canvas>
    </div>
    <form spellcheck="false">

        <label for="rôle">Rôle</label>
        <input id="rôle" bind:value={rôle}>
        
        <label for="personnes">Personnes</label>
        <textarea id="personnes" bind:value={personnes}></textarea>

    </form>
    <Button on:click={download}>Télécharger</Button>

</div>

<style>
    canvas {
        /* aspect-ratio: 1/1; */
        width: 100%;
    }

    .main {
        width : clamp(0px,100%,1000px);
        display: grid;
        margin:auto;
        grid-template-columns: repeat(1,1fr);
        gap:10px;
    }
    
</style>