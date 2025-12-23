## Common Mistakes to Avoid
- **Never create build directories manually** - Use the unified test system with `utilities/run_tests.sh`
- **Never build from `core/` or `plugins/` directories** - Always use the unified test script which handles all building automatically
- **Don't be too agreeable** - Scrutinize the user's assumptions and suggestions. If they suggest something you do not agree with, don't just automatically implement it. Bring this to their attention and wait for a rebuttal.  
- **Don't implement fallbacks** - Helios follows a fail-fast philosophy - never implement silent fallbacks that hide issues from users. NEVER return fake values (0.0, empty lists, fake IDs), silently catch and ignore exceptions, or continue with misleading fallback functionality when core features fail. Instead, always raise explicit `helios_runtime_error()` with clear, actionable error messages that explain what failed, why it failed, and how to fix it. 
- **Don't implement placeholders** - When implementing new functionality, do not leave placeholder code that does not actually do anything. If you are not sure how to implement a function, leave it unimplemented and raise a `helios_runtime_error()` with a clear message explaining that the function is not yet implemented. This will help users understand that they need to implement the functionality before it can be used.

## CRITICAL Git Safety Rules
**NEVER USE THESE DESTRUCTIVE GIT COMMANDS:**
- **NEVER use `git checkout HEAD`** - This overwrites ALL uncommitted changes and destroys work-in-progress
- **NEVER use `git reset --hard`** - This permanently destroys uncommitted changes 
- **NEVER use `git clean -fd`** - This deletes untracked files that may contain important work
- **NEVER use `git checkout .`** - This discards all local changes in the working directory
- **NEVER use `git stash --include-untracked && git stash drop`** - This permanently destroys stashed changes

**Safe Git Commands Only:**
- Use `git status` to check repository state
- Use `git diff` to see changes before committing
- Use `git add` to stage specific files
- Use `git commit` to save changes with descriptive messages
- If you need to discard changes to a specific file, ask the user first and use `git checkout -- filename` only for that specific file
- If repository state is unclear, ask the user how they want to proceed rather than making destructive changes

**Before Any Git Command:**
1. Always run `git status` first to understand the current state
2. If there are uncommitted changes, ask the user what they want to do with them
3. Never make assumptions about what changes the user wants to keep or discard
4. When in doubt, do nothing and ask the user for explicit instructions

## Sub-Agents
- Make sure to familiarize yourself with the available sub-agents (@.claude/agents/) and use them efficiently:
  - context-gatherer: Expert code archaeologist and information synthesizer specializing in rapidly discovering, analyzing, and summarizing relevant context for development tasks.
  - research-specialist: Expert technical web researcher specializing in finding and evaluating open source codebases, scientific literature, and algorithmic implementation.
  - code-architect: Analyzes existing codebases holistically and create comprehensive implementation plans that consider architectural integrity, maintainability, and scalability.
  - debug-specialist: Expert software debugging specialist with deep C++ and cmake expertise.
  - performance-optimizer: Specialist in code profiling, performance analysis, and bottleneck identification.
  - test-coverage-maximizer: Writes tests that achieve maximum code coverage while ensuring robust functional testing that catches real bugs and edge cases.

## Code Style
- Code style should follow the style given in `.clang-format`.
- Standard C++ library include headers are listed in the file `core/include/global.h`. Check these includes to make sure you do not add unnecessary includes.
- Prefer descriptive variable names that 'self-document' the code.
- Prefer clearer code over clever optimized code that provides only marginal performance improvements.
- When implementing new function/method definitions, be aware of the organization of the source file. Don't just automatically add them at the end of the file, but keep them grouped with similar definitions if possible.
- Use `helios_runtime_error()` for runtime errors. Don't manually throw exceptions or write to std::cerr and exit.

## Code Structure
- The code is organized into a core library and plugins. The core library contains the main functionality, while plugins provide additional features.
- CMakeLists.txt files inside `core` and `plugins` do not build by themselves. You need to build a project that uses `core/CMake_project.cmake` to build the core library and any plugins.

## Testing

