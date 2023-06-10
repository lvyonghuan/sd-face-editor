# Face Editor
Face Editor for Stable Diffusion.
It can be used to repair broken faces in images generated by Stable Diffusion.

![example](./images/image-01.jpg)

This is a [extension](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Extensions) of [AUTOMATIC1111's Stable Diffusion Web UI](https://github.com/AUTOMATIC1111/stable-diffusion-webui).

This software improves facial images in these features:
- txt2img
- img2img
- batch processing (batch count / batch size)
- img2img Batch

## Setup
1. Open the "Extensions" tab then the "Install from URL" tab.
2. Enter "https://github.com/ototadana/sd-face-editor.git" in the "URL of the extension's git repository" field.
   ![Install from URL](./images/setup-01.png)
3. Click the "Install" button and wait for the "Installed into /home/ototadana/stable-diffusion-webui/extensions/sd-face-editor. Use Installed tab to restart." message to appear.
4. Go to "Installed" tab and click "Apply and restart UI".


## Usage
1. Click "Face Editor" and check **"Enabled"**.
   ![Check Enabled](./images/usage-01.png)
2. Then enter the prompts as usual and click the "Generate" button to modify the faces in the generated images.
   ![Result](./images/usage-02.png)
3. If you are not satisfied with the results, adjust the [parameters](#parameters) and rerun. see [Tips](#tips).


## Tips
### Contour discomfort
If you feel uncomfortable with the facial contours, try increasing the **"Mask size"** value. This discomfort often occurs when the face is not facing straight ahead.

![Mask size](./images/tips-02.jpg)

If the forelock interferes with rendering the face properly, generally, selecting "Hair" from **"Affected areas"** results in a more natural image.

![Affected ares - UI](./images/tips-08.png)

This setting modifies the mask area as illustrated below: 

![Affected ares - Mask images](./images/tips-07.jpg)


---
### When multiple faces are close together
When multiple faces are close together, one face may collapse under the influence of the other.
In such cases, enable **"Use minimal area (for close faces)"**.

![Use minimal area for close faces](./images/tips-04.png)

---
### Change facial expression
Use **"Prompt for face"** option if you want to change the facial expression.

![Prompt for face](./images/tips-03.jpg)

#### Individual instructions for multiple faces
![Individual instructions for multiple faces](./images/tips-05.jpg)

Faces can be individually directed with prompts separated by `||` (two vertical lines).

![Individual instructions for multiple faces - screen shot](./images/tips-06.png)

- Each prompt is applied to the faces on the image in order from left to right.
- The number of prompts does not have to match the number of faces to work.
- If you write the string `@@`, the normal prompts (written at the top of the screen) will be expanded at that position.
- If you are using the [Wildcards Extension](https://github.com/AUTOMATIC1111/stable-diffusion-webui-wildcards), you can use the `__name__` syntax and the text file in the directory of the wildcards extension as well as the normal prompts.

---
### Fixing images that already exist
If you wish to modify the face of an already existing image instead of creating a new one, follow these steps:

1. Open the image to be edited in the img2img tab
   It is recommended that you use the same settings (prompt, sampling steps and method, seed, etc.) as for the original image. 
   So, it is a good idea to start with the **PNG Info** tab.
   1. Click **PNG Info** tab.
   2. Upload the image to be edited.
   3. Click **Send to img2img** button.
2. Set the value of **"Denoising strength"** of img2img to `0`. This setting is good for preventing changes to areas other than the faces and for reducing processing time.
3. Click "Face Editor" and check "Enabled".
4. Then, set the desired parameters and click the Generate button.

---
## How it works
This script performs the following steps:

### Step 0
First, image(s) are generated as usual according to prompts and other settings. This script acts as a post-processor for those images.

### Step 1. Face Detection
Detects faces on the image.
![step-1](./images/step-1.jpg)

### Step 2. Crop and Resize the Faces
Crop the detected face image and resize it to 512x512.
![step-2](./images/step-2.jpg)

### Step 3. Recreate the Faces
Run **img2img** with the image to create a new face image.
![step-3](./images/step-3.jpg)

### Step 4. Paste the Faces
Resize the new face image and paste it at the original image location.
![step-4](./images/step-4.jpg)

### Step 5. Blend the entire image
To remove the borders generated when pasting the image, mask all but the face and run **inpaint**.
![step-5](./images/step-5.jpg)

### Completed
![step-6](./images/step-6.jpg)

## Parameters
### Basic Options
##### Use minimal area (for close faces)
When pasting the generated image to its original location, the rectangle of the detected face area is used. If this option is not enabled, the generated image itself is pasted. In other words, enabling this option applies a smaller face image, while disabling it applies a larger face image.

##### Save original image
Specify whether to save the image before modification.

##### Show intermediate steps
Specifies whether to display images of detected faces and masks.

If the generated image is unnatural, enabling it may reveal the cause.

##### Prompt for face
Prompt for generating a new face.
If this parameter is not specified, the prompt entered at the top of the screen is used.

For more information, please see: [here](#change-facial-expression).




##### Mask size (0-64)
Size of the mask area when inpainting to blend the new face with the whole image.

**size: 0**
![mask size 0](./images/mask-00.jpg)

**size: 10**
![mask size 10](./images/mask-10.jpg)

**size: 20**
![mask size 20](./images/mask-20.jpg)

##### Mask blur (0-64)
Size of the blur area when inpainting to blend the new face with the whole image.


---
### Advanced Options
#### Step 1. Face Detection
##### Maximum number of faces to detect (1-20)
Use this parameter when you want to reduce the number of faces to be detected.
If more faces are found than the number set here, the smaller faces will be ignored.

##### Face detection confidence (0.7-1.0)
Confidence threshold for face detection. Set a lower value if you want to detect more faces.

#### Step 2. Crop and Resize the Faces
##### Face margin (1.0-2.0)
Specify the size of the margin for face cropping by magnification.

If other parameters are exactly the same but this value is different, the atmosphere of the new face created will be different.

![face margin](./images/face-margin.jpg)

##### Size of the face when recreating 
Specifies one side of the image size when creating a face image. Normally, there should be no need to change this from the default value (512), but you may see interesting changes if you do.

##### Ignore faces larger than specified size
Ignore if the size of the detected face is larger than the size specified in "Size of the face when recreating".

For more information, please see: [here](https://github.com/ototadana/sd-face-editor/issues/65).

#### Step 3. Recreate the Faces
##### Denoising strength for face images (0.1-0.8)
Denoising strength for generating a new face.
If the value is too small, facial collapse cannot be corrected, but if it is too large, it is difficult to blend with the entire image.

**strength: 0.4**
![strength 0.4](./images/deno-4.jpg)

**strength: 0.6**
![strength 0.6](./images/deno-6.jpg)

**strength: 0.8**
![strength 0.8](./images/deno-8.jpg)

#### Step 4. Paste the Faces
##### Apply inside mask only
Paste an image cut out in the shape of a face instead of a square image.

For more information, please see: [here](https://github.com/ototadana/sd-face-editor/issues/33).

#### Step 5. Blend the entire image
##### Denoising strength for the entire image (0.0-1.0)
Denoising strength when inpainting to blend the new face with the whole image.
If the border lines are too prominent, increase this value.


## API
If you want to use this script as an extension (alwayson_scripts) in the [API](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/API), specify **"face editor ex"** as the script name as follows:

```
   "alwayson_scripts": {
      "face editor ex": {
         "args": [{"prompt_for_face": "smile"}]
      },
```

By specifying an **object** as the first argument of args as above, parameters can be specified by keywords. We recommend this approach as it can minimize the impact of modifications to the software. If you use a script instead of an extension, you can also specify parameters in the same way as follows:

```
   "script_name": "face editor",
   "script_args": [{"prompt_for_face": "smile"}],
```

- See [source code](https://github.com/ototadana/sd-face-editor/blob/main/scripts/entities/option.py) for available keywords.
