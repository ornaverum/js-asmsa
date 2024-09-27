<script lang='ts'>
    import jsfeat from 'jsfeat';
    import { onMount } from 'svelte';
    import {getGrayscaleImage, showMat} from '$lib/featscripts';
  
    let vidURL = '/Soccer.MOV';
    // let vidURL = '/Running.MOV';
    let video;
    let cnv: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let height;
    let width;
    let img_u8 = new jsfeat.matrix_t(width, height, jsfeat.U8_t | jsfeat.C1_t);
  
    onMount(async () => {
        cnv = document.querySelector('#video-output');
        video = document.querySelector('#video-input');
        video.src = vidURL;
        ctx = cnv.getContext('2d', { willReadFrequently: true });

        async function loadVideo() {
        return new Promise((resolve) => {
            if (video.readyState >= 1) {
            resolve(1);
            } else {
            video.addEventListener('loadedmetadata', () => {
                resolve(1);
            });
            }
        });
        }

        await loadVideo();

        height = video.videoHeight || 500;
        width = video.videoWidth || 500;

        let maxWidth = 500;       //window.innerWidth
        let maxHeight = 500;    //window.innerHeight 

        console.log(width, height);
        if (height > maxHeight || width > maxWidth) {
        let ratio = Math.min(maxHeight / height, maxWidth / width);
        height *= ratio;
        width *= ratio;
        }
        
        cnv.width = width;      
        cnv.height = height;
        
        let imageData = ctx.getImageData(0, 0, width, height);
        console.log(imageData);

        video.addEventListener('play', () => {
        tick();
        });

        let lastRender = 0;

        let gray_img = new jsfeat.matrix_t(width, height, jsfeat.U8_t | jsfeat.C1_t);

        function tick(timestamp = 0) {
        if (timestamp - lastRender > 1000 / 30) { // Throttling to 30 FPS
            lastRender = timestamp;
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                processFrame();
            }
        }
        if(!video.paused && !video.ended) {
            window.requestAnimationFrame(tick);
        }
        }

        function processFrame() {
            ctx.drawImage(video, 0, 0, width, height);
            imageData = ctx.getImageData(0, 0, width, height);
            let gray_img = getGrayscaleImage(imageData);
            showMat(gray_img, ctx, imageData)
        }
    });
</script>

<button on:click={() => video.play()}>play</button>
<button on:click={() => video.pause()}>pause</button>

<video id='video-input' controls style='display: none'></video>
<canvas id='video-output'></canvas>
