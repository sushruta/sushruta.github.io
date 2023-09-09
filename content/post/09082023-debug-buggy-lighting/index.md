---
title: "Debugging Buggy Lighting Code"
description: How to debug broken lighting code in vulkan
slug: buggy-vulkan-lighting-code
date: 2023-09-08T18:11:54-07:00
categories:
  - Computer Graphics
  - Vulkan
tags:
  - Computer Graphics
  - Vulkan
---

## Background

I wrote some vulkan code to load a model and render it using Phong Shading. It involved some vulkan code and shaders in glsl. However, lighting wouldn't work. I would get a blank screen when running the code.

Here is code for the two shaders

```
/////////// VERTEX SHADER ///////////////
#version 450

layout(binding = 0) uniform UniformBufferObject {
  mat4 model;
  mat4 view;
  mat4 proj;
} ubo;

layout(location = 0) in vec3 inPosition;
layout(location = 1) in vec3 inNormal;
layout(location = 2) in vec2 inTexCoord;

layout(location = 0) out vec3 fragNormal;
layout(location = 1) out vec3 fragPosition;
layout(location = 2) out vec2 fragTexCoord;

void main() {
  mat4 nmat = transpose(inverse(ubo.view * ubo.model));
  fragNormal = mat3(nmat) * inNormal;

  fragTexCoord = inTexCoord;
  fragPosition = vec3(ubo.view * ubo.model * vec4(inPosition, 1.0));
  
  gl_Position = ubo.proj * ubo.view * ubo.model * vec4(inPosition, 1.0);
}

/////////// FRAGMENT SHADER ///////////////
#version 450

layout(location = 0) in vec3 fragNormal;
layout(location = 1) in vec3 fragPosition;

layout(location = 0) out vec4 outColor;

void main() {
  // point light
  vec4 lightPos = vec4(0.0, 0.0, 0.0, 1.0);

  vec4 ambientColor = vec4(0.2, 0.2, 0.2, 1.0);
  vec4 diffuseColor = vec4(0.6, 0.2, 0.3, 1.0);
  vec4 specularColor = vec4(1.0, 1.0, 1.0, 1.0);

  outColor = vec4(0.0, 0.0, 0.0, 1.0);

  outColor += ambientColor;

  vec3 camera_space_normal = normalize(fragNormal);
  vec3 camera_space_lightDir = normalize(lightPos.xyz - fragPosition);

  float diffuseIntensity = max(dot(camera_space_normal, camera_space_lightDir), 0.0);
  outColor += diffuseColor * diffuseIntensity;

  outColor = clamp(outColor, 0, 1);
}
```

![Only Ambient Lighting shows up](01-only-ambient-shows-up.gif)

### Remove ambient lighting and check diffuse light

The result was that only ambient lighting was showing up. This is same as a constant color being added everywhere. Clearly the diffuse component wasnt being added. It may mean that the diffuse component is infact always zero. That can mean that diffuse component is infact always negative and it is getting clamped at `0.0` because of the `max` component. Let us investigate that by assigning the diffuse component at the component of light. Also, I will turn off ambient lighting. The resulting fragment shader would like the following

```
...
void main() {
  ...

  outColor = vec4(0.0, 0.0, 0.0, 1.0);

  ...

  float diffuseIntensity = dot(camera_space_normal, camera_space_lightDir);
  
  if (diffuseIntensity < 0) {
    outColor = vec4(-diffuseIntensity, -diffuseIntensity, -diffuseIntensity, 1.0);
  }
}
```

This resulted in nothing being rendered :(

### Check if the dot product is always zero

This resulted in nothing being visible which means `diffuseIntensity` is not negative either. It could mean that `diffuseIntensity` is always zero?!?!?? It could be that the dot product is always zero. This is possible if all normals are always perpendicular to the light dir. This is very very unlikely. The more likely thing happening here could be that either one or both of `camera_space_normal` or `camera_space_lightdir` are always zero.

#### Set outColor as the normal at that point

Let us check `camera_space_normal` first. We will set `outColor` to the absolute values of the normal.

```
// set outColor to the absolute values of normal to check if they are ever non-zero
outColor = vec4(abs(camera_space_normal), 1.0);
```

This resulted in the following resulted in a blank screen again meaning the normal is zero everywhere.

#### Set outColor as the lightDir at that point

Let us also check `camera_space_lightDir` by setting its absolute value as outColor. Both could be zero.

```
outColor = vec4(abs(camera_space_lightDir), 1.0);
```

![setting outColor as lightDir](02-camera-space-lightDir-color.gif)

Clearly, lightDir is not zero everywhere meaning lightDir could be correct.
