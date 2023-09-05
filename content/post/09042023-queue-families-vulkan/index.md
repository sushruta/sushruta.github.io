---
title: "How do Queues and Queue Families operate in Vulkan"
description: Look at different queues and how they work
slug: vulkan-queue-families
date: 2023-09-04T08:13:12-07:00
categories:
  - Computer Graphics
  - Vulkan
tags:
  - Computer Graphics
  - Vulkan
---

One uses Queues for doing things on the GPU. This generally means submitting commands on to the queue which are read and executed at a later point in time. These queues can be viewed as freeways where data, commands, etc. are being passed around.

[Vulkan API specs](https://registry.khronos.org/vulkan/specs/1.3-extensions/man/html/VkQueueFlagBits.html) define the following queues -

```
VK_QUEUE_GRAPHICS_BIT
VK_QUEUE_COMPUTE_BIT
VK_QUEUE_TRANSFER_BIT
VK_QUEUE_SPARSE_BINDING_BIT
VK_QUEUE_PROTECTED_BIT
```

They are each uint32_t values and let us get the value and the name in a map for later use.

```
std::map<VkQueueFlagBits, std::string> flagMap{
  {VK_QUEUE_GRAPHICS_BIT, "VK_QUEUE_GRAPHICS_BIT"},
  {VK_QUEUE_COMPUTE_BIT, "VK_QUEUE_COMPUTE_BIT"},
  {VK_QUEUE_TRANSFER_BIT, "VK_QUEUE_TRANSFER_BIT"},
  {VK_QUEUE_SPARSE_BINDING_BIT, "VK_QUEUE_SPARSE_BINDING_BIT"},
  {VK_QUEUE_PROTECTED_BIT, "VK_QUEUE_PROTECTED_BIT"}
};

std::string getSupportedFlagsStr(VkQueueFlags flags) {
  std::string result{""};
  for (const auto& [flagBit, flagName] : flagMap) {
    if (flag & flagBit) {
      result = result.empty() ? flagName : result + " | " + flagName;
    }
  }
  return result;
}
```

Based on the kind of GPU you have, it can support different kinds of queues. For example, I can write code like this in my vulkan application to query for the various queue families my GPU supports -


```
// assign a variable to track queueFamilyCount and create a large vector
uint32_t queueFamilyCount{0};
std::vector<VkQueueFamilyProperties> queueFamilies(100);
vkGetPhysicalDeviceQueueFamilyProperties(physicalDevice, &queueFamilyCount, queueFamilies.data());

// resize the vector to keep the relevant queuefamilies
queueFamilies.resize(queueFamilyCount);

// print the queue families
std::cout << "<<<QUEUE FAMILY INFORMATION>>>" << std::endl;               
for (const auto& qf : queueFamilies) {                                    
  std::cout << "Queue Count: " << std::to_string(qf.queueCount) << std::endl;
  std::cout << "Queue Flags: " << getSupportedFlagsStr(qf.queueFlags) << std::endl;
}                                                                         
std::cout << "<<<QUEUE FAMILY INFORMATION COMPLETE>>>" << std::endl; 
```

On running the code within a simple vulkan application, I get the following output -

```
<<<QUEUE FAMILY INFORMATION>>>
Queue Count: 16
Queue Flags: VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT | VK_QUEUE_TRANSFER_BIT | VK_QUEUE_SPARSE_BINDING_BIT
Queue Count: 2
Queue Flags: VK_QUEUE_TRANSFER_BIT | VK_QUEUE_SPARSE_BINDING_BIT
Queue Count: 8
Queue Flags: VK_QUEUE_COMPUTE_BIT | VK_QUEUE_TRANSFER_BIT | VK_QUEUE_SPARSE_BINDING_BIT
Queue Count: 1
Queue Flags: VK_QUEUE_TRANSFER_BIT | VK_QUEUE_SPARSE_BINDING_BIT
Queue Count: 1
Queue Flags: VK_QUEUE_TRANSFER_BIT | VK_QUEUE_SPARSE_BINDING_BIT
<<<QUEUE FAMILY COMPLETE>>>
```
We notice the following -

There are `28` queues which expose varying levels of flexibility. For example, there are `8` queues which support only compute and transfer and bind abilities.

### Ordering Semantics for Queues

Elements within a queue are guranteed to be ordered amongst themselves. An element A which is in the queue before an element B is guranteed to execute before B is executed.

However, elements across queues are not ordered in any way. If A exists in one queue and B in another queue, there is no ordering among those two elements. The only way to enforce an ordering is by using `VkSemaphore`.

### Queues and Concurrency

As a developer, I'd have to understand what capabilities I need. Let's say I am going to write a raytracing application. I would need `GRAPHICS` bit for displaying the image through Vulkan. I'll need `TRANSFER` bit for transferring data to GPU. I'll need `COMPUTE` bit for doing the actual raytracing logic in GPU.

As I start the application, I'll ask for queues that support these bits and Vulkan will give you a handle to these queues. It is NOT guranteed that we will get one queue. However, it is definitely possible to ask for one queue. Usually using one queue means everything will happen in order and there is no need to synchronize anything. The downside is that we are throwing away free parallelism that we could probably use by using multiple queues.

Anytime in Vulkan that you do any sort of GPU based rendering, you do that by taking commands in a command buffer and submitting them to a queue. Now this might be a graphics queue for drawing (VK_QUEUE_GRAPHICS_BIT), queue for transferring (VK_QUEUE_TRANSFER_BIT) or for compute shaders (VK_QUEUE_COMPUTE_BIT).
