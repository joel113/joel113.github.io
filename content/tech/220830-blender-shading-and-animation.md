+++
author = "Johannes Ehm"
title = "Blender Shading and Animation"
date = "2022-08-23"
description = "A summary of my experience of working with blender shading and animation"
tags = [
    "flink",
		"streaming"
]
draft = true
+++

I did use the open source software blender to visualize furniture. This week, I did a project which did lead to a dive into Blender shading and animation after the famous [BlenderGuru](https://www.blenderguru.com/) did post a [Youtube video tutorial](https://www.youtube.com/watch?v=0YZzHn0iz8U&t=31s) to model, shade and render a foto realistic earth. While adding the sphere object was easy and well knonw, the possibility of shading and animation was new to me. Blender Guru has also [a nice Youtube video](https://www.youtube.com/watch?v=Xf07p_SjXKI) showing his render results only.

## Shading

Blender [Shading](https://www.blenderguru.com/) is were you add a texture to your objects. Blender offers a variety of possibilities to source, transform and compute a texture. It offers an editor where you can maintain a texture workflow including the needed workflow items. In the case of the earth project, the textures are sources from NASA files available on the internet. The NASA texture files are used as texture sources and added as `Image Texture` to the texture worfklow. The project uses the textures

- earth color,
- earth landocean,
- earth topography and
- earth nightlights.

Furthermore, the workflow items

- hue saturation value,
- map range,
- displacement,
- blackbody,
- multiply,
- normal,
- texture and
- principled bsdf

are used to avoid the shinyness of landmasses, the displacements of mountains and the correct application of nightlights with respect to the sun.

### Principled BSDF

### Hue Saturation Value

### Map Range

### Displacement

### Blackbody

### Multiply

### Map Range

### Normal

### Texture Coordinate


## Animation

## FFMPEG

https://askubuntu.com/questions/610903/how-can-i-create-a-video-file-from-a-set-of-jpg-images?noredirect=1&lq=1

```
ffmpeg -framerate 12 -i %04d.png -c:v libx264 -profile:v high -crf 20 -pix_fmt yuv420p output-12fps.mp4
```

## Results

https://vimeo.com/744165987

## References


