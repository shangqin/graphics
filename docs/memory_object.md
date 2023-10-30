# Vulkan和OpenGL ES通过memory object共享数据

## OpenGL ES扩展
### GL_EXT_memory_object
- https://registry.khronos.org/OpenGL/extensions/EXT/EXT_external_objects.txt

**Memory object**

```cpp
void CreateMemoryObjectsEXT(sizei n, uint *memoryObjects);

void DeleteMemoryObjectsEXT(sizei n, const uint *memoryObjects);

boolean IsMemoryObjectEXT(uint memoryObject);

void TexStorageMem2DEXT(enum target,
                        sizei levels,
                        enum internalFormat,
                        sizei width,
                        sizei height,
                        uint memory, // memory object
                        uint64 offset);

void BufferStorageMemEXT(enum target,
                         sizeiptr size,
                         uint memory, // memory object
                         uint64 offset);
```

### GL_EXT_memory_object_fd
- https://registry.khronos.org/OpenGL/extensions/EXT/EXT_external_objects_fd.txt

```cpp
#define HANDLE_TYPE_OPAQUE_FD_EXT 0x9586
void ImportMemoryFdEXT(uint memory,
                       uint64 size,
                       enum handleType,
                       int fd);
```

## Vulkan扩展

### VK_KHR_external_memory
- https://registry.khronos.org/vulkan/specs/1.3-extensions/man/html/VK_KHR_external_memory.html

### VK_KHR_external_memory_fd
- https://registry.khronos.org/vulkan/specs/1.3-extensions/man/html/VK_KHR_external_memory_fd.html
- An application may wish to reference device memory `in multiple Vulkan logical devices or instances`, `in multiple processes`, and/or `in multiple APIs`.
- This extension enables an application to `export POSIX file descriptor handles` from Vulkan memory objects and to import Vulkan memory objects from POSIX file descriptor handles exported from other Vulkan memory objects or from similar resources in other APIs.

```cpp
VkResult vkGetMemoryFdKHR(
    VkDevice                                    device,
    const VkMemoryGetFdInfoKHR*                 pGetFdInfo,
    int*                                        pFd);

typedef struct VkMemoryGetFdInfoKHR {
    VkStructureType                       sType;
    const void*                           pNext;
    VkDeviceMemory                        memory;
    VkExternalMemoryHandleTypeFlagBits    handleType;
} VkMemoryGetFdInfoKHR;
```

- `handleType` must be `VK_EXTERNAL_MEMORY_HANDLE_TYPE_OPAQUE_FD_BIT` or `VK_EXTERNAL_MEMORY_HANDLE_TYPE_DMA_BUF_BIT_EXT`
- `VK_EXTERNAL_MEMORY_HANDLE_TYPE_OPAQUE_FD_BIT` specifies a `POSIX file descriptor handle` that has only limited valid usage outside of Vulkan and other compatible APIs. It must be compatible with the POSIX system calls dup, dup2, close, and the non-standard system call dup3. Additionally, it must be transportable over a socket using an SCM_RIGHTS control message. It owns a reference to the underlying memory resource represented by its Vulkan memory object.
- `VK_EXTERNAL_MEMORY_HANDLE_TYPE_DMA_BUF_BIT_EXT` is a file descriptor for a `Linux dma_buf`. It owns a reference to the underlying memory resource represented by its Vulkan memory object.

## 参考

### Vulkan & OpenGL 纹理共享与同步
- https://zhuanlan.zhihu.com/p/620899883
- 代码：https://github.com/keith2018/SoftGLRender

```cpp
// 从创建好的 vkMemory 对象中获取共享用的 handle
VkMemoryGetHandleInfo memoryGetInfo{};
memoryGetInfo.sType = VK_STRUCTURE_TYPE_MEMORY_GET_HANDLE_INFO;
memoryGetInfo.memory = memory;
memoryGetInfo.handleType = VK_EXTERNAL_MEMORY_HANDLE_TYPE;
VK_CHECK(vkGetMemoryHandle(device_, &memoryGetInfo, &sharedMemory_.handle));

// 拿到 handle 后，就可以给到 OpenGL 进行 Memory Object 的创建
GL_CHECK(glCreateMemoryObjectsEXT(1, &sharedMemory_.glRef));
GL_CHECK(glImportMemory(sharedMemory_.glRef, sharedMemory_.allocationSize, GL_HANDLE_TYPE, sharedMemory_.handle));

// 最后，我们将 OpenGL 的 Memory Object 设置到 Texture
GL_CHECK(glTextureStorageMem2DEXT(texture, levels, internalFormat, width, height, sharedMemory_.glRef, 0));

// 到此，我们就完成了纹理共享的全过程，当然共享的 Memory 除了用于纹理，也可以用于 Buffer 对象
```
### OpenGL and Vulkan Interoperability on Linux
- https://eleni.mutantstargoat.com/hikiko/category/opengl-and-vulkan-interoperability/

#### [OpenGL and Vulkan Interoperability on Linux] Part 1: Introduction
- https://eleni.mutantstargoat.com/hikiko/gl-vk-interop-intro/

In order to reuse Vulkan allocated resources we usually need to:
1. Use the appropriate flags at allocation to allow “external” (OpenGL) access to them.
2. Use the `EXT_external_objects_fd` provided functions to take a file descriptor that points to that memory from Vulkan and give it to OpenGL in order to gain access too.
3. Create an OpenGL object that corresponds to that memory. Use that object like an ordinary OpenGL object (such as texture, buffer, depth or stencil buffer).
4. Synchronize the access to that object with a special type of semaphores when necessary (for example when OpenGL must wait for Vulkan to finish an operation before it uses the object or the opposite).

`EXT_external_objects` provide all the functions for the steps 3-4, step 1 is done in Vulkan, and `EXT_external_objects_fd` is required for step 2.

#### [OpenGL and Vulkan Interoperability on Linux] Part 2: Using OpenGL to draw on Vulkan textures.
- https://eleni.mutantstargoat.com/hikiko/vk-gl-interop-2/