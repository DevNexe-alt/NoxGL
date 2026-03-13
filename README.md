<div align="center">

<img src="documentation/docs/static/icon.png" alt="NoxGL Logo" width="100" />

# NoxGL

**Modern OpenGL toolkit for the Nox programming language.**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Nox](https://img.shields.io/badge/nox-0.1.2%2B-blueviolet)](https://github.com/devnexe/nox)
[![Backend](https://img.shields.io/badge/backend-GLFW%20%2B%20OpenGL%203.3-orange)](#)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)](#)

</div>

---

## What is NoxGL?

NoxGL is a complete OpenGL 3.3 core toolkit for [Nox](https://github.com/devnexe/nox). It wraps GLFW window management and modern OpenGL into a clean, idiomatic Nox API — no boilerplate, no raw C calls.

Write shaders, build meshes, load textures, handle input, and render 2D/3D scenes entirely in Nox.

---

## Installation

```bash
nox package install NoxGL
```

Or from GitHub directly:

```bash
nox package install devnexe-alt/NoxGL
```

---

## Quick Start

```nox
connect NoxGL as gl

app = gl.create("Hello NoxGL", 960, 540)

vs = """
#version 330 core
layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aColor;
out vec3 vColor;
void main() {
    gl_Position = vec4(aPos, 1.0);
    vColor = aColor;
}
"""

fs = """
#version 330 core
in vec3 vColor;
out vec4 FragColor;
void main() {
    FragColor = vec4(vColor, 1.0);
}
"""

prog = app.create_program(vs, fs)
if not prog["ok"]:
    display(prog["log"])
    app.should_close(true)

verts = [
     0.0,  0.7, 0.0,   1.0, 0.2, 0.2,
    -0.7, -0.6, 0.0,   0.2, 1.0, 0.2,
     0.7, -0.6, 0.0,   0.2, 0.4, 1.0,
]
mesh = gl.create_mesh_f32(app, verts, [3, 3])

define frame(g):
    g.use_program(prog["id"])
    mesh.draw()

app.run(frame)
```

---

## Features

| Category | What's included |
|---|---|
| **Window** | Create window, set title, resize, vsync, close |
| **Shaders** | Compile vertex/fragment shaders, link programs, detailed error logs |
| **Buffers** | VAO, VBO (float32), EBO (uint32), attribute layout helpers |
| **Textures** | RGBA textures, filtering, mipmaps, multi-unit binding |
| **Uniforms** | `mat4`, `int`, `float`, `vec2/3/4` setters |
| **Cameras** | `Camera2D` (ortho + zoom + rotation) and `Camera3D` (perspective + free-look) |
| **Math** | `vec3`, `mat4` — translate, scale, rotate X/Y/Z, ortho, perspective, look-at |
| **Input** | `key_down`, `mouse_down`, `mouse_pos`, `on_key`, `on_mouse`, `on_cursor` |
| **Diagnostics** | `check_gl` — drains the OpenGL error queue with human-readable names |

---

## API Reference

### Creating a Window

```nox
app = gl.create("My App", 1280, 720)

if not app.ready:
    display("Failed to initialize")
```

| Method | Description |
|---|---|
| `gl.create(title, width, height)` | Create window and initialize OpenGL context |
| `app.alive()` | Returns `true` while window is open |
| `app.begin(r, g, b, a)` | Clear framebuffer with given color |
| `app.end()` | Swap buffers and poll events |
| `app.run(frame_fn)` | Run the main loop calling `frame_fn(app)` each frame |
| `app.close()` | Destroy window and terminate GLFW |
| `app.should_close(value)` | Request window close |
| `app.set_title(title)` | Update window title |

---

### Shaders & Programs

```nox
prog = app.create_program(vertex_source, fragment_source)

if not prog["ok"]:
    display("Shader error: " + prog["log"])

app.use_program(prog["id"])
app.delete_program(prog["id"])
```

---

### Meshes

```nox
# layout = floats per attribute: [3, 3] = position(3) + color(3)
mesh = gl.create_mesh_f32(app, vertices, [3, 3])
mesh.draw()
mesh.destroy()

# Indexed mesh
mesh = gl.create_indexed_mesh_f32(app, vertices, [3, 2], indices)
mesh.draw()
mesh.destroy()
```

---

### Textures

```nox
# rgba_pixels = flat list of [R, G, B, A, R, G, B, A, ...]
tex = app.create_texture_rgba(width, height, rgba_pixels)

app.bind_texture(tex, 0)    # bind to texture unit 0
app.uniform1i(prog["id"], "u_texture", 0)

app.delete_texture(tex)
```

---

### Uniforms

```nox
app.uniform_mat4(prog["id"], "u_mvp", matrix)
app.uniform1i(prog["id"],   "u_tex", 0)
app.uniform1f(prog["id"],   "u_time", 1.5)
```

---

### Cameras

#### 2D Camera

```nox
cam = gl.camera2d()
cam.position = [100.0, 50.0]
cam.zoom = 2.0
cam.rotation = 15.0

mvp = cam.matrix(app.width, app.height)
app.uniform_mat4(prog["id"], "u_mvp", mvp)
```

#### 3D Camera

```nox
cam = gl.camera3d()
cam.fov = 75.0

define frame(g):
    g.enable_depth(true)
    view = cam.view_matrix()
    proj = cam.projection_matrix(g.width / g.height)
    mvp  = gl.mat4_mul(proj, view)
    g.use_program(prog["id"])
    g.uniform_mat4(prog["id"], "u_mvp", mvp)
    mesh.draw()
```

| Method | Description |
|---|---|
| `cam.move_forward(amount)` | Move along the look direction |
| `cam.move_right(amount)` | Strafe right |
| `cam.forward()` | Returns normalized forward vector |
| `cam.update_target()` | Recalculate target from yaw/pitch |

---

### Input

```nox
# Poll every frame
if app.key_down(65):   # A key
    cam.move_forward(0.1)

pos = app.mouse_pos()   # [x, y]

# Event callbacks
app.on_key(256, define handler(g, key, state):  # ESC
    g.should_close(true)
)

app.on_cursor(define handler(g, x, y, dx, dy):
    cam.yaw   = cam.yaw   + dx * 0.1
    cam.pitch = cam.pitch - dy * 0.1
    cam.update_target()
)

app.cursor_disabled(true)   # Capture cursor for FPS-style look
```

---

### Math Utilities

```nox
m = gl.mat4_identity()
m = gl.mat4_translate(x, y, z)
m = gl.mat4_scale(x, y, z)
m = gl.mat4_rotate_x(radians)
m = gl.mat4_rotate_y(radians)
m = gl.mat4_rotate_z(radians)
m = gl.mat4_ortho(left, right, bottom, top, near, far)
m = gl.mat4_perspective(fov_deg, aspect, near, far)
m = gl.mat4_look_at(eye, target, up)
m = gl.mat4_mul(a, b)

v = gl.vec3(x, y, z)
v = gl.v3_add(a, b)
v = gl.v3_sub(a, b)
v = gl.v3_dot(a, b)
v = gl.v3_cross(a, b)
v = gl.v3_norm(a)
```

---

### Diagnostics

```nox
# Check for GL errors — returns true if any were found
app.check_gl("after draw")
```

Output example:
```
NoxGL error at after draw
  -> GL_INVALID_OPERATION
```

---

## Complete 3D Example

```nox
connect NoxGL as gl

app = gl.create("3D Cube", 960, 540)
cam = gl.camera3d()
cam.fov = 60.0

vs = """
#version 330 core
layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aColor;
uniform mat4 u_mvp;
out vec3 vColor;
void main() {
    gl_Position = u_mvp * vec4(aPos, 1.0);
    vColor = aColor;
}
"""

fs = """
#version 330 core
in vec3 vColor;
out vec4 FragColor;
void main() {
    FragColor = vec4(vColor, 1.0);
}
"""

prog = app.create_program(vs, fs)

verts = [
    -0.5, -0.5,  0.5,  1.0, 0.3, 0.3,
     0.5, -0.5,  0.5,  0.3, 1.0, 0.3,
     0.5,  0.5,  0.5,  0.3, 0.3, 1.0,
    -0.5,  0.5,  0.5,  1.0, 1.0, 0.3,
]

indices = [0, 1, 2, 2, 3, 0]
mesh = gl.create_indexed_mesh_f32(app, verts, [3, 3], indices)

app.on_key(256, define handler(g, key, state):
    g.should_close(true)
)

define frame(g):
    g.enable_depth(true)
    aspect = g.width / g.height
    view = cam.view_matrix()
    proj = cam.projection_matrix(aspect)
    mvp  = gl.mat4_mul(proj, view)
    g.use_program(prog["id"])
    g.uniform_mat4(prog["id"], "u_mvp", mvp)
    mesh.draw()

app.run(frame)
```

---

## Platform Notes

NoxGL auto-detects the OpenGL runtime for your platform:

| Platform | Library |
|---|---|
| Windows | `opengl32.dll` via `opengl32.h` |
| macOS | `opengl_macos.h` |
| Linux | `libGL.so.1` |

GLFW is loaded similarly (`glfw3.dll` / `libglfw.so.3`). Make sure GLFW 3.x is installed on your system.

---

## Requirements

- Nox `0.1.2+`
- GLFW 3.x
- OpenGL 3.3 core profile capable GPU
- Python 3.8+ (via Nox runtime)

---

<div align="center">

MIT License — part of the [Nox](https://github.com/devnexe/nox) ecosystem

</div>
