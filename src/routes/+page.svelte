<script lang="ts">
    import { onMount } from "svelte";

    let title = $state("GRAND TITRE")
    let texte = $state("#Titre\nsalut ceci est une démonstration")
    let variante = $state("")
    let files:FileList|undefined = $state()

    let canvas : HTMLCanvasElement
    let ctx : CanvasRenderingContext2D;

    let template: Template

    onMount(()=>{

        ctx = canvas.getContext('2d')!;
        canvas.height = 1440
        canvas.width = 1440

    })

    class Template {

        private offset:number
        private background:Promise<HTMLImageElement>
        private image:Promise<HTMLImageElement>|undefined
        private title:string
        private texte:string

        private imageHeight = 400
        private padding = 150

        constructor(backgroundURL:string,imageFiles:FileList|undefined,offset:number,texte:string,title:string){

            this.texte = texte
            this.title = title

            this.offset = offset

            let background = new Image()
            background.width = 1440
            background.height = 1440
            background.src = backgroundURL;

            this.background = new Promise((resolve,reject) =>{
                background.onload = () => {
                    resolve(background)
                }
            })

            if (imageFiles){
                let ctx = this

                var fr= new FileReader();
                let image = new Image()

                fr.onload = function () {
                    image.src = fr.result as string
                }

                fr.readAsDataURL(imageFiles[0]);

                this.image = new Promise((resolve,reject) =>{
                    image.onload = function (){
                        image.width = this.width / this.height * ctx.imageHeight
                        image.height = ctx.imageHeight
                        resolve(image)
                    }
                })
            }
            else {
                this.image = undefined
            }

        }

        private async drawBackground(ctx:CanvasRenderingContext2D){
            let bg = await this.background
            ctx.drawImage(bg, 0, 0);
        }

        private drawTitle(ctx:CanvasRenderingContext2D) {
            ctx.font = "bold 130px LeagueSpartan";
            ctx.textAlign = "center";
            ctx.fillStyle = "#ffffff"
            
            this.offset += this.padding
            ctx.fillText(this.title,720,260)
        }

        private async drawImage(ctx:CanvasRenderingContext2D){

            let img = await this.image

            if (img === undefined) {return}

            ctx.drawImage(img, 200, this.offset, img.width, img.height);
            this.offset += this.imageHeight
        }

        private drawTexte(ctx:CanvasRenderingContext2D){
            ctx.textAlign = "left";

            let p = this.texte.split("\n")

            let lineheight : number

            for (let l of p){
                if (l.startsWith("#")){
                    ctx.font = "bold 90px LeagueSpartan";
                    ctx.fillStyle = "#ff0000"
                    lineheight = 90*1.25

                    ctx.fillText(l.slice(1), 200, this.offset);
                }
                else {
                    ctx.font = "bold 65px LeagueSpartan";
                    ctx.fillStyle = "#000000"
                    lineheight = 65*1.25

                    ctx.fillText(l, 200, this.offset);
                }

                this.offset += lineheight
            }
        }

        async draw(ctx : CanvasRenderingContext2D) {
            await this.drawBackground(ctx)
            await this.drawImage(ctx)
            this.drawTitle(ctx)
            this.drawTexte(ctx)
        }

    }


    $effect(() => {

        template = new Template("./template.png",files,400,texte,title)
        template.draw(ctx)

    })


</script>
<div class="main">

    <div>
        <canvas bind:this={canvas}></canvas>
    </div>
    <form>

        <label>Titre</label>
        <input bind:value={title}>
        <label>Image</label>
        <input type="file" bind:files={files}>


        <label>Contenu</label>
        <textarea bind:value={texte} cols="30" rows="10"></textarea>
        <label>Template</label>
        <select bind:value={variante}>

        </select>
    </form>

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
    
    
    form {
        gap:10px;
        display: flex;
        flex-direction: column;
    }

    label,input,select,textarea {
        font-size: 20px;
        color : var(--text);
    }
    input,select,textarea{
        padding:10px;
        outline: unset;
        border: unset;
        background-color: var(--container);
        box-shadow: var(--shadow);
        border-radius: 10px;
    }


    textarea{
        resize: vertical;
    }


</style>