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

### Background

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

The result was that only ambient lighting was showing up. This is same as a constant color being added everywhere. Clearly the diffuse component wasnt being added.
