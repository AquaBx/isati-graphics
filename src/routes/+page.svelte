<script lang="ts">
    import { onMount } from "svelte";

    let title = $state("Grand titre")
    let texte = $state("#Titre\nsalut ceci est une démonstration")
    let variante:string = $state(undefined)
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

        private offsetTop:number
        private offsetLeft:number
        private background:Promise<HTMLImageElement>
        private image:Promise<HTMLImageElement>|undefined
        private title:string
        private texte:string
        private ctx:CanvasRenderingContext2D
        private canvas:HTMLCanvasElement

        private imageHeight = 500
        private padding = 50

        constructor(backgroundURL:string,imageFiles:FileList|undefined,offsetLeft:number,offsetTop:number,texte:string,title:string){
            this.canvas = document.createElement("canvas")
            this.canvas.height = 1440
            this.canvas.width = 1440
            this.ctx = this.canvas.getContext('2d')!

            this.texte = texte
            this.title = title

            this.offsetLeft = offsetLeft
            this.offsetTop = offsetTop

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

        private async drawBackground(){
            let bg = await this.background
            this.ctx.drawImage(bg, 0, 0, 1440, 1440);
        }

        private drawTitle() {
            this.ctx.font = "900 120px LeagueSpartan";
            this.ctx.textAlign = "left";
            this.ctx.fillStyle = "#D82B2B"
            
            this.ctx.fillText(this.title,this.offsetLeft + 40,this.offsetTop)
            this.offsetTop += 120*1.25
        }

        private async drawImage(){

            let img = await this.image

            if (img === undefined) {return}

            this.ctx.drawImage(img, 1440/2 - img.width/2, this.offsetTop, img.width, img.height);
            this.offsetTop += this.imageHeight
        }

        private drawTexte(){
            this.ctx.textAlign = "left";

            let p = this.texte.split("\n")

            let lineheight : number

            for (let l of p){
                if (l.startsWith("#")){
                    this.ctx.font = "600 80px LeagueSpartan";
                    this.ctx.fillStyle = "#262626"
                    lineheight = 60*1.25

                    this.ctx.fillText(l.slice(1), this.offsetLeft, this.offsetTop);
                }
                else {
                    this.ctx.font = "500 60px LeagueSpartan";
                    this.ctx.fillStyle = "#262626"
                    lineheight = 40*1.25

                    this.ctx.fillText(l, this.offsetLeft, this.offsetTop);
                }

                this.offsetTop += lineheight*1.25
            }
        }

        async draw(destctx : CanvasRenderingContext2D) {
            this.ctx.clearRect(0, 0, 1440, 1440);

            await this.drawBackground()
            this.drawTitle()
            // this.offsetTop += this.padding
            this.drawTexte()
            // this.offsetTop += this.padding
            await this.drawImage()

            destctx.drawImage(this.canvas, 0, 0);

        }

    }


    $effect(() => {

        if (variante == "1"){
            template = new Template("./fond_idd_1.png",files,200,250,texte,title)
            template.draw(ctx)
        }
        else if (variante == "2"){
            template = new Template("./fond_idd_2.png",files,200,250,texte,title)
            template.draw(ctx)
        }
        else if (variante == "3") {
            template = new Template("./fond_idd_3.png",files,200,250,texte,title)
            template.draw(ctx)
        }

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
            <option value="1" selected={true}>Fond 1</option>
            <option value="2">Fond 2</option>
            <option value="3">Fond 3</option>
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