<template>
    <section ref="element" class="Home">
        <canvas id="canvasBar"></canvas>
    </section>
</template>

<script>
import * as PIXI from "pixi.js";
import Stats from "stats.js";
import Noise from "src/foo/utils/Noise";
const stats = new Stats();

export default {
    name: 'Home',
    data() {
        return {
            applyFilterContainer: new PIXI.Container(),
            audioAnalyser: undefined,
            audioContext: undefined,
            audioFrequencySource: [],
            audioPlaying: false,
            backgroundAudio: undefined,
            backroundImage: "https://images.pexels.com/photos/213399/pexels-photo-213399.jpeg?auto=compress&cs=tinysrgb&dpr=3&h=750&w=1260",
            barsCanvas: undefined,
            barsCanvasContext: undefined,
            canvasElement: undefined,
            canvasContext: undefined,
            displacementFilter: null,
            displacementImage: "https://raw.githubusercontent.com/Pierrinho/elephant/master/pattern-clouds.jpg",
            displacementSpeed: [0, 5],
            displacementSprite: null,
            menuItems: [
                {
                    title: "Listening Room"
                },
                {
                    title: "Sience"
                },
                {
                    title: "Technology"
                },
                {
                    title: "About"
                }
            ],
            move: {
                x: 0,
                y: 0
            },
            renderer: null,
            stage: new PIXI.Container(),
            stageHeight: 0,
            stageWidth: 0,
            ticker: new PIXI.ticker.Ticker(),
        }
    },
    methods: {
        createText(text, idx) {
            const { displacementFilter, stageHeight, stageWidth, stage } = this;
            const style = new PIXI.TextStyle({
                align: "right",
                fill: "#FFF",
                fontFamily: "Canela",
                fontSize: 96,
            });
            const textObject = new PIXI.Text(text, style);
            textObject.interactive = true;
            textObject.buttonMode = true;
            textObject.anchor.set(1, 1);
            textObject.x = stageWidth - 180;
            textObject.y = (stageHeight / 2) + (idx * textObject.height * 1.2);
            textObject.alpha = 0.1;
            textObject.mouseover = function() {
                textObject.filters = [ displacementFilter ];
                textObject.alpha = 1;
            };
            textObject.mouseout = () => {
                textObject.alpha = 0.1
                textObject.filters = [];
            };
            stage.addChild(textObject);
        },
        drawBars() {
            const { canvasContext: context, barsCanvas, barsCanvasContext, audioFrequencySource } = this;
            const { innerWidth, innerHeight } = window;
            barsCanvasContext.clearRect(0, 0, window.innerWidth, window.innerHeight);
            context.clearRect(0, 0, window.innerWidth, window.innerHeight);
            for (
                let i = 0,
                    ctx = barsCanvasContext,
                    fs = audioFrequencySource,
                    h = innerHeight,
                    w = innerWidth,
                    m = 500;
                 i < m;
                 i++
            ) {
                ctx.fillStyle = "#f81fa9";
                ctx.fillRect(((i * (w / m))) - 8, h - ((fs[i] || 1) / 2), (w / (m - 200)) + 2, h);
            }
            context.save();
            context.filter = "opacity(20%) blur(130px)";
            context.drawImage(barsCanvas, 0, 0);
            context.restore();
            context.save();
            context.filter = `blur(100px)`;
            // context.globalCompisteOperation = "lighter";
            context.drawImage(barsCanvas, 0, 0);
            context.restore();

            context.save();
            context.filter = 'blur(50px)';
            // context.globalCompisteOperation = "lighter";
            context.drawImage(barsCanvas, 0, 0);
            context.restore();
        },
        everyTick(delta) {
            stats.begin();
            const { audioPlaying, displacementSpeed, displacementSprite, getAudioFrequency, renderer, stage } = this;
            displacementSprite.x += displacementSpeed[0] * delta;
            displacementSprite.y += displacementSpeed[1] * delta;
            if (audioPlaying) {
                getAudioFrequency();
            }
            this.drawBars();
            renderer.render(stage);
            stats.end();
        },
        getAudioFrequency(){
            const analyser = this.audioAnalyser;
            analyser.getByteFrequencyData(this.audioFrequencySource);
        },
        init(){
            this.setTicker();
            this.setStage();
            // this.setBackground();
            this.setText();
        },
        setAudio() {
            const asset = require("src/assets/media/ocean.mp3");
            const audio = new Audio();
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const analyser = audioContext.createAnalyser();
            audio.addEventListener("playing", () => {
                this.audioPlaying = true;
                const frequencySource = new Uint8Array(this.audioAnalyser.frequencyBinCount);
                this.audioFrequencySource = frequencySource;
            });
            audio.src = asset;
            audio.controls = false;
            audio.loop = true;
            analyser.connect(audioContext.destination);
            analyser.smoothingTimeConstat = 0.65;
            analyser.fttSize = 8192;
            const audioSrc = audioContext.createMediaElementSource(audio);
            audioSrc.connect(analyser);
            this.backgroundAudio = audio;
            this.audioContext = audioContext;
            this.audioAnalyser = analyser;
        },
        setBackground() {
            const { applyFilterContainer, backroundImage, displacementFilter, stageHeight, stageWidth } = this;
            // eslint-disable-next-line
            const texture = new PIXI.Texture.fromImage(backroundImage);
            const img = new PIXI.Sprite(texture);
            img.anchor.set(0.5);
            img.x = stageWidth / 2;
            img.y = stageHeight / 2;
            img.filters = [ displacementFilter ];
            applyFilterContainer.addChild(img);
        },
        setCanvas() {
            const barsCanvas = document.createElement("canvas");
            const barsCanvasContext = barsCanvas.getContext("2d");
            const canvas = document.getElementById("canvasBar");
            const ctx = canvas.getContext("2d");
            canvas.width = barsCanvas.width = window.innerWidth;
            canvas.height = barsCanvas.height = window.innerHeight;
            this.barsCanvas = barsCanvas;
            this.barsCanvasContext = barsCanvasContext;
            this.canvasElement = canvas;
            this.canvasContext = ctx;
        },
        setDisplacementSprite() {
            const { displacementSprite, stageHeight, stageWidth, } = this;
            displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
            displacementSprite.anchor.set(0.5);
            displacementSprite.x = stageWidth / 2;
            displacementSprite.y = stageHeight / 2;
            displacementSprite.scale.x = 2;
            displacementSprite.scale.y = 2;
        },
        setRenderer() {
            const { renderer: { view: { style } } } = this;
            style.objectFit = "cover";
            style.height = "100%";
            style.width = "100%";
            style.top = "50%";
            style.left = "50%";
            style.webkitTransform = "translate(-50%, -50%)";
            style.transform = "translate(-50%, -50%)";
            //style.background = "#000";
        },
        setStage() {
            const { applyFilterContainer, displacementSprite, stage } = this;
            stage.addChild(applyFilterContainer);
            this.$refs.element.appendChild(this.renderer.view);
            stage.interactive = true;
            this.setRenderer();
            this.setDisplacementSprite();
            stage.on("click", () => {
                this.audioContext.resume();
                this.backgroundAudio.play();
            });
            stage.addChild(displacementSprite);
        },
        setStats() {
            document.body.appendChild(stats.domElement);
        },
        setText() {
            const { menuItems } = this;
            menuItems.forEach((item, idx) => {
                this.createText(item.title, idx);
            });
        },
        setTicker(){
            this.ticker.autoStart = true;
            this.ticker.add(this.everyTick);
        },
        setup(){
            const { innerHeight, innerWidth } = window;
            this.setAudio();
            this.stageHeight = innerHeight;
            this.stageWidth = innerWidth;
            this.renderer = new PIXI.WebGLRenderer(this.stageWidth, this.stageHeight, { transparent: true });
            // eslint-disable-next-line
            this.displacementSprite = new PIXI.Sprite.fromImage(this.displacementImage);
            this.displacementFilter = new PIXI.filters.DisplacementFilter(this.displacementSprite);
        }
    },
    created(){
        this.setup();
    },
    mounted() {
        this.init();
        this.setCanvas();
        this.setStats();
    }
}
</script>

<style src="styles/views/Home.styl" lang="stylus"></style>