### Test File Organization
- Core tests are located in `core/tests/` with 5 main test header files:
    - `Test_utilities.h`: Vector types, colors, dates/times, coordinate systems
    - `Test_functions.h`: Global helper functions and math utilities
    - `Test_XML.h`: XML parsing functions
    - `Test_context.h`: Context class methods
    - `Test_data.h`: Context data management (primitive, object, global data)
- Plugin tests are in `plugins/[plugin_name]/tests/selfTest.cpp`
- Tests use the doctest framework with specific patterns (DOCTEST_TEST_CASE, DOCTEST_CHECK, etc.)
- When adding new functions or classes, always add a test for it in the appropriate test file.

### Building and Running Tests
1. **Use the Unified Test System**: Always use `utilities/run_tests.sh` to build and run tests
    ```bash
    cd utilities
    ./run_tests.sh  # Runs all tests
    ```
   The script automatically:
   - Creates temporary build directories
   - Configures CMake with appropriate plugins
   - Builds only the required test targets
   - Runs the tests and cleans up
   - Builds in Release mode by default (use `--debugbuild` for Debug mode)

### Test Workflow

**CRITICAL DEBUGGING REQUIREMENTS:**
When investigating test failures, you MUST:
1. **ALWAYS use `--verbose`** - Without it, you won't see doctest output explaining why tests failed
2. **ALWAYS use `--project-dir <name>`** - Recompiling from scratch wastes minutes per iteration
3. **ALWAYS clean up with `rm -rf <project_dir>`** - Don't leave persistent directories behind

### Running Tests - Quick Reference

1. **First-Time / Exploratory Testing**:
    ```bash
    cd utilities
    ./run_tests.sh                              # Run all tests
    ./run_tests.sh --test photosynthesis        # Run single plugin test
    ./run_tests.sh --tests "radiation,lidar"    # Run multiple plugin tests
    ./run_tests.sh --nogpu                      # Run only non-GPU tests
    ```

2. **Debugging Test Failures (THE RIGHT WAY)**:
    ```bash
    # Initial run - creates persistent project
    ./run_tests.sh --project-dir debug_proj --test energybalance --verbose

    # See what failed? Fix code, then rerun (5-10x faster!)
    ./run_tests.sh --project-dir debug_proj --test energybalance --verbose

    # List specific test cases
    ./run_tests.sh --project-dir debug_proj --doctestargs "--list-test-cases"

    # Run just one failing test case
    ./run_tests.sh --project-dir debug_proj --testcase "MyFailingTest" --verbose

    # CRITICAL: Clean up when done
    rm -rf debug_proj
    ```

3. **Advanced Test Options**:
    ```bash
    ./run_tests.sh --testcase "Test Name"       # Run specific doctest case
    ./run_tests.sh --doctestargs "--help"       # Pass args directly to doctest
    ./run_tests.sh --debugbuild                 # Build in Debug mode
    ./run_tests.sh --nogpu                      # Run only non-GPU tests
    ./run_tests.sh --memcheck                   # Enable memory checking
    ./run_tests.sh --verbose                    # Show full build output
    ./run_tests.sh --log-file test.log          # Log output to file
    ./run_tests.sh --project-dir my_project     # Use persistent project directory
    ```

4. **Custom Tests**:
    - It is sometimes necessary to write custom tests for specific functionality such as testing performance. In this case, add a custom project in `samples/`.
    - Follow the pattern of existing projects in `samples/` to create a new test project.
    - You can also use the `utilities/create_project.sh` script to automate creation of new projects.
    - Be sure to clean up the files after you are done.

5. **Common Build Issues**:
    - Always check compilation errors carefully for missing includes or function signature mismatches
    - Plugin tests may have different data label expectations than actual implementation (verify against source code)
    - Tests failing with "does not exist" errors usually indicate incorrect data labels in tests

6. **Assessing Success/Failure**:
   - **If a test fails, you MUST rerun with `--verbose` to see the actual doctest error messages**
   - Without --verbose, you only see "failed" but not WHY it failed
   - Always check for error/warning messages first before declaring success
   - Look for specific failure indicators like "WARNING", "ERROR", "FAILED"
   - Follow the "Don't be too agreeable" principle - be critical and thorough
   - Stop and analyze failures instead of proceeding when things are clearly broken
   - All tests should be passing before considering the implementation complete. 100% success rate is the only acceptable outcome.

