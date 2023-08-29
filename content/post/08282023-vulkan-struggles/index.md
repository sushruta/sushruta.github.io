---
title: "Learnings from Several Hours of Splitting Hairs over Vulkan"
description: A very highlevel view of how a vulkan pipeline looks like
slug: vulkan-at-very-highlevel
date: 2023-08-28T18:38:12-07:00
image: output-shot.jpg
categories:
  - Computer Graphics
  - Vulkan
tags:
  - Computer Graphics
  - Vulkan
---

#### Foundational Part - Getting an Instance and a PhysicalDevice

You talk to Vulkan API through a `VkInstance`. You use this to get `VkPhysicalDevice` from among your various Vulkan compatible devices. One can choose the appropriate `VkPhysicalDevice` based on various properties and constraints like features available, size of memory, etc.

#### Stepping from Physical to Logical

We now step into the logical world where we create a `VkDevice` that allows you do stuff on the physical device without getting into the weeds of how the physical device works. Think of it as an abstraction over `VkPhysicalDevice`.

Now, when we use this `VkDevice` for rendering, drawing, GPGPU, etc., we submit these steps to a `VkQueue`. To create a `VkQueue`, we select it from a range of `Queue Families`. Again, different queue families provide different features and we should choose the queue family that matches our needs. For example, if we want to write a simple OpenGL like program that takes vertices in a VBO and displays the result onto the screen, we will ask for a queue(s) that give us support to draw things, transferring vertex data to GPU, etc.

#### Don't Forget to Create the Window!

Use GLFW3 for rendering to a window. Most tutorials on the internet use that for Vulkan and it seems straightforward.

We will use `VkSurfaceKHR` to talk to GLFW and display our rendering.

Hand in hand with `VkSurfaceKHR` goes the `VkSwapchainKHR`. Here's how I understand it.

We ask Vulkan to do some calculations and run the pipeline and then draw the result. When Vulkan is "done with drawing the image", it puts the result in `VkSwapchainKHR`. At any point, this swap chain might hold multiple images that need to shown somewhere. Think of it as a queue of images. You mention several properties here like -

1. How images are used from the swapchain - do we send results straightaway? do we use a FIFO style queue? Do we use tricks like `mailbox` and `relaxed` to account for applications being slower or faster than refresh rates.
2. Specifying the size of the image a.k.a swap extent
3. telling the application about the color space. This is beyond my understanding and I'll assume you always use something like `VK_FORMAT_B8G8R8A8_SRGB`.

Now, these images need to be actually presented on the screen and this is where `VkSurfaceKHR` comes in. Think of it as working with GLFW to show the image. You mention some
