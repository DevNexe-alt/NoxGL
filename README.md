# NoxGL

NoxGL is an OpenGL toolkit for Nox built on top of GLFW.

It gives you a practical API for creating windows, compiling shaders, building
meshes, uploading textures, setting uniforms, handling input, and rendering 2D
or 3D scenes directly from Nox code.

## Features

- Window creation and frame loop helpers
- Shader compilation and program linking
- VAO, VBO, and EBO mesh utilities
- Texture upload and binding helpers
- Uniform setters for common OpenGL values
- Camera2D and Camera3D helpers
- Matrix and vector math utilities
- Keyboard, mouse, and cursor input callbacks
- OpenGL error diagnostics

## Installation

```bash
nox package install NoxGL
```

## Requirements

- Nox 0.1.4 or newer
- GLFW 3.x
- OpenGL 3.3 capable runtime

## Entry Point

The library entry file is `main.nox`.

## License

MIT