### Test Coverage
- The script `utilities/generate_coverage_report.sh` generates comprehensive coverage reports for specific tests
- **Usage requires specifying which tests to analyze**:
    ```bash
    cd utilities
    ./generate_coverage_report.sh --test context           # Core context tests
    ./generate_coverage_report.sh --test radiation         # Radiation plugin tests
    ./generate_coverage_report.sh --tests "radiation,lidar" # Multiple plugin tests
    ```
- The script can be run from any directory within the Helios repository
- **Coverage projects are persistent** - reports and build artifacts are preserved in descriptively named directories:
    - `coverage_context/` for single tests
    - `coverage_radiation_etc/` for multiple tests
    - Custom directories with `--project-dir my_coverage`
- **Prefer HTML output (default)** over text output - provides color-coded coverage percentages and clickable file navigation in `coverage/index.html`
- Coverage analysis automatically builds and runs tests, then generates reports
- Coverage improvements should focus on:
    - Adding tests for uncovered functions (check function names against test coverage)
    - Verifying data manipulation functions use correct parameter types and labels
    - Testing edge cases and error conditions
    - Ensuring tests are organized with similar functionality and non-redundant

**Coverage Report Options**:
```bash
./generate_coverage_report.sh --test context -r html        # HTML report (default)
./generate_coverage_report.sh --test context -r text        # Text report
./generate_coverage_report.sh --test context --project-dir my_coverage  # Custom directory
```

### Debugging Plugin Tests
- When plugin tests fail, first check if the test expectations match the actual implementation
- Always verify test expectations against the source code in `plugins/[name]/src/`
- Tests should not write any errors messages to std::cerr. Use the struct `capture_cerr` (defined in `core/include/global.h`) to capture any error messages and check them in the test. There is a similar method `capture_cout` if needed.
- **CRITICAL: Doctest Output Scoping** - Always ensure `capture_cout` and `capture_cerr` objects go out of scope BEFORE `DOCTEST_CHECK` assertions:
  ```cpp
  // ❌ WRONG - capture remains in scope during assertions
  capture_cout capture;
  model.run();
  std::string output = capture.get_captured_output();
  DOCTEST_CHECK(output.find("something") != std::string::npos);  // doctest failure output will be captured!

  // ✅ CORRECT - capture destroyed before assertions
  std::string output;
  {
      capture_cout capture;
      model.run();
      output = capture.get_captured_output();
  }  // capture destroyed here
  DOCTEST_CHECK(output.find("something") != std::string::npos);  // doctest failure output prints correctly
  ```
  If a capture object is in scope during assertions, doctest's failure messages will be captured instead of displayed, making debugging impossible.
- Make sure that any functions marked `[[nodiscard]]` assign their return value to a variable in the test, otherwise the compiler will issue a warning.
- **Plugin `selfTest.cpp` files must include `#define DOCTEST_CONFIG_IMPLEMENT` before including `doctest.h`** - without this, doctest will auto-discover ALL tests linked into the binary (including core tests), causing the plugin to run hundreds of unrelated tests instead of just its own.

## Documentation
- When adding new functions or classes, always add a docstring to the header file, and consider whether additional documentation is needed (in `doc/*.dox` for the core or `plugins/[plugin_name]/doc/*.dox` for plugins).
- If changes are made to docstrings or function signatures, build the documentation file `doc/Doxyfile` using doxygen to test it. Some notes on this are given below:
- Run `doxygen doc/Doxyfile` to generate the documentation from the root directory not the `doc` directory.
- Check doxygen output for warnings. It is ok to ignore warnings of the form " warning: Member XXX is not documented."
- The main concern when changing docstrings or function signatures is the potential for breaking \ref references, which produces a warning like:  warning: unable to resolve reference to XXX for \ref command".
  When fixing these, don't just remove the funciton signature or use markdown `` ticks to suppress the warning. It is important to keep the signature in the reference for overloaded functions/methods. Figure out why the function signature is not matching and thus causing Doxygen to treat it as plain text.

**Before Completing an Implementation Task**:
1. Add any tests to the relevant `selfTest.cpp` file based on new code.
2. Review the documentation to ensure it is up-to-date (either in `doc/*.dox` or in `plugins/*/doc/*.dox`).

