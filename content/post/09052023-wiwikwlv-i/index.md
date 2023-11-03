---
title: "What I Wish I Knew While Learning Vulkan - I"
description: The standard recipe for requesting resources in Vulkan
slug: wiwikwlv-i
date: 2023-09-05T17:13:12-07:00
categories:
  - Computer Graphics
  - Vulkan
  - Patterns
tags:
  - Computer Graphics
  - Vulkan
  - wiwikwlv
---

### Query for Size and then for Properties separately

Throughout vulkan, there is a pattern of getting the supported properties/resources/... and then choosing one from amongst them.

As an example, look at this code to choose a `VkPhysicalDevice`.

```
void pickPhysicalDevice() {
    uint32_t deviceCount = 0;
    vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);

    if (deviceCount == 0) {
        throw std::runtime_error("failed to find GPUs with Vulkan support!");
    }

    std::vector<VkPhysicalDevice> devices(deviceCount);
    vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());

    // iterate through the list and choose a device
}
```

We first get the `deviceCount`. While getting this value, we are explicit about setting the fourth parameter as `nullptr` telling that we do not need the data and we are only interested in the count.

We then do some error checking around the count and then create a vector of the resource by preallocating the size according to the count. We use this vector as the fourth parameters while calling the same function again. But this time, we also get the handles in that vector.

### Generalizing it a bit

Essentially, we are -

1. query the size or count or number of resources by calling the `Vk` method
2. do some error check
3. pre-allocate a vector of the aforementioned size
4. call the same `Vk` method with the array pointer passed in so that it is populated with handles

Vulkan does this and encourages us to follow this practice because it allows us to run the code which will be robust across various systems. 

 ### Diving into some examples

 I'll keep referencing code in [Vulkan Examples Repo by Sascha Willems](https://github.com/SaschaWillems/Vulkan). This is a very well maintained repository and it is often cited as a reference for learning various vulkan techniques and recipes. In all of the below examples, we follow the pattern of querying the number of resources/handles/properties that are available. We then create an array of that particular size and call the function again to fill in the information.

#### Querying for Queue Family Properties

Look at the [code](https://github.com/SaschaWillems/Vulkan/blob/master/base/VulkanDevice.cpp#L38-L42). We get the queue family properties for different queues and select the queue families that fit our usecase.

#### Querying for Available Extensions

Look at the [code](https://github.com/SaschaWillems/Vulkan/blob/master/base/VulkanDevice.cpp#L45-L58) to see how to get the vulkan extensions that are supported on a specific system. Once we get a list of these extensions, we can check against them for the functionalities we require and provide an experience that works on that system.

#### Querying for the Surface Formats available for Vuklkan Swapchain

Look at the [code](https://github.com/SaschaWillems/Vulkan/blob/master/base/VulkanSwapChain.cpp#L168-L174) to see how we can get the surface formats that are available on our system. Once we get these, we can check if the format we wish to use is there or not. If supported, we continue using it.

#### Querying for the Validation Layers before using a validation layer

Look at this [code](https://github.com/SaschaWillems/Vulkan/blob/master/base/vulkanexamplebase.cpp#L127-L130) to see how we can get the validation layers that are available for us on a system. We can then proceed to use a validation layer if it is available to us on this system.

#### Getting a handle for Swapchain Images

Look at this [code](https://github.com/SaschaWillems/Vulkan/blob/bb81bbd5355a0ec3e309f7655feb44e3fbf8db2c/base/VulkanSwapChain.cpp#L364-L368) where we query for the number of images that be supported by our Swapchain. We allocate an array of that size and then pass this array as an argument to the same function again. In the end, he have an array that has the handles for the swap images.
