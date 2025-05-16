# AI Coding Conventions

A comprehensive set of coding conventions for a variety of popular frameworks and languages, designed to inform and improve AI-assisted coding workflows.

These conventions are especially useful when working with AI coding assistants such as [aider](https://aider.chat/docs/usage/conventions.html), [GitHub Copilot](https://github.com/features/copilot), [Cursor](https://www.cursor.so/), or similar tools. They can also be adapted for use in any editor or IDE to help teams maintain high code quality and consistency.

---

## Table of Contents

- [Overview](#overview)
- [Supported Conventions](#supported-conventions)
- [How to Use with Aider](#how-to-use-with-aider)
- [How to Use with Other IDEs](#how-to-use-with-other-ides)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This repository provides opinionated, practical guidelines for several popular frameworks and languages. Each conventions file is self-contained and can be used independently or in combination with others, depending on your project's needs.

The conventions cover topics such as:

- Project structure and organization
- Coding style and formatting
- Naming conventions
- Error handling and security
- Testing and documentation
- Framework- and language-specific best practices
- Guidelines for effective AI-assisted development

These guidelines are intended to help both human and AI contributors write code that is readable, maintainable, and consistent across your codebase.

---

## Supported Conventions

The following conventions are included in this repository:

- [`laravel-conventions.md`](laravel-conventions.md): Coding conventions for Laravel (PHP), including TALL stack and FilamentPHP best practices.
- [`python-conventions.md`](python-conventions.md): Coding conventions for Python projects, with a focus on FastAPI, type hints, HTTPX, and scalable project structure.
- [`wordpress-conventions.md`](wordpress-conventions.md): Coding conventions for WordPress (PHP), synthesizing best practices from WordPress Core, VIP, and leading agencies.
- [`nodejs-conventions.md`](nodejs-conventions.md): Coding conventions for Node.js applications, including project structure, JavaScript style, error handling, and AI-assisted workflows.
- [`bash-conventions.md`](bash-conventions.md): Conventions for Bash and ZSH scripting, covering structure, safety, cross-platform compatibility, and AI-assisted scripting.

Feel free to fork this repository and adjust the guidelines to fit your team's needs. Contributions and suggestions are welcome!

---

## How to Use with Aider

Aider is an AI pair programming tool that can use these conventions to guide its code generation and editing.

**To use these conventions with aider:**

1. **Add conventions to your project**  
   Copy the relevant conventions file(s) into your project repository.

2. **Reference conventions in your aider session**  
   When starting an aider session, mention the conventions file(s) in your prompt or add them to the chat.  
   For example:
   ```
   Please follow the guidelines in python-conventions.md for all code changes.
   ```

3. **Load conventions as read-only**  
   For best results, load the conventions file with `/read python-conventions.md` or by running:
   ```
   aider --read python-conventions.md myfile.py
   ```
   This marks the conventions as read-only and ensures aider always references them.

4. **Keep conventions up to date**  
   Update the conventions files as your team's practices evolve. Aider will use the latest version present in your repo.

For more information, see the [aider documentation on conventions](https://aider.chat/docs/usage/conventions.html).

---

## How to Use with Other IDEs

You can also use these conventions with other editors and AI coding assistants, such as Cursor, VS Code, or GitHub Copilot.

- **Manual Reference**  
  Open the relevant conventions file in a split window or tab for easy reference while coding.

- **AI Integration**  
  If your IDE supports AI coding assistants, you can:
  - Paste relevant sections of the conventions into the AI's context window or prompt.
  - Instruct the AI to follow the conventions by referencing the file directly in your prompt.

- **Team Onboarding**  
  Share these conventions with your team to ensure consistent code style and best practices across your project.

- **Continuous Improvement**  
  Encourage your team to update and refine the conventions as your project evolves.

---

## Contributing

Contributions and suggestions are welcome! Please open an issue or submit a pull request to propose changes, additions, or improvements to the conventions.

When contributing, please:

- Follow the style and structure of the existing conventions files.
- Provide clear explanations and references for any new guidelines.
- Ensure that your changes are relevant and broadly applicable.

---

## License

This repository is provided under the MIT License. See [LICENSE](LICENSE) for details.