## MCP: Knowledge-graph memory policy

- Server alias: `memory` (added via `claude mcp add memory npx:@modelcontextprotocol/server-memory`).
- Purpose: persist structured facts and relationships about this repo, projects, and collaborators using the knowledge-graph memory tools.
- Safety: summarize what you plan to store before writing; do not store secrets or API keys.

### **CRITICAL: How MCP Search Actually Works**

MCP memory uses **simple case-insensitive substring matching** - NOT semantic search or AI understanding. It searches for your query string within entity names, types, and observations using basic `toLowerCase().includes()`.

**Key Implications:**
- Query "test crash" searches for the EXACT phrase "test crash" as a substring
- Query "radiation model plugin error" WON'T match unless that exact phrase exists
- NO fuzzy matching, NO synonyms, NO semantic understanding
- Case-insensitive: "Test" finds "test", "TEST", "testing"
- Substring matching: "photo" finds "photosynthesis", "photography"

### Search Strategy (MANDATORY)

**✅ GOOD Query Patterns:**
```python
# Single keyword queries work best
mcp__memory__search_nodes(query="radiation")
mcp__memory__search_nodes(query="photosynthesis")
mcp__memory__search_nodes(query="cmake")

# Short phrases that appear verbatim
mcp__memory__search_nodes(query="energy balance")
mcp__memory__search_nodes(query="test failure")

# Word stems for broader matching
mcp__memory__search_nodes(query="photo")  # finds "photosynthesis", "photorespiration"
mcp__memory__search_nodes(query="optim")  # finds "optimization", "optimizer"
```

**❌ BAD Query Patterns:**
```python
# Complex multi-term queries (searches for EXACT phrase)
mcp__memory__search_nodes(query="radiation model plugin GPU acceleration error fix")

# Natural language questions (no semantic understanding)
mcp__memory__search_nodes(query="why does the photosynthesis model crash?")

# Multiple disconnected concepts (won't match unless exact phrase exists)
mcp__memory__search_nodes(query="CMake build shader compilation OptiX")
```

**Query Refinement Strategy:**
1. **Start broad, then narrow**: "plugin" → "radiation" → "radiation GPU" → "radiation optix"
2. **Use word stems**: "photo" instead of "photosynthesis", "optim" instead of "optimization"
3. **Multiple simple searches** over one complex search: Do 3 searches with ["radiation", "GPU", "crash"] instead of one "radiation plugin GPU crash error"
4. **Search by entity type**: "technical_issue", "solution", "bug_fix"

### When to write memory
Trigger a write when any of the following occur:
1. A new project, module, or dataset is introduced.
2. A design decision or convention is finalized.
3. A collaborator's role, preference, or responsibility is clarified.
4. A technical issue is discovered and solved.
5. A plugin implementation pattern or architectural insight is learned.
6. Build system or test infrastructure changes are made.

### How to write memory

**Entity Naming Standards (MANDATORY):**
- Use underscores for multi-word names: `Radiation_Plugin`, `CMake_OptiX_Fix`, `Photosynthesis_Model`
- Be specific but concise (3-5 words max): `radiation_optix_path_issue_2025_10`
- Include dates for time-sensitive items: `cmake_fix_2025_10_06`
- Use consistent casing: Choose `snake_case` or `CamelCase` and stick with it
- Make names searchable: Include keywords you'd search for

**Atomic Observation Principle:**
One observation = one fact. Break compound statements into separate observations.

✅ **GOOD Observations:**
```python
observations = [
    "Requires OptiX 7.3 or higher for GPU acceleration",
    "CMake variable OPTIX_INCLUDE_DIR must be set",
    "Fixed in commit abc123 on 2025-10-06",
    "Located in plugins/radiation/src/RadiationModel.cpp",
    "Bug affects only Windows builds with CUDA 11.8"
]
```

❌ **BAD Observations:**
```python
observations = [
    "Requires OptiX 7.3 or higher for GPU acceleration and CMake variable OPTIX_INCLUDE_DIR must be set, fixed in commit abc123 affecting only Windows with CUDA 11.8"
]
```

