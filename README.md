# face-geometry

This repo is the beginnings of some work looking at using Google's Mediapipe facemesh within A-Frame.

https://google.github.io/mediapipe/solutions/face_mesh.html

https://aframe.io/

This work did not get very far, but I'm sharing it in case it should prove to be a useful start for someone else.  And there's a chance I may come back to this & extend it.



## What's in this repo?

So far, this is very limited.  Depending on my time & interest, it may grow over time.

3 things that may be of interest:

- [original](https://diarmidmackenzie.github.io/face-geometry/mediapipe/original.html) - this is a direct copy/past of Google's javascript example from their docs here: https://google.github.io/mediapipe/solutions/face_mesh.html#javascript-solution-api

- [testing](https://diarmidmackenzie.github.io/face-geometry/testing/face-mesh-testing.html) - a simple side-by-side display of the original camera, and the face-mesh - with the ability to toggle on/off display of parts of the mesh.

  - Intended for looking at how good individual parts of the mesh (e.g. lips) are at tracking that feature.
  - Some of these are known to be not great - see "How accurate is Google Mediapipe Facemesh" below.

- [basic-example](https://diarmidmackenzie.github.io/face-geometry/examples/basic-example.html) - an example that shows facemesh rolled up into an A-Frame component

  - This displays the index of each point in the face mesh
  - It also shows the full range of the points on each of the x, y & z axes.
  - However there are issues with the mapping into 3D space - see below.  These become particularly evident when the face is to the left or right of the screen.

- [block-head](https://diarmidmackenzie.github.io/face-geometry/examples/block-head.html) - a minecraft-style cubs-shaped head, animated to match your expressions.

  - Matches head tilt, eye height, eyebrow position & mouth height & width.

  

## How accurate is Google Mediapipe Facemesh?

From what I can tell, it's the best Open Source face mapping technology available (as of time of writing, Sep-2021).

But that doesn't mean it is amazing.

Lip tracking in particular has obvious flaws when e.g.

- Lips are pouted or puckered
- Lips are asymmetric
- One or both lips is concealed within the mouth

If you look into the Mediapipe github issues, it's clear that Google know it's not great, and specifically they think it's not suitable for some applications, such as lipstick VTOs.
https://github.com/google/mediapipe/issues/1003

The paper linked from the issue says:

- existing facemesh is observably not good enough for lipstick VTOs (and various other applications) because of lip inaccuracies.
- this paper shows how you can do much better, still within reasonable performance parameters.

The issue asks whether this enhanced model will be opened sourced.

- Google say...
  - Aug 2020: yes, maybe by end-2020.
  - Dec 2020: we do not have plans to open source the lip model.

The paper says the results achievable with their new model were so good that:

- 46% of AR samples were classed as real
- while 38% of real samples were classed as AR
- (however, this was with static images - I think it's unlikely this level of realism would be achieved with a webcam feed).




## Issues with Mapping into 3D Space

The co-ordinates returned by Google's facemesh are not regular 3D co-ordinates that you'd be used to if you've worked with A-Frame.
The issue is explained in detail here:

https://google.github.io/mediapipe/solutions/face_mesh.html#face-geometry-module

In summary, the points are provided in a ["weak perspective projection"](https://en.wikipedia.org/wiki/3D_projection#Weak_perspective_projection)

If you've ever drawn a one-point perspective drawing, with a single vanishing point in the center of the screen, this basically uses the same concepts...

- each point has x/y co-ordinates which indicate a line radiating from the vanishing point 
- the z point indicates where on this line the point lies, with zero being a plane a short way in front of the camera (I believe zero is the average position of all the points, but don't quote me on that).

Interestingly, the model doesn't actually have any idea how far away the face is (i.e. what the true z co-ordinate is).  If you go to the [basic-example.html](https://diarmidmackenzie.github.io/face-geometry/examples/basic-example.html), and move your face forwards/backwards you'll see that the range of z-coordinates doesn't change as you'd expect if the co-ordinates were regular x/y/z co-ordinates in A-Frame.

How does my basic example convert these into A-Frame co-ordinates?

Well, it ignores all these subtleties, and just uses them as x/y/z co-ordinate offsets, relative to a fixed point.  That's encoded in the HTML as

```
position = "-0.5 2.1 -1.15"
```

(noting camera height is y=1.6)

Why these values?  Well, they seemed to make things fit about right...!

This looks *ok* when the face is in the center of the screen, but it starts to get significantly misaligned when the face moves to the edge of the screen.

A *proper* mapping to 3D space requires:

- Some estimate of the actual distance of the face from the camera
  - Could be done based on x-range and y-range of the points on the face (i.e. assume the face is of some expected size)
  - Could (somewhat more ambitiously, and possibly more accurately) be done by measuring the iris dimensions - see: https://google.github.io/mediapipe/solutions/iris.html#depth-from-iris
- Some "clever maths" to map one co-ordinate system to the next
  - Probably not that hard, but I've not found any references for this, so probably need to dust off the trigonometry and work it out from scratch...

I haven't tried to tackle either of these yet.

- The Google Facemesh docs point to a [Face Geometry module](https://github.com/google/mediapipe/tree/master/mediapipe/modules/face_geometry) that apparently does the necessary maths.  That's potentially pretty helpful, except that it's written in C++, so would need porting to JS (or compilation into WASM , maybe?).

Until this is resolved, this is pretty much unusable for any AR applications where you want to overlay A-Frame objects over the camera feed.... but it might be usable if you wanted to render a 3D version of someone's face, matching their expressions etc., without worrying about alignment with the camera feed.



## What are the indices of points in the face mesh?

Take a look here...

https://raw.githubusercontent.com/google/mediapipe/master/mediapipe/modules/face_geometry/data/canonical_face_model_uv_visualization.png



## API documentation

There's one A-Frame component, track-face.

It takes no parameters.

It renders the face in the form of some 400-odd numbers, each one representing the id of a point on the face mesh.

To use the component, attach it to an entity with a position that shows where you want the face to be rendered.

for example (from basic-example.html):

```html
<a-entity position = "-0.5 2.1 -1.15" track-face>
</a-entity>
```



## Installation of track-face component

I'm not expecting anyone to use the track-face component as it is.

But if you did want to, including the following in your HTML header should make it available to you...

<script src="https://cdn.jsdelivr.net/gh/diarmidmackenzie/face-geometry@latest/src/face-geometry.min.js"></script>



## Future work

I have no firm plans to take this further right now, but I may come back to it at some point.

Obvious next steps would be:

- solve the 3D geometry issues mentioned above, to get mesh points in proper 3D co-ordinates
- extend the track-face component to be more flexible, allow affixing 3D objects to parts of the face, retexturing parts of the face etc.
- or maybe... look at some of the other cool stuff in MediaPipe like pose detection, hand tracking etc. and do something with those...

