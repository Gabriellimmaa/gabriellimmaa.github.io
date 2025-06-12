---
title: Porting Arkanjo to macOS
categories: [Open Source Software Development, Arkanjo, Contributions, MAC0470]
tags: [linux, refactoring, patch, open-source, arkanjo, macos]
render_with_liquid: false
---
Recently I contributed to the [Arkanjo project](https://github.com/LipArcanjo/arkanjo), a tool for detecting code clones. It was originally configured to run only on Linux (Ubuntu/Debian), and I took the initiative to adapt it to macOS. In this post, I’ll walk through the main issues I faced and how I solved them.

---

## Prerequisites on macOS

These are the dependencies required to run Arkanjo on macOS. All can be installed using [Homebrew](https://brew.sh/):

```bash
brew install python
pip3 install nltk gensim astor
python3 -m nltk.downloader punkt
brew install jsoncpp
```

---

## Main Issues and Fixes

### 1. `#include <bits/stdc++.h>`

This header is available only on Linux-based compilers. On macOS, I had to manually replace it with the specific headers used in each file — a tedious but necessary process.

---

### 2. `#include <jsoncpp/json/json.h>`

After installing `jsoncpp` via Homebrew, the include path on macOS became `<json/json.h>`. I automated this replacement using the command:

```bash
find . -name "*.hpp" -o -name "*.cpp" | xargs sed -i '' 's|<jsoncpp/json/json.h>|<json/json.h>|g'
```

Additionally, I updated the `Makefile` to correctly locate the installed headers and link the library:

```make
CXXFLAGS = -std=c++17 -g -I/opt/homebrew/include
LDFLAGS  = -L/opt/homebrew/lib -ljsoncpp
```

---

### 3. Shell commands (`rm`) in C++ code

The original code used system calls like:

```cpp
string command_rm_tmp = "rm -r -f " + base_path + "/";
system(command_rm_tmp.c_str());
```

I replaced this with a more secure and portable approach using C++17's `<filesystem>`:

```cpp
#include <filesystem>
namespace fs = std::filesystem;

fs::path dir_to_remove = base_path;
if (fs::exists(dir_to_remove)) {
    fs::remove_all(dir_to_remove);
}
```

---

## Updated Makefile

Here is the final version of the `Makefile` that works correctly on macOS:

```make
CXX = g++
CXXFLAGS = -std=c++17 -g -I/opt/homebrew/include
LDFLAGS = -L/opt/homebrew/lib -ljsoncpp

SRC_BASE = base/utils.cpp base/config.cpp base/path.cpp base/similarity_table.cpp base/function.cpp
SRC_PRE = pre/function_breaker_util.cpp pre/function_breaker_c.cpp pre/function_breaker_java.cpp pre/function_breaker.cpp pre/parser.cpp pre/duplication_finder_tool.cpp pre/duplication_finder_diff.cpp pre/preprocessor.cpp
SRC_E2E = tests/e2e/test.cpp
SRC_EXTRA = finder/similar_function_finder.cpp counter/counter_duplication_code_trie.cpp counter/counter_duplication_code.cpp explorer/similarity_explorer.cpp big_clone/big_clone_formater.cpp big_clone/big_clone_tailor_evaluator.cpp rand/random_selector.cpp orchestrator.cpp

all: exec preprocessor test

exec:
	$(CXX) $(CXXFLAGS) $(SRC_BASE) $(SRC_PRE) $(SRC_EXTRA) -o exec $(LDFLAGS)

preprocessor:
	$(CXX) $(CXXFLAGS) $(SRC_BASE) $(SRC_PRE) pre/preprocessor_main.cpp -o preprocessor $(LDFLAGS)

test:
	$(CXX) $(CXXFLAGS) $(SRC_BASE) $(SRC_PRE) $(SRC_E2E) -o test $(LDFLAGS)

clean:
	rm -f exec preprocessor test
```

---

## Frequently Used Commands

These were the key commands I ran during the build process:

```bash
make clean
make
./preprocessor
```

---

## Issue Resolved

This contribution was made to solve the following open issue:

🔗 [Arkanjo Issue #4 - Adapt build for macOS](https://github.com/LipArcanjo/arkanjo/issues/4)

---

## Conclusion

Porting a C++ project designed for Linux to macOS requires special attention to header compatibility, library paths, and shell command usage. It was a great learning experience and a valuable contribution to make the project cross-platform.

If you’re interested in contributing too, check out the [open issues](https://github.com/LipArcanjo/arkanjo/issues) on the Arkanjo repository. Every bit of help counts!

Feel free to reach out or fork the project on GitHub! 👨🏻‍💻
