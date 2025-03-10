# CImGui
This package provides a Julia language wrapper for
[cimgui](https://github.com/cimgui/cimgui): a thin C wrapper
programmatically generated for the excellent C++ immediate mode gui [Dear
ImGui](https://github.com/ocornut/imgui). Dear ImGui is mainly for creating
content creation tools and visualization / debug tools. You could browse
[Gallery](https://github.com/ocornut/imgui/issues/2265) to get an idea of its
use cases.

These docs were generated against this upstream Dear ImGui version:
```@repl
import CImGui as ig
ig.imgui_version()
```

## Installation
```julia-repl
pkg> add CImGui
```

You will probably also want to add the necessary packages for one of the
backends. Currently we only support OpenGL3/GFLW, so you'd need:
```julia-repl
pkg> add ModernGL, GLFW
```

Note that you don't have to explicitly use these packages, but they must be
installed and loaded because the backend renderloop is a package extension that
requires them.

## How to start
#### 1. Run `demo/demo.jl` to test whether the official demo works at all
```julia-repl
julia> import CImGui, ModernGL, GLFW
julia> include(joinpath(pathof(CImGui), "..", "..", "demo", "demo.jl"))
julia> official_demo()
```

#### 2. Run `examples/demo.jl` and browse demos in the `examples` folder to learn how to use the API
```julia-repl
julia> import CImGui, ModernGL, GLFW
julia> include(joinpath(pathof(CImGui), "..", "..", "examples", "demo.jl"))
julia> julia_demo()
```

[All of these
examples](https://github.com/JuliaImGui/CImGui.jl/tree/master/examples) are
one-to-one ported from [Dear ImGui's C++
examples](https://github.com/ocornut/imgui/blob/master/imgui_demo.cpp) and there
is an [interactive
manual](https://pthom.github.io/imgui_manual_online/manual/imgui_manual.html)
for quickly locating the code. You could also run `? CImGui.xxx` to retrieve
docs:
```
help?> CImGui.Button
  Button(label) -> Bool
  Button(label, size) -> Bool

  Return true when the value has been changed or when pressed/selected.
```

#### 3. Your first GUI \o/
Note that all ImGui widgets should run within `CImGui.Begin()`...`CImGui.End()`,
if not, a crash is waiting for you. For example, directly running
`CImGui.Button("My button")` in REPL will crash Julia. That being said, here's
how a quick example of how to use the default backend to draw a little GUI:
```julia-repl
julia> import CImGui as ig, ModernGL, GLFW
julia> ig.set_backend(:GlfwOpenGL3)
julia> ctx = ig.CreateContext()
julia> ig.render(ctx; window_size=(360, 480), window_title="ImGui Window") do
           ig.Begin("Hello ImGui")
           if ig.Button("My Button")
               @info "Triggered"
           end
           ig.End()
       end
```

Note that neither ImGui nor OpenGL are thread-safe, and because of this
[`CImGui.render()`](@ref) will pin the renderloop to thread 1 by default (see the
docstring for more information).

## Usage
The API provided in this package is as close as possible to the original C++
API. When translating C++ code to Julia, please follow the tips below:
- Replace `ImGui::` to `CImGui.` (or `ig.` if you've run `import CImGui as ig`).
- `using CImGui.lib` to import all of the `ImGuiXXX` types into the current namespace.
- Member function calling should be translated in Julia style:
  `fonts.AddFont(cfg)` => `ig.AddFont(fonts, cfg)`.
- [`using CImGui.CSyntax`] provides two useful macros: `@c` for translating C's
  `&` operator on immutables and `@cstatic`-block for emulating C's `static`
  keyword.

As mentioned before, this package aims to provide the same user experience as
the original C++ API, so any high-level abstraction should go into a more
high-level package. [`Redux.jl`](https://github.com/Gnimuc/Redux.jl) might be of
interest to you if you're looking for state management frameworks.

For writing tests you can use
[ImGuiTestEngine.jl](https://juliaimgui.github.io/ImGuiDocs.jl/imguitestengine).
