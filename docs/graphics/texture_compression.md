# Texture Compression


## KTX

- https://www.khronos.org/ktx/
- `KTX (Khronos Texture)` is an efficient, lightweight `container format` for reliably distributing GPU textures to diverse platforms and applications. 
- The contents of a KTX file can range from a simple base-level `2D texture` to a `cubemap array texture with mipmaps`. 
- KTX files hold all the parameters needed for efficient texture loading into 3D APIs such as OpenGL® and Vulkan®, including access to individual mipmap levels
- `KTX 2.0` adds support for `Basis Universal` supercompressed GPU textures
- Khronos has released the `KHR_texture_basisu` extension that enables `glTF to contain KTX 2.0 textures`, resulting in universally distributable glTF assets that reduce download size and use natively supported texture formats to reduce GPU memory size and boost rendering speed on diverse devices and platforms.

![](https://www.khronos.org/assets/uploads/apis/2021-ktx-universal-gpu-compressed-textures.png)

## Basis Universal

- https://github.com/BinomialLLC/basis_universal
- Basis Universal is a "supercompressed" GPU texture data interchange system
- Supports two highly compressed intermediate file formats
  
  - `.basis` 
  - `.KTX2` open standard from the Khronos Group
- Can be quickly transcoded to a very wide variety of GPU compressed and uncompressed pixel format
- Supports two modes

  - UASTC
    - high quality mode
    - based off the `UASTC` compressed texture format
    - for extremely high quality (similar to BC7 quality)
  - ETC1S
    - lower quality mode
    - based off a subset of `ETC1`
    - for very small files