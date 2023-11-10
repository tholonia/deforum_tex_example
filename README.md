## Final Video:

[![Example final video](https://img.youtube.com/vi/MvSfY1iaIrI/0.jpg)](https://www.youtube.com/watch?v=MvSfY1iaIrI)

## Requirements

To deploy this process you will need:

- A working knowledge of Parseq.

- A1111 server with the following installed:
  - Protogen_V2.2.ckpt [bb725eaf2e] (https://huggingface.co/darkstorm2150/Protogen_v2.2_Official_Release/blob/main/Protogen_V2.2.ckpt)
  
  - Deforum extension
  
  - ControlNets
    - `control_v11f1e_sd15_tile [a371b31b]`
    - `diff_control_sd15_temporalnet_fp16 [adc6bd97]`
    - `control_v11p_sd15_openpose [cab727d4]`
    - `control_v11p_sd15_canny [d14c016b]`
  


- Command line access to `ffmpeg`.

- Davinci Resolve Studio (not the free version), or another tool that can automatically identify and mask selected objects.

- A video of the subject(s).  In this example, that video is called `tex_org.mp4`and is a video of two people dancing tango. The subject(s) in the video needs to be maskable (i.e., contrasting against background)
	- Example `tex_org.mp4`
	- [![Example tex_mask.mp4](https://img.youtube.com/vi/7iTA_Du584Q/0.jpg)](https://www.youtube.com/watch?v=7iTA_Du584Q)

- The following Loras are needed:

- Jellyfish_forest (https://civitai.com/models/106312/lora-jellyfish-forest-concept-with-dropout-and-noise-version)
- lego_v2.0 (https://civitai.com/models/92444/lelo-lego-lora-for-xl-and-sd15)
- margotrobbie_v2 (https://civitai.com/models/59383/margot-robbie




## Davinci

- Create a folder anywhere called **`tex/`** to hold your working files.   Put your original video of dancers (here called **`tex_org.mp4`**) into this folder.

- Create a subfolder called **`tex/bgimages`**. Put the images to be used as backgrounds (6 images, in this case).  These images should be the same resolution as the video output (1280x720 in this case).

- Open **`tex_org.mp4`** in Davinci Resolve Studio. Mask out the background and render the file to **`tex/tex_mask.mp4`**. 
  - Render DANCERS ONLY to **`tex/tex_mask.mp4`**
    - [![Example tex_mask.mp4](https://img.youtube.com/vi/Z1rG5hP39k4/0.jpg)](https://www.youtube.com/watch?v=Z1rG5hP39k4)
  - Render ALL the channels to **`tex.mp4`**
  - - [![Example tex_mask.mp4](https://img.youtube.com/vi/WhAY2y2vJrs/0.jpg)](https://www.youtube.com/watch?v=WhAY2y2vJrs)
  - To test the timeline and to identify the in/out points (in points of each background image, add the images to the video tracks and adjust accordingly.  Render this video to visually test.  In this timeline, only the out-points are recorded as the in-points are determined by the length of each clip in Parseq.  If the clips were of different length, both in-points and out-points would need to be reflected in the timeline.
  - Example of Davinci Resolve Timeline of dancers and backgrounds.
  - ![timeline.jpg](/home/jw/src/sdc/settings/tex/timeline.jpg)
  - Example of Davinci Output of test layout with masked dancers and backgrounds.
  - [![Example tex_mask.mp4](https://img.youtube.com/vi/hWxd2VV1r4U/0.jpg)](https://www.youtube.com/watch?v=hWxd2VV1r4U)
  - Render BACKGROUND IMAGES ONLY to **`tex-background.mp4`**, which are all the background clips without the dancer clip.
  - [![Example tex_mask.mp4](https://img.youtube.com/vi/FbPpefidp4k/0.jpg)](https://www.youtube.com/watch?v=FbPpefidp4k)
  - ***Note: As the fps of this video will get reduced to 10, we can use the prompt fade values (from parseq) to locate the seconds of where each image should begin and end. These timing were previously determioned in teh test video above.  Set the image to/from seconds for each image that corresponds to each prompt fade from/to value. So, image 1 starts at 0s and ends at 9.9s because the `prompt fade from/to` values are 0 and 99.  Image 2 starts at 7s and ends as 18.4s, and so on.  Apply some transition (I use fade-out) for smooth transitions.***
  
- Extract the audio from the video to a separate file
  - `ffmpeg -i tex_org.mp4 -q:a 0 -map a tex_audio.mp3`

- Davinci does not output at 10 fps, so reduce the fps of  **`tex_mask.mp4`** and **`tex.mp4`** to 10 with `ffmpeg` to **`tex_mask_10.mp4`** and **`tex_10.mp4`**
  - ` ffmpeg -i tex_mask.mp4 -vf "minterpolate='mi_mode=mci:mc_mode=aobmc:vsbmc=1:fps=10'" tex_mask_10.mp4`
  - ` ffmpeg -i tex_mask.mp4 -vf "minterpolate='mi_mode=mci:mc_mode=aobmc:vsbmc=1:fps=10'" tex_mask_10.mp4`


- Extract the first frame of the video to **`tex_frame_0.jpg`**
  -  `ffmpeg -i tex_10.mp4 -vf "select=eq(n\,0)" -q:v 3 tex_frame_0.jpg`



## Parseq

Create the Parseq URL.

the URL used for this example is https://firebasestorage.googleapis.com/v0/b/sd-parseq.appspot.com/o/rendered%2FFU5aOZQ2PXclwylgTbeQiLjX8rM2%2Fdoc-1eff40d2-7610-4fed-a661-5b393fd2fd6c.json?alt=media&token=6e60cb57-9801-4797-8dc5-102f7557839a.  The Parseq JSON file is also saved here as **`parseq.json`**

Open the web app page https://sd-parseq.web.app/ and load the **`parseq.json`** file, which has the following settings:

- FPS = 10
- Final Frame = *\<Final number of inputframes\>*
  - ***Note: At this point just estimate the frames by multiplying the total seconds of the video by 10.  When you make your first Deforum run, you will need to look in `<A1111 root directory>/outputs/img2img-images/tex/inputframes/` folder and look at the exact number of frames that were extracted.  That is the number to use here.  If this number is off by 1 the render will crash when it gets to the end.***

- Prompt 1 = "two jellyfish dancing tango in a (((forest))) \<lora:Jellyfish_forest:1.0\>,((underwater photography))"
- Prompt 2 = "LEGO MiniFig couple of Brad Pitt and Helen Hunt dancing tango in a LEGO village \<lora:lego_v2.0_XL_32:1.0\> ((LEGO blocks)) "
- Prompt 3 = "a (((short skinny clown))) and a tall very (((fat obese clown))) dancing tango in a (((circus ))) "
- Prompt 4 = "An Argentine gaucho dancing tango with Margot Robbie in a (((cow field))), \<lora:margotrobbie_v2:1\>"
- Prompt 5 = "Leonardo Davinci's statue of David and Praxiteles of Athens statue of Aphrodite dancing tango inside (((Buckingham palace)))"
- Prompt 6 = "old Maggie Smith and old Ian McKellen dancing tango in the (((streets of Argentina)))"
  - ***Note: using named personalities and the `margorobbie` lora is an attempt to minimize the faces of peopel radically changing***.

- Positive common = "high quality,  masterpiece, high detail, 8k, uhd,"
- Negative common = "(((people, bodies, men, woman, humans))), EasyNegative, badhandv5, watermark, logo, text, signature, copyright, writing, letters, low quality, artifacts, cropped, bad art, poorly drawn, simple, pixelated, grain, noise, blurry, cartoon, computer game, video game, painting, drawing, sketch, deformed feet"
  - ***NOTE: Negating people, bodies, men, women, etc., is an attempt to minimize the number of people that pop up in the background, as I am trying to keep the background free of other people***,

- Evenly Spaced Prompts
  - Fade Frames = 30
  - Managed fields = None

**Note: After creating the evenly spaced prompt fades. Write down the `From` and `To` frame numbers.  In this example, there is a total of 507 frames, so the prompts have the `From/To` values of :**

- **Prompt 1: 0/99**
- **Prompt 1: 70/184**
- **Prompt 1: 155/260**
- **Prompt 1: 239/353**
- **Prompt 1: 324/438**
- **Prompt 1: 409/507**



## Deforum

- Start A1111/Deforum. Set the model to `Protogen_v2.2`, and the following variables:
- **Run**
  - Sampler = Euler a
  - steps = 15  
    - ***Note: I have gone as low as 11 with acceptable result. The higher the steps, the more 'busy' the image gets which allows for more unpredictable 'things' popping up in the background.***
  - w/h = 1280/720
  - Restore faces = ON
  - Tiling = ON
  - batchname = tex
- **Keyframes**
  - strength = 0:(0.0)
  - 3D = ON
  - Cadence = 1
  - CFG=0.5
  - Seed = fixed
  - Motion = Set all to 0:(0.0)
  - Noise = 0
  - **Coherence**
    - Color Coherence = None
    - 	Optical flow cadence = DIS Medium
  - Anti-blur
    - Amount schedule = 0:(0.0)
- **Init**
  - Image Init
    - Use init = ON
    - Init image URL = ~/projects/tex/tex_frame_0.jpg
  - Video Init
    - Video init path/URL = ~/projects/tex/text_mask_10.mp4
  - Parseq
    - Parseq manifest (JSON or URL) = *\<Paste Parseq URL after clicking on `RE-RENDER` and `UPLOAD OUTPUT`, then `COPY URL` from within the Parseq web app page.\>*
    - Use FPS, max_frames and cadence from the Parseq manifest, if present  = ON
- **ControlNet**
  - **CN Model 1**
    - Enabled = ON
    - Pixel Perfect = ON
    - Model = `control_v11f1e_sd15_tile [a371b31b]`
    - Preprocessor = None
    - Starting Control Step schedule = 0:(0.0)  ***Note: start applying  CN at step 0 of 15 steps***
    - Ending Control Step schedule = 0:(0.5)  ***Note: stop applying CN at step 7 of 15 steps***
    - ControlNet Input Video=`~/projects/tex/tex_10.mp4` ***<= Note: background and dancers together***
    - ControlNet is more important = ON
  - **CN Model 2**
    - Enabled = ON
    - Pixel Perfect = ON
    - Model = `diff_control_sd15_temporalnet_fp16 [adc6bd97]`
    - Preprocessor = None
    - Starting Control Step schedule = 0:(0.5)  ***Note: start applying  CN at step 8 of 15***
    - Ending Control Step schedule = 0:(1.0)  ***Note: stop applying  CN at step 15 of 15***
    - ControlNet Input Video=`~/projects/tex/tex_10.mp4 ***<=Note: background and dancers together***
    - ControlNet is more important = ON
- - **CN Model 3**
    - Enabled = ON
    - Pixel Perfect = ON
    - Model = `control_v11p_sd15_openpose [cab727d4]`
    - Preprocessor = dw_openpose_full
    - ControlNet Input Video= `~/projects/tex/tex_mask_10.mp4` ***<= Note: masked dancers only***
    - ControlNet is more important = ON
- - **CN Model 4**
    - Enabled = ON
    - Pixel Perfect = ON
    - Model = `control_v11p_sd15_canny [d14c016b]`
    - Preprocessor = canny
    - ControlNet Input Video=`~/projects/tex/tex_10.mp4` ***<=Note: background and dancers together***
      - ***Note: Canny is used here to minimize the changes in the background image***
    - ControlNet is more important = ON
    - Canny Low Threshold = 82
    - Canny High Threshold = 216


- **Hybrid Video**
  - Before Motion = ON
  - Generate Input Frames = ON
  - Optical Flow = DIS Medium
- **Output**
  - FPS = 10
  - Add soundtrack = File
  - Soundtrack path = `~/projects/tex/tex_audio.mp3`
  - Frame Interpolation = Film
    - Interp X = 3

- Save the settings file as `~projects/tex/tex_settings.txt`
- 

On my machine this render takes about 4 hours and renders at about `2.11 it/s`.

This produces two files in `outputs/img2img-images/tex/`

- \<timestamp>_FILM_x3.mp4 (30 fps interpolated)
- \<timestamp>.mp4 (10 fps)
- \<timestamp>_settings.txt

SAVE the settings file to ~/projects/text` as a backup because sometimes there is a problem reloading `teh tex_settings.txt`, but the generated settings file loads OK.







