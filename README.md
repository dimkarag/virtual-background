# Virtual Background

Creation of a component which requires an image for the background and a video or image to be displayed on top. The background of the video/image is been removed using 2 methods. The first is person recognition and segmentation using tensorflow bodypix and the second is chroma color removal in which you can provide the color to be removed from the background.

## Properties

* backgroundImageSrc: The src of the image to be set as virtual background of the rest content.
* contentSrc: The src of video ('mp4', 'wav') or image ('jpeg', 'jpg', 'png') to be loaded and displayed on top of the virtual background.
* removalType: The removal type defines the method of removing the backround of the content. The values are 'personSegmentation' or 'chromaColor'. Person segmentation uses tensorflow bodypix to recognize person on the content and remove the rest of the content. Chroma color is used for content images/videos with a specific color (ex green screen videos) on the background. You can define on another property which is this color to be removed.
* bodyPixConfig: If the removal Type is set to 'personSegmetation' a bodyPixConfig is needed. For more info for the load bodypix config please visit https://www.npmjs.com/package/@tensorflow-models/body-pix
The default values are these below:
```    
    {
      internalResolution: 0.2,
      segmentationThreshold: 0.5,
      scoreThreshold: 0.5
    }
```
* personSegmentationConfig: Config for the bodypix person segmentation. For more info for the load bodypix config please visit https://www.npmjs.com/package/@tensorflow-models/body-pix
The default values are these below:

```
      {
        architecture: 'MobileNetV1',
        outputStride: 16,
        multiplier: 1,
        quantBytes: 4
      }
 ```
* fps: If the content is a video you can define the requested fps. Note that the preferable fps is been reached only if the device is able to reach it. Video size, rezolution, frame rate but also the resolution requested on config for bodyfix may affect the fps.

* chromaColorConfig: If the removalType is set to 'chromaColor' the chromaColorConfig defines the type of RGB values to be removed from the image or video frame.
The property keys that can be set are like Rgt, Rlt, Ggt, Glt, Bgt, Blt which mean Red grater than, Red less than, Green greater than etc. The default values can remove a variety of green color. The default values are showed below: 

```
    {
      Rgt: 10,
      Ggt: 80,
      Blt: 30
    }
```

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

### Lints and fixes files
```
yarn lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
