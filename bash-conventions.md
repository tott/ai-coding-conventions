# Bash/ZSH Scripting Conventions

This document outlines conventions and best practices for Bash/ZSH scripting in our projects. Following these guidelines ensures consistency, readability, maintainability, and security across shell scripts while maximizing the effectiveness of AI-assisted coding. Scripts following these conventions will work reliably across Linux, macOS, and Windows Subsystem for Linux (WSL) environments.

## Shell Selection

### When to use Bash vs ZSH

- **Bash**: Preferred for scripts that need high portability across environments
    - Strikes a good balance between portability and developer experience[^16]
    - Use the shebang `#!/usr/bin/env bash` for better portability[^16]
- **ZSH**: Best for interactive shells and scripts leveraging ZSH's advanced features
    - Offers powerful tab completion, clever history, and pattern matching[^15]
    - Use the shebang `#!/usr/bin/env zsh`
    - Reset options at beginning with `emulate -LR zsh` to ensure predictable behavior[^12]


## File Structure and Organization

### Directory Structure

```
project/
├── bin/        # Executable scripts for general use
├── lib/        # Library functions and modules
├── conf/       # Configuration files
├── test/       # Test scripts
├── modules/    # Dynamically loaded modules
├── data/       # Data files used by scripts
└── tmp/        # Temporary files (ensure proper cleanup)
```


### File Naming Conventions

- Use `.sh` extension for Bash scripts and `.zsh` for ZSH scripts[^17]
- Use descriptive names with lowercase and underscores (e.g., `process_data.sh`)[^17]
- Avoid spaces in filenames


### Module Organization

- Split code into logical modules (e.g., `network_utils.sh`, `string_manipulation.sh`)[^17]
- Use `source` or `.` to include modules in scripts
- Prefer `source` in ZSH and `.` in Bash for maximum portability


## Script Structure

### Standard Template

```bash
#!/usr/bin/env bash
# or #!/usr/bin/env zsh

# Set strict mode
set -o errexit   # Exit on error
set -o pipefail  # Exit if any command in a pipeline fails
set -o nounset   # Error on unset variables

# Debug mode (enabled with TRACE=1)
if [[ "${TRACE-0}" == "1" ]]; then
  set -o xtrace
fi

# Script description, usage, and version
VERSION="1.0.0"
SCRIPT_NAME="$(basename "$0")"

usage() {
  cat <<EOF
Usage: $SCRIPT_NAME [options] <arguments>

Description:
  Brief description of what the script does.

Options:
  -h, --help     Show this help message and exit
  -v, --version  Show version information and exit
EOF
}

# Process command-line arguments
if [[ $# -eq 0 ]]; then
  usage
  exit 0
fi

while [[ $# -gt 0 ]]; do
  case "$1" in
    -h|--help)
      usage
      exit 0
      ;;
    -v|--version)
      echo "$SCRIPT_NAME v$VERSION"
      exit 0
      ;;
    # Add more options here
    *)
      # Handle positional arguments
      break
      ;;
  esac
  shift
done

# Main function
main() {
  # Script logic here
  echo "Script running..."
}

# Call main function
main "$@"
```


### ZSH Specific Template Additions

For ZSH scripts, include after the shebang:

```zsh
#!/usr/bin/env zsh

# Reset ZSH options to ensure predictable behavior
emulate -LR zsh
```


## Coding Style and Formatting

### General Guidelines

- Indent with 2 spaces (not tabs)[^2]
- Aim for 80 characters per line, with hard limit of 100[^2]
- Use whitespace to separate commands from arguments[^2]
- Add blank lines to separate logical sections


### Variable Naming and Usage

- Use lowercase for local variables with underscores to separate words[^3]
- Use UPPERCASE for environment variables and constants[^3]
- Always quote variable expansions: `"${variable}"`[^2][^16]
- For variables that might be unset, use `${variable:-default}`[^16]


### Control Structures

```bash
# If statement - put "then" on same line as "if"
if [[ -d "${dir}" ]]; then
  echo "Directory exists"
else
  mkdir -p "${dir}"
fi

# For loop - put "do" on same line as "for"
for file in "${files[@]}"; do
  process_file "${file}"
done

# While loop
while read -r line; do
  echo "$line"
done < "${input_file}"
```


### Functions

```bash
# Define with parentheses and space before brace
function process_file() {
  local file="$1"  # Use local variables in functions
  
  if [[ -f "${file}" ]]; then
    # Process file
    return 0
  else
    echo "Error: File not found" >&2
    return 1
  fi
}
```


### Command Substitution and Expansion

- Use `$(command)` for command substitution, not backticks[^3]
- Always quote command substitution: `"$(command)"`
- Prefer parameter expansion over external tools[^10]:

```bash
# Instead of:
addition="$(expr "${X}" + "${Y}")"
substitution="$(echo "${string}" | sed -e 's/^foo/bar/')"

# Prefer:
addition="$(( X + Y ))"
substitution="${string/#foo/bar}"
```


## Error Handling and Debugging

### Safety Options

