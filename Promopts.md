# Prompt

Build a client side spa application that is an ui wrapper around ffmpeg. Use Svelte, Tailwindcss and Ffmpeg. Make a simple and modern looking user interface which is mainly one file input for input video.

An image element will have an image that shows selected frame. Below that a draggable scrollbar that lets you either drag and seek the video, or some buttons below that lets the user go to next, previous frames.

On the preview screen also let the user crop the video. When the user crops video using cropping tool on preview image, Ffmpeg crops the video at that resolution.

The user interface includes all the options for configuring ffmpeg.

Input/Output

-f format: Force input/output format
-c:v codec: Set video codec (e.g., libx264, libvpx)
-c:a codec: Set audio codec (e.g., aac, libmp3lame)

Video Settings

-r fps: Set frame rate
-s WxH: Set frame size (e.g., 1920x1080)
-b:v bitrate: Video bitrate (e.g., 2M)
-crf value: Constant Rate Factor (0-51, lower is better quality)
-preset: Encoding speed (ultrafast to veryslow)
-profile:v: Set codec profile (baseline, main, high)
-pix_fmt: Pixel format (yuv420p, rgb24)

Audio Settings

-b:a bitrate: Audio bitrate
-ar rate: Audio sample rate
-ac channels: Number of audio channels
-vol volume: Change audio volume (256=normal)

Filters

-vf "filter": Video filters (scale, crop, rotate)
-af "filter": Audio filters (volume, equalizer)
-filter_complex: Complex filter chains


A button that says "convert" will process the input file using ffmpeg in browser and download the output file. Then add another ui element to preview the video. 



