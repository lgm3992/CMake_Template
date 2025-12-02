# CLAUDE.md - AI Assistant Guide for CMake_Template

> **Last Updated:** December 2, 2025
> **Repository:** CMake_Template
> **Purpose:** OpenGL 3.3+ Graphics Application Template using Modern C++

This document provides comprehensive guidance for AI assistants working on this codebase.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Build System](#build-system)
4. [Dependencies](#dependencies)
5. [Code Architecture](#code-architecture)
6. [Code Conventions](#code-conventions)
7. [Development Workflow](#development-workflow)
8. [Common Tasks](#common-tasks)
9. [Testing and Debugging](#testing-and-debugging)
10. [Troubleshooting](#troubleshooting)

---

## Project Overview

### Purpose
This is a **CMake-based OpenGL template project** demonstrating modern C++ graphics programming with:
- OpenGL 3.3+ Core Profile rendering
- GLFW window and input management
- GLAD OpenGL function loading
- spdlog structured logging
- Clean, extensible architecture for graphics applications

### Current State
- **Functionality:** Renders a single white point at screen center (0.5, 0.5)
- **Stage:** Template/starter project ready for extension
- **Platform:** Cross-platform (Windows, Linux, macOS)
- **Language:** C++17

### Key Technologies
- **Build System:** CMake 3.13+
- **Graphics API:** OpenGL 3.3 Core Profile
- **Window/Input:** GLFW 3.3.5
- **Function Loader:** GLAD v0.1.36
- **Logging:** spdlog v1.x

---

## Codebase Structure

```
CMake_Template/
├── CMakeLists.txt              # Main build configuration
├── Dependency.cmake            # External dependency management
├── README.md                   # Project documentation (Korean)
├── .gitignore                  # Git exclusions (build/)
├── .vscode/
│   └── settings.json          # VS Code C++ settings
├── src/                       # C++ source code
│   ├── main.cpp               # Application entry point (87 lines)
│   ├── common.h/.cpp          # Utility macros and file loading (13 lines)
│   ├── context.h/.cpp         # Rendering context management (35 lines)
│   ├── shader.h/.cpp          # Shader compilation (41 lines)
│   └── program.h/.cpp         # Shader program linking (32 lines)
└── shader/                    # GLSL shader files
    ├── simple.vs              # Vertex shader (GLSL 330)
    └── simple.fs              # Fragment shader (GLSL 330)
```

**Total Lines of Code:** ~296 lines (excluding dependencies)

### File Locations
- **Source Code:** All in `src/` directory
- **Shaders:** In `shader/` directory (paths relative to executable)
- **Build Output:** `build/` directory (git-ignored)
- **Dependencies:** Auto-downloaded to `build/dep_*-prefix/src/`
- **Installed Libraries:** `build/install/` (includes and libs)

---

## Build System

### CMake Configuration

**File:** `CMakeLists.txt`

**Key Settings:**
```cmake
cmake_minimum_required(VERSION 3.13)
set(PROJECT_NAME CMake_Template)
set(CMAKE_CXX_STANDARD 17)

# Window configuration (injected as compile definitions)
set(WINDOW_NAME "Hello, OpenGL!")
set(WINDOW_WIDTH 960)
set(WINDOW_HEIGHT 540)
```

**Compilation Flags:**
- MSVC: `/utf-8` for UTF-8 source file handling
- All platforms: C++17 standard enabled

**Build Targets:**
- Single executable: `CMake_Template`
- All source files compiled into one binary

### Build Commands

```bash
# First-time setup (downloads and builds dependencies)
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build the project
cmake --build build

# Run the executable
./build/CMake_Template
```

**Build Time:**
- First build: 3-5 minutes (downloads GLFW, GLAD, spdlog)
- Incremental builds: <10 seconds

### Compile-Time Configuration

Window properties are **compile-time macros** defined in `CMakeLists.txt:38-42`:
```cmake
target_compile_definitions(${PROJECT_NAME} PUBLIC
  WINDOW_NAME="${WINDOW_NAME}"
  WINDOW_WIDTH=${WINDOW_WIDTH}
  WINDOW_HEIGHT=${WINDOW_HEIGHT}
)
```

**To change window settings:**
1. Edit values in `CMakeLists.txt` lines 13-15
2. Rebuild with `cmake --build build`

---

## Dependencies

### Dependency Management

**File:** `Dependency.cmake`

**Method:** CMake `ExternalProject_Add` (automatic download and build)

### External Libraries

#### 1. **spdlog v1.x** - Fast C++ Logging
- **Repository:** https://github.com/gabime/spdlog.git
- **Purpose:** Structured logging with `SPDLOG_INFO()`, `SPDLOG_ERROR()` macros
- **Library:** `spdlog` (Debug: `spdlogd`)
- **Usage:** Throughout codebase for initialization and error reporting

#### 2. **GLFW 3.3.5** - Window and Input
- **Repository:** https://github.com/glfw/glfw.git
- **Tag:** v3.3.5
- **Purpose:** Cross-platform window creation, OpenGL context, event handling
- **Library:** `glfw3`
- **Build Options:** Examples, tests, docs disabled
- **Usage:** `src/main.cpp` for window lifecycle

#### 3. **GLAD v0.1.36** - OpenGL Function Loader
- **Repository:** https://github.com/Dav1dde/glad
- **Tag:** v0.1.36
- **Purpose:** Load OpenGL 3.3 Core Profile function pointers
- **Library:** `glad`
- **Usage:** `gladLoadGLLoader()` in `main.cpp:58`

### Dependency Installation

Dependencies install to:
```
build/install/
├── include/           # Headers (glad/, GLFW/, spdlog/)
└── lib/              # Static libraries (.a / .lib)
```

CMake variables for linking:
- `DEP_INCLUDE_DIR` - Include directory path
- `DEP_LIB_DIR` - Library directory path
- `DEP_LIBS` - Library names for linking
- `DEP_LIST` - Dependency targets

---

## Code Architecture

### Layered Design

```
main.cpp (Entry Point)
    ↓
Context (Rendering Manager)
    ↓
Program (Shader Program)
    ↓
Shader (Individual Shaders)
    ↓
OpenGL (via GLAD/GLFW)
```

### Class Hierarchy

#### **Context** (`src/context.h/.cpp`)
**Purpose:** Manages rendering state and OpenGL operations

**Key Methods:**
- `static ContextUPtr Create()` - Factory method (returns nullptr on failure)
- `void Render()` - Called each frame to render scene
- `bool Init()` - Private initialization (loads shaders, creates program)

**Current Rendering:**
```cpp
void Context::Render() {
    glClear(GL_COLOR_BUFFER_BIT);
    m_program->Use();
    glDrawArrays(GL_POINTS, 0, 1);  // Draws single point
}
```

**Member Variables:**
- `ProgramUPtr m_program` - Shader program (vertex + fragment)

---

#### **Program** (`src/program.h/.cpp`)
**Purpose:** Links multiple shaders into an executable GPU program

**Key Methods:**
- `static ProgramUPtr Create(const std::vector<ShaderPtr>& shaders)` - Links shaders
- `void Use()` - Binds program for rendering (`glUseProgram()`)
- `uint32_t Get()` - Returns OpenGL program handle

**Lifecycle:**
1. Create shaders individually
2. Pass shader vector to `Program::Create()`
3. Link validation with error logging
4. Destructor auto-deletes OpenGL program (`glDeleteProgram()`)

---

#### **Shader** (`src/shader.h/.cpp`)
**Purpose:** Compiles individual GLSL shaders (vertex/fragment)

**Key Methods:**
- `static ShaderUPtr CreateFromFile(const std::string& filename, GLenum shaderType)` - Loads and compiles shader from file
- `uint32_t Get()` - Returns OpenGL shader handle

**Shader Types:**
- `GL_VERTEX_SHADER` - Vertex shader
- `GL_FRAGMENT_SHADER` - Fragment shader

**File Loading:**
1. Uses `LoadTextFile()` from `common.cpp`
2. Compiles with `glShaderSource()` + `glCompileShader()`
3. Validates compilation with detailed error logging
4. Destructor auto-deletes shader (`glDeleteShader()`)

---

#### **Common Utilities** (`src/common.h/.cpp`)

**CLASS_PTR Macro:**
```cpp
#define CLASS_PTR(klassName) \
    class klassName; \
    using klassName ## UPtr = std::unique_ptr<klassName>; \
    using klassName ## Ptr = std::shared_ptr<klassName>; \
    using klassName ## WPtr = std::weak_ptr<klassName>;
```

**Generates smart pointer typedefs:**
- `ContextUPtr` → `std::unique_ptr<Context>`
- `ContextPtr` → `std::shared_ptr<Context>`
- `ContextWPtr` → `std::weak_ptr<Context>`

**LoadTextFile Function:**
```cpp
std::optional<std::string> LoadTextFile(const std::string& filename);
```
- Returns file contents or `std::nullopt` on failure
- Logs errors with spdlog

---

### Application Flow

**`src/main.cpp` Initialization:**

1. **GLFW Initialization** (lines 35-41)
   - `glfwInit()` with error handling
   - Sets OpenGL 3.3 Core Profile hints

2. **Window Creation** (lines 48-54)
   - Uses `WINDOW_WIDTH`, `WINDOW_HEIGHT`, `WINDOW_NAME` macros
   - Error handling with `glfwTerminate()` cleanup

3. **GLAD Initialization** (lines 58-64)
   - Loads OpenGL function pointers
   - Logs OpenGL version

4. **Context Creation** (lines 66-71)
   - `Context::Create()` factory method
   - Loads shaders: `shader/simple.vs`, `shader/simple.fs`
   - Creates shader program

5. **Event Callbacks** (lines 73-75)
   - `OnFramebufferSizeChange` - Adjusts viewport on resize
   - `OnKeyEvent` - ESC key closes window

6. **Main Loop** (lines 78-83)
   - `glfwPollEvents()` - Process input
   - `context->Render()` - Draw frame
   - `glfwSwapBuffers()` - Display frame

7. **Cleanup** (lines 84-87)
   - `context.reset()` - Destroy Context (deletes OpenGL resources)
   - `glfwTerminate()` - Cleanup GLFW

---

## Code Conventions

### Memory Management

**RAII and Smart Pointers:**
- **Exclusive ownership:** `std::unique_ptr<>` (most common)
- **Shared ownership:** `std::shared_ptr<>` (for shaders in program)
- **Weak references:** `std::weak_ptr<>` (currently unused)
- **Destructors handle cleanup:** OpenGL resource deletion automatic

**No manual `delete` calls - RAII ensures cleanup**

### Error Handling

**Pattern:** No exceptions; use return values and `std::optional<T>`

```cpp
// Factory methods return nullptr on failure
auto context = Context::Create();
if (!context) {
    SPDLOG_ERROR("failed to create context");
    return -1;
}

// File operations return std::optional
auto code = LoadTextFile(filename);
if (!code) {
    SPDLOG_ERROR("failed to load file: {}", filename);
    return nullptr;
}
```

**Logging at every failure point** - Makes debugging straightforward

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Classes | PascalCase | `Context`, `Shader`, `Program` |
| Member Variables | `m_` prefix + camelCase | `m_program`, `m_shader` |
| Functions/Methods | camelCase | `LoadTextFile`, `Get()`, `Render()` |
| Macros | UPPER_SNAKE_CASE | `CLASS_PTR`, `WINDOW_WIDTH` |
| Header Guards | `__CLASSNAME_H__` | `__CONTEXT_H__`, `__SHADER_H__` |
| Local Variables | camelCase | `window`, `glVersion`, `context` |

### Include Order

```cpp
// 1. Corresponding header (for .cpp files)
#include "context.h"

// 2. Project headers
#include "common.h"
#include "shader.h"

// 3. External libraries
#include <spdlog/spdlog.h>
#include <glad/glad.h>
#include <GLFW/glfw3.h>

// 4. Standard library
#include <memory>
#include <optional>
#include <string>
```

### Class Structure Pattern

```cpp
class ClassName {
public:
    // Factory method (returns smart pointer)
    static ClassNameUPtr Create();

    // Public interface
    void PublicMethod();
    uint32_t Get() const { return m_member; }

private:
    // Private constructor (forces use of factory)
    ClassName() {}

    // Private initialization
    bool Init();

    // Member variables (m_ prefix)
    uint32_t m_member;
};
```

**Rationale:** Factory pattern enables exception-free initialization with nullptr return on failure

---

## Development Workflow

### Git Workflow

**Current Branch:** `claude/claude-md-mioztnks59trgw2d-01L61AKzipnkNtH5Ltozfc5Q`

**Commit Pattern:**
- Frequent commits with descriptive messages
- Focus on incremental features
- Recent history shows documentation updates and rendering additions

**Branch Naming:**
- Claude-generated branches start with `claude/`
- Follow with descriptive identifier

### Development Cycle

1. **Make changes** to source files
2. **Rebuild** with `cmake --build build`
3. **Test** by running `./build/CMake_Template`
4. **Check logs** in terminal (spdlog output)
5. **Commit** when feature is complete

### Adding New Features

**Typical workflow:**

1. **Modify `Context` class** for new rendering features
   - Update `Context::Init()` for initialization
   - Update `Context::Render()` for drawing

2. **Add member variables** to `Context` class
   - Follow `m_` naming convention
   - Use smart pointers for owned resources

3. **Update shaders** if needed
   - Modify `shader/simple.vs` or `shader/simple.fs`
   - No rebuild needed (loaded at runtime)

4. **Add dependencies** if required
   - Edit `Dependency.cmake` with `ExternalProject_Add`
   - Update `DEP_LIBS` list
   - Rebuild to download and link

---

## Common Tasks

### Task 1: Change Window Size or Title

**File:** `CMakeLists.txt` lines 13-15

```cmake
set(WINDOW_NAME "Hello, OpenGL!")  # Change title
set(WINDOW_WIDTH 960)              # Change width
set(WINDOW_HEIGHT 540)             # Change height
```

**After changing:** Rebuild with `cmake --build build`

---

### Task 2: Modify Rendering (Colors, Geometry)

**File:** `src/context.cpp`

**Example: Change background color**

```cpp
bool Context::Init() {
    // ... existing shader loading ...

    // Change clear color (currently black)
    glClearColor(0.1f, 0.2f, 0.3f, 1.0f);  // Dark blue background
    return true;
}
```

**Example: Draw triangle instead of point**

1. Create vertex buffer in `Context::Init()`
2. Update `Context::Render()` to use `GL_TRIANGLES`
3. Modify shaders to accept vertex positions

---

### Task 3: Add New Shader

**Steps:**

1. **Create shader file** in `shader/` directory
   ```glsl
   // shader/new_shader.vs
   #version 330 core
   layout (location = 0) in vec3 aPos;
   void main() {
       gl_Position = vec4(aPos, 1.0);
   }
   ```

2. **Load in Context::Init()**
   ```cpp
   auto vs = Shader::CreateFromFile("shader/new_shader.vs", GL_VERTEX_SHADER);
   auto fs = Shader::CreateFromFile("shader/new_shader.fs", GL_FRAGMENT_SHADER);
   if (!vs || !fs) return false;

   m_program = Program::Create({vs, fs});
   ```

3. **Rebuild and run** - Shader loaded at runtime

---

### Task 4: Add External Library (e.g., GLM for Math)

**File:** `Dependency.cmake`

**Add GLM dependency:**

```cmake
# After existing ExternalProject_Add blocks:
ExternalProject_Add(
    dep_glm
    GIT_REPOSITORY "https://github.com/g-truc/glm.git"
    GIT_TAG "0.9.9.8"
    GIT_SHALLOW 1
    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${DEP_INSTALL_DIR}
        -DGLM_TEST_ENABLE=OFF
    TEST_COMMAND ""
)

# Update dependency list
set(DEP_LIST ${DEP_LIST} dep_glm)

# GLM is header-only, no library to link
```

**Then use in code:**
```cpp
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
```

---

### Task 5: Add Vertex Buffers (VAO/VBO)

**Extend `Context` class:**

```cpp
// context.h
class Context {
public:
    // ... existing ...
private:
    ProgramUPtr m_program;
    uint32_t m_vao;  // Vertex Array Object
    uint32_t m_vbo;  // Vertex Buffer Object
};

// context.cpp
bool Context::Init() {
    // ... existing shader loading ...

    // Create triangle vertices
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,
         0.5f, -0.5f, 0.0f,
         0.0f,  0.5f, 0.0f
    };

    glGenVertexArrays(1, &m_vao);
    glGenBuffers(1, &m_vbo);

    glBindVertexArray(m_vao);
    glBindBuffer(GL_ARRAY_BUFFER, m_vbo);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    return true;
}

void Context::Render() {
    glClear(GL_COLOR_BUFFER_BIT);
    m_program->Use();
    glBindVertexArray(m_vao);
    glDrawArrays(GL_TRIANGLES, 0, 3);  // Draw triangle
}

// Add destructor for cleanup
Context::~Context() {
    if (m_vao) glDeleteVertexArrays(1, &m_vao);
    if (m_vbo) glDeleteBuffers(1, &m_vbo);
}
```

**Update `context.h`:**
```cpp
class Context {
public:
    static ContextUPtr Create();
    void Render();
    ~Context();  // Add destructor declaration
private:
    // ...
};
```

---

### Task 6: Debug Shader Compilation Errors

**Location:** `src/shader.cpp` lines 20-30

Shader compilation errors are logged with:
```cpp
SPDLOG_ERROR("failed to compile shader: \"{}\"", filename);
SPDLOG_ERROR("reason: {}", infoLog);
```

**Check terminal output** for error messages when shader fails to compile

**Common issues:**
- GLSL version mismatch (use `#version 330 core`)
- Syntax errors in shader code
- File path incorrect (relative to executable, not source)

---

### Task 7: Add Input Handling

**File:** `src/main.cpp` - Extend `OnKeyEvent` function

**Example: Toggle wireframe mode with 'W' key**

```cpp
void OnKeyEvent(GLFWwindow* window, int key, int scancode, int action, int mods) {
    // ... existing logging ...

    if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS) {
        glfwSetWindowShouldClose(window, true);
    }

    // Add wireframe toggle
    if (key == GLFW_KEY_W && action == GLFW_PRESS) {
        static bool wireframe = false;
        wireframe = !wireframe;
        glPolygonMode(GL_FRONT_AND_BACK, wireframe ? GL_LINE : GL_FILL);
        SPDLOG_INFO("Wireframe mode: {}", wireframe ? "ON" : "OFF");
    }
}
```

**Need context access?** Pass context pointer to callbacks:
```cpp
glfwSetWindowUserPointer(window, context.get());
```

---

## Testing and Debugging

### Current Testing State

**No formal testing framework** - Manual testing only

**Testing approach:**
1. Build project
2. Run executable
3. Check terminal logs (spdlog output)
4. Visually verify rendering in window

### Debugging Strategies

#### 1. **Check Logs**
spdlog provides detailed initialization logging:
```
[info] Start program
[info] Initialize glfw
[info] Create glfw window
[info] OpenGL context version: 3.3.0
[info] Start main loop
```

#### 2. **Shader Debugging**
- Compilation errors logged with line numbers
- Linking errors show failed attachments
- Use `glGetError()` for runtime OpenGL errors

#### 3. **Build Issues**
- Check CMake output for dependency download failures
- Verify `build/install/` contains headers and libraries
- GLAD may need manual CMake version fix (see README.md)

#### 4. **Runtime Crashes**
- Check null pointer returns from factory methods
- Verify shader file paths (relative to executable)
- Validate OpenGL context creation succeeded

---

## Troubleshooting

### Issue: GLAD Build Failure

**Symptom:** CMake fails during GLAD dependency build

**Solution (from README.md):**
1. After first build attempt, edit:
   ```
   build/dep_glad-prefix/src/dep_glad/CMakeLists.txt
   ```
2. Change first line to:
   ```cmake
   cmake_minimum_required(VERSION 3.13)
   ```
3. Rebuild: `cmake --build build`

---

### Issue: Shader Files Not Found

**Symptom:** Error log shows "failed to load file: shader/simple.vs"

**Cause:** Shader paths are relative to **executable working directory**, not source directory

**Solution:**
- Run executable from project root: `./build/CMake_Template`
- Or create `shader/` symlink in build directory
- Or use absolute paths in `Context::Init()`

---

### Issue: Black Screen (Nothing Renders)

**Debugging checklist:**
1. Check shader compilation succeeded (no errors in logs)
2. Verify program linking succeeded
3. Check `glClearColor()` isn't same as geometry color
4. Ensure viewport size matches window: `glViewport(0, 0, width, height)`
5. Verify `glDrawArrays()` count matches geometry

---

### Issue: Window Doesn't Respond to Input

**Check:**
1. Callbacks registered: `glfwSetKeyCallback(window, OnKeyEvent)`
2. `glfwPollEvents()` called each frame
3. Event callback functions have correct signature

---

### Issue: Dependency Download Fails

**Symptom:** CMake can't download GLFW, GLAD, or spdlog

**Solutions:**
- Check internet connection
- Verify Git installed and accessible
- Try clearing build directory: `rm -rf build/`
- Check firewall/proxy settings for Git HTTPS access

---

### Issue: Linker Errors with Dependencies

**Symptom:** Undefined references to GLFW, GLAD, or spdlog functions

**Check:**
1. Dependencies built successfully (check `build/install/lib/`)
2. `DEP_LIBS` includes all required libraries in `Dependency.cmake`
3. Platform-specific library names (`.a` vs `.lib`, `.so` vs `.dll`)
4. Debug/Release library naming (e.g., `spdlogd` for debug)

---

## Future Enhancements

### Potential Additions
1. **Geometry System**
   - VAO/VBO/EBO buffer objects
   - Mesh loading (OBJ, FBX)
   - Vertex attributes (position, normal, UV)

2. **Transformation System**
   - GLM library integration
   - Model/View/Projection matrices
   - Camera class

3. **Texture System**
   - Image loading (STB Image)
   - Texture binding and sampling
   - Texture parameters

4. **Testing Framework**
   - Google Test integration
   - Unit tests for shader compilation
   - Render output validation

5. **CI/CD Pipeline**
   - GitHub Actions for builds
   - Automated testing
   - Cross-platform builds

6. **Documentation**
   - Doxygen code documentation
   - API reference generation
   - Tutorial/example code

---

## Key Insights for AI Assistants

### Design Philosophy
- **Minimal Coupling:** Each class has single responsibility
- **Extensible Architecture:** Easy to add features without modifying core
- **RAII Everywhere:** No manual resource management
- **Exception-Free:** Error handling via return values and logging

### When Modifying Code

1. **ALWAYS read files before editing** - Don't assume structure
2. **Follow existing patterns** - Use factory methods, smart pointers, RAII
3. **Log errors comprehensively** - Use spdlog for all failure points
4. **Test incrementally** - Build and run after each change
5. **Respect naming conventions** - Match existing code style
6. **Don't over-engineer** - Keep solutions simple and focused

### Common Pitfalls to Avoid

1. **Don't create raw pointers** - Use smart pointers exclusively
2. **Don't forget destructors** - OpenGL resources need cleanup
3. **Don't use exceptions** - Codebase is exception-free by design
4. **Don't hardcode paths** - Shader paths should be configurable
5. **Don't skip error checks** - Every OpenGL call can fail
6. **Don't modify CMake carelessly** - Dependency system is fragile

### When Adding Features

**Think in layers:**
1. What OpenGL resources are needed? (buffers, textures, uniforms)
2. How should Context manage them? (member variables, lifecycle)
3. What initialization is needed? (add to `Context::Init()`)
4. What per-frame updates? (add to `Context::Render()`)
5. What cleanup is needed? (add to destructor)

**Example mental model for adding textures:**
- OpenGL resource: Texture ID (`GLuint`)
- Context manages: `uint32_t m_texture` member
- Init: `glGenTextures()`, load image, `glTexImage2D()`
- Render: `glBindTexture()` before drawing
- Cleanup: `glDeleteTextures()` in destructor

---

## Quick Reference

### Build Commands
```bash
cmake -B build                    # Configure
cmake --build build               # Build
./build/CMake_Template            # Run
rm -rf build/                     # Clean (forces dependency re-download)
```

### Key Files to Edit
- `src/context.cpp` - Rendering logic
- `src/main.cpp` - Application lifecycle
- `CMakeLists.txt` - Window config, dependencies
- `shader/*.vs/.fs` - GLSL shaders

### Important Line Numbers
- `CMakeLists.txt:13-15` - Window configuration
- `src/main.cpp:66-71` - Context creation
- `src/context.cpp:Init()` - Shader loading and setup
- `src/context.cpp:Render()` - Per-frame rendering

### OpenGL Version
- **Required:** OpenGL 3.3 Core Profile
- **Configured:** `main.cpp:43-45` (GLFW hints)
- **Verified:** `main.cpp:63` (log GL version)

---

## Contact and Resources

### README Notes
- Original README in Korean
- Notes about GLAD CMakeLists.txt manual fix requirement
- Compile from project root directory

### External Documentation
- **OpenGL:** https://docs.gl/ (function reference)
- **GLFW:** https://www.glfw.org/docs/latest/
- **GLAD:** https://glad.dav1d.de/ (online generator)
- **spdlog:** https://github.com/gabime/spdlog
- **CMake:** https://cmake.org/documentation/

---

**End of CLAUDE.md**