Always include these safety options at the beginning of scripts[^18][^16]:

```bash
set -o errexit   # Exit on error
set -o pipefail  # Exit if any command in a pipeline fails
set -o nounset   # Error on unset variables
```


### Defensive Programming

- Validate input parameters at the beginning
- Use `[[ ]]` for conditions instead of `[ ]` or `test`[^16]
- Always check for command existence before use:

```bash
if ! command -v required_command >/dev/null 2>&1; then
  echo "Error: required_command is not installed" >&2
  exit 1
fi
```


### Error Messages

- Direct error messages to stderr:

```bash
echo "Error: Invalid input parameter" >&2
```

- Use descriptive error messages that suggest resolution


### Debugging

- Enable debug mode with environment variable:

```bash
if [[ "${TRACE-0}" == "1" ]]; then
  set -o xtrace  # Print commands before execution
fi
```

- Run with: `TRACE=1 ./script.sh`
- Use ShellCheck to find and fix warnings[^3]
- Check syntax errors: `bash -n script.sh` or `zsh -n script.zsh`[^3]


## Security Best Practices

- Quote all variable expansions to prevent word splitting and globbing[^2][^16]
- Create temporary files securely using `mktemp`[^3]:

```bash
temp_file=$(mktemp)
trap 'rm -f "${temp_file}"' EXIT
```

- Validate and sanitize all external inputs
- Use restricted permissions: `chmod 755` for scripts
- Avoid storing sensitive data like passwords in scripts
- Never use `eval` unless absolutely necessary


## Cross-Platform Compatibility

### Linux, macOS, and WSL Compatibility

- Use POSIX-compliant features when possible
- Be aware of GNU vs BSD command differences (especially on macOS)
- For platform-specific code, use detection:

```bash
case "$(uname)" in
  Linux*)
    # Linux-specific commands
    ;;
  Darwin*)
    # macOS-specific commands
    ;;
  MINGW*|CYGWIN*|MSYS*)
    # Windows-specific commands
    ;;
  *)
    echo "Unsupported platform: $(uname)" >&2
    exit 1
    ;;
esac
```

- Use portable commands or provide alternatives:

```bash
# Instead of GNU-specific: 
# readlink -f "$file"

# Use this portable function:
realpath() {
  [[ $1 = /* ]] && echo "$1" || echo "$PWD/${1#./}"
}
```


## AI-Assisted Coding Best Practices

### Effective Prompting

- Be specific, descriptive, and detailed in prompts to AI[^7]
- Include the target shell (Bash or ZSH) in your prompt
- Specify environment constraints (Linux, macOS, WSL)
- Provide examples of desired coding style[^7]


### Task Selection

- Choose appropriate tasks for AI assistance[^8]:
    - Well-defined, self-contained functions
    - Standard utility functions
    - Command-line argument parsing
- Maintain human oversight for:
    - Security-critical sections
    - Complex architectural decisions
    - Platform-specific optimizations


### Implementing Guardrails

- Apply Test-Driven Development with AI-generated code[^8]:

1. Ask AI to write tests first
2. Ask AI to implement features that pass tests
3. Iteratively run tests to verify functionality
- Ensure all AI-generated code passes ShellCheck validation[^8]
- Implement type checking where possible using comments and validation


### Code Review

- Thoroughly review all AI-generated code[^8]
- Understand all code before committing
- Verify edge cases and error handling
- Check for potential security vulnerabilities


## Testing and Validation

### Test-Driven Development

- Write tests before implementing features[^8]
- Use a testing framework like Bats (Bash Automated Testing System)
- Keep tests in a separate `test/` directory
- Test both normal operation and edge cases


### Example Test Structure

```bash
#!/usr/bin/env bats

setup() {
  # Setup test environment
  source "../lib/functions.sh"
}

teardown() {
  # Clean up after tests
}

@test "function_name handles valid input" {
  run function_name "valid_input"
  [ "$status" -eq 0 ]
  [ "$output" = "expected output" ]
}

@test "function_name handles invalid input" {
  run function_name "invalid_input"
  [ "$status" -eq 1 ]
}
```


## Differences Between Bash and ZSH

### Key Behavioral Differences

1. **Variable Substitution**: In ZSH, unquoted variables are NOT split by default[^12]

```bash
# In Bash:
words="one two three"
echo $words  # Outputs: one two three (split into arguments)

# In ZSH:
words="one two three"
echo $words  # Outputs: one two three (as a single argument)
```

2. **Array Indexing**: ZSH arrays are 1-indexed by default, Bash arrays are 0-indexed
3. **Globbing Behavior**: ZSH has more powerful globbing features

```zsh
# ZSH only:
ls *.*(.)    # List only regular files
ls **/*(.)   # Recursively list regular files
```

4. **For ZSH Compatibility Mode**:

```zsh
# Make ZSH behave more like Bash when needed
emulate -LR bash
```


## Conclusion

