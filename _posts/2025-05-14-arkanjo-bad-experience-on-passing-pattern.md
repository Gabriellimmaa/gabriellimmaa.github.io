# 🔍 Refactoring Similarity Filtering in Arkanjo: Advanced Support for Path and Function Name Patterns

As part of my recent contributions to the [Arkanjo project](https://github.com/LipArcanjo/arkanjo), I implemented an important refactor in the function similarity exploration system. The main goal was to enable more precise filtering based on two distinct criteria:

- **Pattern in the file path**
- **Pattern in the function name**

This change affected both the `Similarity_Explorer` class and the entry point of the system, the `Orchestrator`.

---

## ✨ Improvements in `Similarity_Explorer`

The `match_pattern` method was redesigned to support two separate patterns:

```cpp
bool Similarity_Explorer::match_pattern(Path path1, Path path2) {
    bool match_path1 = true, match_path2 = true;
    bool match_function1 = true, match_function2 = true;

    if (!pattern_to_match_path.empty()) {
        match_path1 = path1.contains_given_pattern(pattern_to_match_path);
        match_path2 = path2.contains_given_pattern(pattern_to_match_path);
    }

    if (!pattern_to_match_function.empty()) {
        match_function1 = path1.build_function_name().find(pattern_to_match_function) != string::npos;
        match_function2 = path2.build_function_name().find(pattern_to_match_function) != string::npos;
    }

    if (both_path_need_to_match_pattern) {
        return (match_path1 && match_function1) && (match_path2 && match_function2);
    }
    return (match_path1 && match_function1) || (match_path2 && match_function2);
}
```

This logic allows the user to define much more specific filters, combining file path and function name criteria when exploring similarity.

---

## 🧩 Integration with the `Orchestrator`

To make this feature accessible via the command line, the `Orchestrator` was updated. It now supports the `-fn` flag to filter by function name:

```cpp
void exploration_command(vector<string> parameters, Similarity_Table *similarity_table){
    string pattern_path = "";
    string pattern_function = "";
    int limiter = 0;
    bool both_need_to_match = false;
    bool sorted_by_number_of_duplicated_code = false;

    for(int i = 0; i < number_parameters-1; i++){
        string param = parameters[i];
        string next_param = parameters[i+1];
        if(param == "-l") limiter = stoi(next_param);
        if(param == "-p") pattern_path = next_param;
        if(param == "-fn") pattern_function = next_param;
        if(param == "-b") both_need_to_match = (next_param == "T");
        if(param == "-c") sorted_by_number_of_duplicated_code = (next_param == "T");
    }

    Similarity_Explorer similarity_explorer(
        similarity_table,
        limiter,
        pattern_path,
        pattern_function,
        both_need_to_match,
        sorted_by_number_of_duplicated_code
    );
}
```

### 💡 Example usage in terminal:

```bash
./exec ex -c T false -fn sorted_by_distance_to_median -p tests/test_multiple_file_big_functions_against_small
```

## 🧪 Results

With these changes, Arkanjo users now have much more control when analyzing duplicated code. This is particularly useful in large codebases, or when trying to detect repeated logic across similarly named utility functions.
