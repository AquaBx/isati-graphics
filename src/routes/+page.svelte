<script lang="ts">
    import Button from "$lib/components/individuels/Button.svelte";
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
        private padding : number

        private imageHeight = 500

        constructor(backgroundURL:string,padding:number,imageFiles:FileList|undefined,offsetLeft:number,offsetTop:number,texte:string,title:string){
            this.canvas = document.createElement("canvas")
            this.canvas.height = 1440
            this.canvas.width = 1440
            this.ctx = this.canvas.getContext('2d')!

            this.texte = texte
            this.title = title
            this.padding = padding

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
            this.offsetTop += 120
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

            for (let l of p){
                let fontSize
                let fontWeight
                let color
                
                if (l.startsWith("#")){
                    fontSize = 80
                    fontWeight = 700
                    color = "#262626"
                    l = l.slice(1)
                }
                else {
                    fontSize = 60
                    fontWeight = 500
                    color = "#262626"
                }

                this.ctx.fillStyle = color
                this.ctx.font = `${fontWeight} ${fontSize}px LeagueSpartan`;


                let l_liste = l.split(" ")
                let text = l_liste[0]
                let i = 1

                while (text != undefined) {
                    if (i == l_liste.length){
                        this.ctx.fillText(text, this.offsetLeft, this.offsetTop);
                        this.offsetTop += fontSize
                        break
                    }

                    let w = l_liste[i]
                    if (this.ctx.measureText(text + " " + w).width > 1140){
                        this.ctx.fillText(text, this.offsetLeft, this.offsetTop);
                        this.offsetTop += fontSize

                        text = w
                    }
                    else {
                        text += " " + w
                    }
                    i++
                }

            }
        }

        async draw(destctx : CanvasRenderingContext2D) {
           
            await this.drawBackground()
            this.drawTitle()
            this.offsetTop += this.padding
            this.drawTexte()
            // this.offsetTop += this.padding
            await this.drawImage()
            
            destctx.beginPath()
            destctx.drawImage(this.canvas, 0, 0);


        }

    }


    $effect(() => {

        switch (variante) {
            case "0" : template = new Template("./demo.png"                ,50,files,150,235,texte,title);template.draw(ctx);break;
            case "1" : template = new Template("./templates/Template_1.png",50,files,150,235,texte,title);template.draw(ctx);break;
            case "2" : template = new Template("./templates/Template_2.png",50,files,150,235,texte,title);template.draw(ctx);break;
            case "3" : template = new Template("./templates/Template_3.png",50,files,150,235,texte,title);template.draw(ctx);break;
            case "4" : template = new Template("./templates/Template_4.png",50,files,150,235,texte,title);template.draw(ctx);break;
        }

    })

    var download = function(){
        var link = document.createElement('a');
        link.download = 'idd.png';
        link.href = canvas.toDataURL()
        link.click();
    }

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
            <option value="0">Fond Demo (ne pas utiliser, juste pour debug)</option>
            <option value="1" selected={true}>Fond 1</option>
            <option value="2">Fond 2</option>
            <option value="3">Fond 3</option>
            <option value="4">Fond 4</option>
        </select>

    </form>
    <Button on:click={download}>Télécharger</Button>

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