These conventions provide a foundation for creating maintainable, secure, and cross-platform shell scripts. When using AI assistance for shell scripting, always ensure the AI follows these conventions and review all generated code thoroughly. Remember that while AI can help with implementation details, human oversight remains essential for ensuring that scripts meet security, performance, and reliability requirements.

## References

- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [Shell Script Best Practices](https://sharats.me/posts/shell-script-best-practices/)
- [ShellCheck](https://www.shellcheck.net/)
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/)
- [ZSH Documentation](https://zsh.sourceforge.io/Doc/)

[^1]: https://bertvv.github.io/cheat-sheets/Bash.html

[^2]: https://styles.goatbytes.io/lang/shell/

[^3]: https://github.com/SixArm/unix-shell-script-tactics

[^4]: https://www.chromium.org/chromium-os/developer-library/reference/style-guides/shell/

[^5]: https://www.reddit.com/r/zsh/comments/m4khgf/best_tutorial_for_learning_zsh_scripting_in_2021/

[^6]: https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/

[^7]: https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api

[^8]: https://engineering.axur.com/2025/05/09/best-practices-for-ai-assisted-coding.html

[^9]: https://www.gnu.org/software/autoconf/manual/autoconf-2.61/html_node/Portable-Shell.html

[^10]: https://google.github.io/styleguide/shellguide.html

[^11]: https://dev.to/unfor19/writing-bash-scripts-like-a-pro-part-1-styling-guide-4bin

[^12]: https://scriptingosx.com/2019/08/moving-to-zsh-part-8-scripting-zsh/

[^13]: https://thevaluable.dev/zsh-completion-guide-examples/

[^14]: https://www.reddit.com/r/bash/comments/17fj8x0/my_tips_and_tricks_for_bash_scripting_after/

[^15]: https://www.sitepoint.com/zsh-tips-tricks/

[^16]: https://sharats.me/posts/shell-script-best-practices/

[^17]: https://www.projectrules.ai/rules/zsh

[^18]: https://sap1ens.com/blog/2017/07/01/bash-scripting-best-practices/

[^19]: https://hcsonline.com/images/PDFs/Scripting_Intro_Zsh.pdf

[^20]: https://google.github.io/styleguide/shellguide.html

[^21]: https://github.com/bahamas10/bash-style-guide

[^22]: https://apple.stackexchange.com/questions/396316/best-practice-for-user-shell-scripts-location-and-or-path

[^23]: https://zsh.sourceforge.io/Guide/zshguide01.html

[^24]: https://foojay.io/today/code-reviews-with-ai-a-developer-guide/

[^25]: https://www.pluralsight.com/resources/blog/software-development/prompt-engineering-for-developers

[^26]: https://belief-driven-design.com/grabbing-screen-text-with-a-shell-script-f6bdd/

[^27]: https://unix.stackexchange.com/questions/82244/bash-in-linux-v-s-mac-os

[^28]: https://bookdown.org/yihui/rmarkdown-cookbook/eng-bash.html

[^29]: https://unix.stackexchange.com/questions/42847/are-there-naming-conventions-for-variables-in-shell-scripts

[^30]: https://swimm.io/learn/ai-tools-for-developers/ai-code-review-how-it-works-and-3-tools-you-should-know

[^31]: https://www.ibm.com/think/insights/ai-code-review

[^32]: https://graphite.dev/guides/ai-code-review-implementation-best-practices

[^33]: https://github.com/resources/articles/ai/ai-code-reviews

[^34]: https://www.sonarsource.com/blog/enhancing-team-code-reviews-with-ai-generated-code/

[^35]: https://code.visualstudio.com/docs/copilot/copilot-tips-and-tricks

[^36]: https://unix.stackexchange.com/questions/555099/what-is-the-most-portable-shell-and-relevant-best-practices-to-follow

[^37]: https://stackoverflow.com/questions/19428418/what-is-the-use-of-portable-shell-scripts

[^38]: https://www.usenix.org/system/files/login/articles/login_spring16_09_tomei.pdf

[^39]: https://people.redhat.com/~thuth/blog/general/2021/04/27/portable-shell.html

[^40]: https://www.youtube.com/watch?v=j40datoFrNw

[^41]: https://serverfault.com/questions/865874/bin-sh-vs-bin-bash-for-maximum-portability

[^42]: https://www.reddit.com/r/vscode/comments/1c6hazp/wsl_on_mac/

[^43]: https://stackoverflow.com/questions/20303826/how-to-highlight-bash-shell-commands-in-markdown

[^44]: https://youtrack.jetbrains.com/issue/IJPL-99008/Markdown-Shell-script-highlighting-for-bash-or-sh

[^45]: https://github.com/tcr/markdown-corpus/blob/master/unixorn_awesome-zsh-plugins_README.md

[^46]: https://www.reddit.com/r/linuxquestions/comments/p50jvl/those_of_you_who_prefer_zsh_to_bash_why/

[^47]: https://styles.goatbytes.io/lang/shell/

[^48]: https://gist.github.com/pjeby/c137ace4d91e61e8f1f80e92d84e8b70

[^49]: https://www.linkedin.com/pulse/bash-scripting-conventions-engin-polat