**Recommended Entity Types for Helios:**
- **Components**: `plugin`, `module`, `library`, `test_suite`
- **Knowledge**: `technical_issue`, `solution`, `bug_fix`, `performance_fix`
- **Process**: `build_pattern`, `test_pattern`, `cmake_pattern`
- **Project**: `project`, `feature`, `dataset`, `experiment`

**Creating Entities and Relations:**
```python
# 1. Create entities with atomic observations
mcp__memory__create_entities(entities=[
    {
        "name": "Radiation_OptiX_Path_Issue",
        "entityType": "technical_issue",
        "observations": [
            "OptiX include path not found on Windows builds",
            "CMake looks in hardcoded Linux paths",
            "Affects radiation plugin compilation",
            "Discovered during Windows CI build 2025-10-06"
        ]
    }
])

# 2. Create meaningful relations
mcp__memory__create_relations(relations=[
    {
        "from": "Radiation_OptiX_Path_Fix",
        "to": "Radiation_OptiX_Path_Issue",
        "relationType": "solves"
    }
])

# 3. Update existing entities (use add_observations, NOT append_observations)
mcp__memory__add_observations(observations=[
    {
        "entityName": "Radiation_OptiX_Path_Fix",
        "contents": ["Verified working on Windows CI as of 2025-10-07"]
    }
])
```

### When to read memory

**ALWAYS search before creating** to avoid duplicates:
```python
# Before creating new entity, search for existing
results = mcp__memory__search_nodes(query="radiation")
# If found, use add_observations to update
# If not found, create new entity
```

**Search at session start** to gather context:
```python
# Use 2-3 keyword searches
results1 = mcp__memory__search_nodes(query="radiation")
results2 = mcp__memory__search_nodes(query="optix")
results3 = mcp__memory__search_nodes(query="cmake")
```

**Use open_nodes when you know exact names:**
```python
entities = mcp__memory__open_nodes(names=[
    "Radiation_Plugin_Architecture",
    "CMake_OptiX_Configuration_Pattern"
])
```

### Common Pitfalls and Solutions

**Pitfall 1: Overly specific queries return nothing**
- Problem: `search_nodes(query="radiation plugin fails with OptiX 7.3 on Windows 11")`
- Solution: Break into separate searches: `search_nodes(query="radiation")`, `search_nodes(query="optix")`

**Pitfall 2: Duplicate entities**
- Problem: Creating `Radiation_Plugin`, `RadiationModel`, `radiation_model` separately
- Solution: ALWAYS search before creating: `search_nodes(query="radiation")`

**Pitfall 3: Generic entity names**
- Problem: Entity named "bug_fix" is impossible to find among many bug fixes
- Solution: Include distinctive identifiers: `radiation_optix_path_fix_2025_10`

**Pitfall 4: Compound observations**
- Problem: "Fixed OptiX path and updated CMake and added tests"
- Solution: Break into atomic facts: ["Fixed OptiX include path detection", "Updated CMake FindOptiX module", "Added Windows CI test"]

### Helios-Specific Memory Structure

**Core Categories:**
```python
# Plugins (main focus)
"Radiation_Plugin", "Photosynthesis_Plugin", "EnergyBalance_Plugin", "Visualizer_Plugin"

# Technical challenges
"cmake_optix_detection", "test_coverage_gap", "build_system_pattern"

# Solutions and patterns
"unified_test_system", "cmake_plugin_pattern", "fail_fast_error_handling"

# Architecture
"plugin_architecture", "context_primitive_system", "xml_parsing_framework"
```

**Relation Types to Use:**
- `solves`, `fixes` (solution → problem)
- `depends_on`, `requires` (component → dependency)
- `implements`, `provides` (implementation → interface)
- `part_of`, `contains` (child → parent)
- `tested_by`, `validates` (code → test)

### Quick Reference

**Search Commands:**
```python
mcp__memory__search_nodes(query="keyword")           # Discovery search
mcp__memory__open_nodes(names=["Entity_Name"])       # Specific retrieval
mcp__memory__read_graph()                            # Export full graph
```

**When Search Returns Nothing:**
1. Try shorter query (use word stem: "photo" not "photosynthesis")
2. Try related keywords ("GPU" if "OptiX" fails)
3. Search entity types: `search_nodes(query="technical_issue")`
4. Use `read_graph()` to see all entities 