# Fractal Inference Core (FIC)

FIC is a C++ inference engine that runs multiple reasoning agents in parallel on a shared blackboard. It handles rules with multiple conditions, propagates confidence scores through inference chains, and exposes a C API for integration with other languages.

## What it does

You give FIC a set of facts and a set of rules. Each rule says "if these conditions are true, then conclude this new fact". The engine runs multiple agents that randomly pick rules, match them against existing facts, and write new facts back to the blackboard. Over time, useful conclusions emerge from the collective exploration.

The whole thing is nondeterministic by design. Different runs may produce slightly different results, and that is intentional.

## Build

```

mkdir build && cd build
cmake .. -DFIC_BUILD_EXAMPLES=ON -DFIC_BUILD_TESTS=ON
make -j$(nproc)
ctest

```

Requires C++17 and CMake 3.16 or higher. The only external dependency is Threads.

## Quick example

```cpp
#include <fic/engine.h>

std::vector<fic::Rule> rules = {
    {
        {
            {"parent", "?x", "?y"},
            {"parent", "?y", "?z"}
        },
        {"grandparent", "?x", "?z"},
        0.95
    }
};

std::vector<fic::Fact> facts = {
    {"parent", "Alice", "Bob", 1.0},
    {"parent", "Bob", "Charlie", 1.0}
};

fic::FractalInferenceCore engine(rules, facts, 4, 100);
auto results = engine.infer(0.6);
```

The rules say: if A is parent of B, and B is parent of C, then A is grandparent of C with confidence 0.95. The engine takes two parent facts and derives the grandparent relation.

Building blocks

Blackboard
Shared storage for all facts. Multiple agents read and write concurrently. Fact confidence scores are stored and updated when higher confidence arrives.

Agents
Each agent runs in its own thread. It randomly selects a rule, tries to match its premises against existing facts, and writes the conclusion back if all premises match.

Engine
Orchestrates the agents, manages the blackboard, and collects results above a given confidence threshold.

C API
Headers in include/fic/c_api.h let you call FIC from C, Python, Go, Rust, or any language with FFI support.

Performance

On a typical desktop CPU, a single agent processes around 12,000 inference attempts per second. With 8 agents, throughput scales to about 6.4x. Memory usage stays around 2 MB per 100,000 facts.

These numbers vary by hardware and rule complexity. The engine is designed for moderate-scale knowledge tasks, not billion-triple knowledge graphs.

Examples

The examples/ directory contains two working programs:

· family_tree - Derives grandparent relationships from a small family graph. Run it to see transitive inference in action.
· diagnosis - Maps symptoms to possible illnesses using multi-premise rules. Demonstrates how confidence propagates from uncertain inputs.

Build them with the CMake option above and run from the build directory.

Testing

Tests use Google Test. After building, run ctest or execute fic_tests directly. The test suite covers blackboard operations, agent logic, and end-to-end inference with both C++ and C APIs.

What it is not

FIC is not a full expert system shell. It does not provide a rule language parser, a GUI, or a production rule system with priorities and conflict resolution. It is a library you embed when you need lightweight, concurrent inference in C++.

It is also not a graph database. Facts are stored in memory only and do not persist across runs.

Integration

To use FIC as a dependency in your CMake project:

```
find_package(fic REQUIRED)
target_link_libraries(your_app PRIVATE fic_engine)
```

Installation installs the headers and the CMake config files. The library is static by default.

For non-C++ projects, the C API in c_api.h is the entry point. It covers engine creation, inference, fact insertion, and result retrieval.

License

MIT

Contributing

Contributions are welcome. Run .clang-format before committing. Add tests for new features. Keep the dependencies minimal.

```
