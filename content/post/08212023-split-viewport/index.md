---
title: "Split Viewport in OpenGL"
description: Helpful way to visualize camera and lights
slug: split-viewpport-opengl
date: 2023-08-21T18:22:37-07:00
image: output-shot.jpg
categories:
  - Computer Graphics
  - OpenGL
tags:
  - Computer Graphics
  - OpenGL
---

### TL;DR Summary

I have demonstrated a simple way to visualize the scene from two views - one from the actual camera and one from a further distance
away. This way of visualizing the scene in two ways provides us with an intuitive feel of the camera and lights and the geometry in
the scene and help us reason about it better.

### Background

Lights are usually defined in the view space (also known as camera space). There is a very nice reason to do this. It helps
us reason about the scene in a much easier way. Imagine having lights defined in the world coordinate system. One would have
to mentally apply the view transformation so that one can mentally picture the position of the light w.r.t camera.

To make it convenient and not get into the circus of mentally imagining the lights and their transformation, one usually defines
lights in view coordinate system so that we do not have to transform it in our head.

### How is this related to Split Viewport?

The above fact is something I learnt the hard way. I assumed the light to be in world space and would do the light calculations
from there. Unfortunately my scene would be very dark because the light in the view space would be right in front of me and
thereby behind the geometry I was viewing.

In the process of debugging, I had the desire to have two displays rendered. One from the point of the view of the camera and other
from a distant further point towards the center of the geometry. In this scene, I should be able to view my perspective
frustrum, my geometry and my lights. This would let me understand the scene in a very intuitive manner.

### Let us dive into the code

I wrote some simple OpenGL code to create this split viewport kind of a setup. Let us look at it.

#### Rendering the scene twice

We will have to render the scene twice. From two different points of view. For this, we will call the draw methods
on the geometry twice but with different `model` and `view` matrices. The resulting `mvp` changes but the rendering
logic remains same.

```
class RenderParams
{
  public:
    unsigned int startW, startH, width, height;
    glm::mat4 model, view, projection;

    unsigned int* VAO_address;
    std::vector<glm::vec3> positions;
};

void render(RenderParams& rp, Shader& shader)
{
  glViewport(rp.startW, rp.startH, rp.width, rp.height);

  shader.setMat4("projection", rp.projection);
  shader.setMat4("view", rp.view);

  glBindVertexArray(*(rp.VAO_address));
  // for (auto position : rp.positions)
  for (unsigned int i=0; i<rp.positions.size(); ++i)
  {
    glm::mat4 model = glm::mat4(1.0f);
    model = glm::translate(model, rp.positions[i]); // rp contains the translation of the object
    float angle = 15.0f * i;
    model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));
    shader.setMat4("model", model);

    glDrawArrays(GL_TRIANGLES, 0, 36);
  }
}
```

I will created two instances of `RenderParams`, two instances describing a different mvp matrix. We call the render
function twice with different `RenderParam` instance.

Also, note that I called glViewport inside the draw loop. Usually, viewport is set once and doesn't change every
frame. That is why it is set up outside the draw loop. In this case, we need to draw to left part of the screen
first and then on the right part of the screen. Therefore, before drawing, we have to explicitly assign the viewport.

Hence, it is being done everytime in the draw loop. In the future, I'll do some experiments to understand if this
is bad from performance standpoint.

### Code is available on Github

Please look at the [cpp file here](https://github.com/sushruta/starter-code-opengl/blob/main/applications/split-viewport.cpp) to see the code in action. You can also download the repo and run it.

### Come back soon for the next part!

In the next post, I will show the difference between lighting when the light's position is defined in world coordinate space and
when it is defined in camera coordinate space.
