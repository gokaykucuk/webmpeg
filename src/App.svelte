<script>
  import { onMount } from 'svelte';
  import { FFmpeg } from '@ffmpeg/ffmpeg';
  import { fetchFile, toBlobURL } from '@ffmpeg/util';
  
  let ffmpeg;
  let videoFile;
  let currentFrame;
  let videoElement;
  let canvasElement;
  let cropArea = { x: 0, y: 0, width: 0, height: 0 };
  let isCropping = false;
  let currentTime = 0;
  let duration = 0;
  
  // FFmpeg settings
  let settings = {
    format: 'mp4',
    videoCodec: 'libx264',
    audioCodec: 'aac',
    fps: 30,
    resolution: '1920x1080',
    videoBitrate: '2M',
    crf: 23,
    preset: 'medium',
    profile: 'main',
    pixelFormat: 'yuv420p',
    audioBitrate: '128k',
    audioRate: 44100,
    audioChannels: 2,
    volume: 256
  };

  onMount(async () => {
    ffmpeg = new FFmpeg();
    await ffmpeg.load({
      coreURL: await toBlobURL('/ffmpeg-core.js', 'text/javascript'),
      wasmURL: await toBlobURL('/ffmpeg-core.wasm', 'application/wasm')
    });
  });

  async function handleFileSelect(event) {
    videoFile = event.target.files[0];
    const videoUrl = URL.createObjectURL(videoFile);
    videoElement.src = videoUrl;
  }

  function updateFrame() {
    if (!videoElement || !canvasElement) return;
    
    const ctx = canvasElement.getContext('2d');
    canvasElement.width = videoElement.videoWidth;
    canvasElement.height = videoElement.videoHeight;
    ctx.drawImage(videoElement, 0, 0);
    currentFrame = canvasElement.toDataURL();
  }

  async function convertVideo() {
    if (!videoFile) return;
    
    try {
      await ffmpeg.writeFile('input.mp4', await fetchFile(videoFile));
      
      let command = [
        '-i', 'input.mp4',
        '-c:v', settings.videoCodec,
        '-c:a', settings.audioCodec,
        '-r', settings.fps.toString(),
        '-b:v', settings.videoBitrate,
        '-crf', settings.crf.toString(),
        '-preset', settings.preset,
        '-profile:v', settings.profile,
        '-pix_fmt', settings.pixelFormat
      ];

      if (isCropping) {
        command.push('-vf', `crop=${cropArea.width}:${cropArea.height}:${cropArea.x}:${cropArea.y}`);
      }

      command.push('output.mp4');
      
      await ffmpeg.exec(command);
      
      const data = await ffmpeg.readFile('output.mp4');
      const blob = new Blob([data], { type: 'video/mp4' });
      const url = URL.createObjectURL(blob);
      
      const a = document.createElement('a');
      a.href = url;
      a.download = 'converted-video.mp4';
      a.click();
    } catch (error) {
      console.error('Error during conversion:', error);
    }
  }
</script>

<main class="container mx-auto px-4 py-8">
  <div class="max-w-4xl mx-auto">
    <h1 class="text-3xl font-bold mb-8">FFmpeg Video Processor</h1>
    
    <div class="mb-8">
      <input
        type="file"
        accept="video/*"
        on:change={handleFileSelect}
        class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
      />
    </div>

    {#if videoFile}
      <div class="mb-8">
        <div class="relative">
          <canvas
            bind:this={canvasElement}
            class="w-full aspect-video bg-black"
          ></canvas>
          <video
            bind:this={videoElement}
            class="hidden"
            on:loadedmetadata={() => duration = videoElement.duration}
            on:timeupdate={updateFrame}
          ></video>
        </div>
        
        <input
          type="range"
          min="0"
          max={duration}
          bind:value={currentTime}
          class="w-full mt-4"
          on:input={() => videoElement.currentTime = currentTime}
        />
        
        <div class="flex justify-center gap-4 mt-4">
          <button
            class="px-4 py-2 bg-gray-200 rounded"
            on:click={() => videoElement.currentTime -= 1/settings.fps}
          >
            Previous Frame
          </button>
          <button
            class="px-4 py-2 bg-gray-200 rounded"
            on:click={() => videoElement.currentTime += 1/settings.fps}
          >
            Next Frame
          </button>
        </div>
      </div>

      <div class="grid grid-cols-2 gap-4 mb-8">
        <!-- FFmpeg Settings -->
        <div class="space-y-4">
          <h2 class="text-xl font-semibold">Video Settings</h2>
          <label class="block">
            Format
            <select bind:value={settings.format} class="block w-full mt-1 rounded border-gray-300">
              <option value="mp4">MP4</option>
              <option value="webm">WebM</option>
              <option value="mov">MOV</option>
            </select>
          </label>
          <!-- Add more video settings inputs -->
        </div>
        
        <div class="space-y-4">
          <h2 class="text-xl font-semibold">Audio Settings</h2>
          <!-- Add audio settings inputs -->
        </div>
      </div>

      <button
        class="w-full py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
        on:click={convertVideo}
      >
        Convert Video
      </button>
    {/if}
  </div>
</main>

<style lang="postcss">
  :global(html) {
    @apply bg-gray-50;
  }
</style> 