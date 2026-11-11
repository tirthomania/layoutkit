# layoutkit — a C++ refresher shaped like EDA code

Seven small milestones that build a toy layout-geometry engine: rectangles, polygons,
STL queries, templates, a spatial index, connectivity extraction (union-find + a mini LVS),
and a Lee maze router. Every function you write has tests; the reference solutions are in
`solutions/` for when you're done (or truly stuck).

## Requirements

- A C++20 compiler: GCC 11+, Clang 14+, or Visual Studio 2022
- CMake 3.20+
- On Windows, WSL (Ubuntu) is the easiest route: `sudo apt install build-essential cmake gdb`
- Editor: VS Code with the *C/C++* or *clangd* extension. The build writes `compile_commands.json` for it.

## Build and test

```bash
cmake -S . -B build                        # configure (Debug by default)
cmake --build build -j                     # compile
ctest --test-dir build --output-on-failure # run all milestones
ctest --test-dir build -R m1 --output-on-failure   # just one milestone
./build/test_m1_geometry                   # or run a test binary directly
```

Each test prints `PASS`, `FAIL` (with the values it got vs expected) or `TODO` (you haven't
replaced the `todo(...)` call yet). The milestone passes when every line is `PASS`.

**Sanitizers on** (do this at least once per milestone — a bug ASan finds is a real bug even if tests pass):

```bash
cmake -S . -B build-asan -DLAYOUTKIT_SANITIZE=ON && cmake --build build-asan -j && ctest --test-dir build-asan --output-on-failure
```

**Reference solutions** (builds `solutions/` instead of `src/`; your files are untouched):

```bash
cmake -S . -B build-sol -DLAYOUTKIT_USE_SOLUTIONS=ON && cmake --build build-sol -j && ctest --test-dir build-sol
```

**Benchmark** (after M5; Release build or the numbers mean nothing):

```bash
cmake -S . -B build-rel -DCMAKE_BUILD_TYPE=Release && cmake --build build-rel -j && ./build-rel/bench_index
```

The reference grid index answers 2,000 window queries over 200,000 rectangles in about 2 ms;
brute force takes about 2 s. Can you match it? What bin size is best, and why?

## Milestones

| # | You write | File(s) | Skills practised |
|---|---|---|---|
| M1 | `Rect`: area, contains, overlaps vs touches, intersection, bloat, union | `src/geometry.cpp` | value types, `const`, `std::optional`, 64-bit coordinates |
| M2 | `Polygon`: shoelace area, bbox, rectilinear check, point-in-polygon, orientation | `src/polygon.cpp` | vectors, exact integer geometry, exceptions |
| M3 | `Layout` queries: by layer, by net, total area, sort by area, extent | `src/layout.cpp` | STL algorithms, lambdas, `unordered_map` |
| M4 | `Interval<T>`, merging intervals, free gaps on a routing track | `include/layoutkit/interval.hpp` | templates, concepts, defaulted `<=>` |
| M5 | `GridIndex`: uniform-grid spatial index | `src/spatial_index.cpp` | hashing, floor division, dedup idiom, performance |
| M6 | `UnionFind`, connected groups through vias, shorts and opens | `src/connectivity.cpp` | graphs, union-find, using your own index |
| M7 | Lee maze router and sequential multi-net routing | `src/maze_router.cpp` | BFS, `std::deque`, path reconstruction |

Do them in order: M5 uses M1, and M6 uses M3 and M5.

## How to work through a milestone

1. Read the header (`include/layoutkit/*.hpp`) — the comments are the spec.
2. Read the test file (`tests/test_m*.cpp`) — the tests are the examples.
3. Replace each `todo("...")` call in `src/` with your implementation.
4. Build, run that milestone's tests, fix, repeat. Then rerun with sanitizers.
5. Paste your file into Gemini with the code-review prompt from the study guide; fix what it flags.
6. Only then compare with `solutions/`. Note one thing the solution does better, and one thing yours does better.

Stuck for more than 45 minutes on one function? Ask Gemini for a *hint*, not the code. Paste
the failing test output — it shows the exact values you got vs expected.

## Stretch goals 

- Replace the test framework with GoogleTest (`FetchContent` in CMake).
- M5: add `remove(id)`; then compare against `boost::geometry::index::rtree`.
- M6: find all overlapping pairs with a sweep line instead of the grid. Which is faster here?
- M7: switch BFS to A* with a Manhattan-distance heuristic; add a cost for bends.
- Add Python bindings with pybind11 and call `connectedGroups` from a Jupyter notebook.
