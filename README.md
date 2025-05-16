# ai-coding-conventions

A set of coding conventions for a variety of projects, designed to inform and improve AI-assisted coding workflows.

This repository provides opinionated, practical guidelines for several popular frameworks and languages. These conventions are especially useful when working with AI coding assistants such as [aider](https://aider.chat/docs/usage/conventions.html), but can also be adapted for use in other editors and IDEs.

## Contents

- `laravel-conventions.md`: Coding conventions for Laravel (PHP)
- `python-conventions.md`: Coding conventions for Python projects
- `wordpress-conventions.md`: Coding conventions for WordPress (PHP)

Feel free to fork this repository and adjust the guidelines to fit your team's needs.

---

## How to Use with Aider

1. **Add conventions to your project**:  
   Copy the relevant conventions file(s) into your project repository.

2. **Reference conventions in your aider session**:  
   When starting an aider session, mention the conventions file(s) in your prompt or add them to the chat.  
   Example:
   ```
   Please follow the guidelines in python-conventions.md for all code changes.
   ```

3. **Keep conventions up to date**:  
   Update the conventions files as your team's practices evolve. Aider will use the latest version present in your repo.

For more information, see the [aider documentation](https://aider.chat/docs/usage/conventions.html).

---

## How to Use with Other IDEs (e.g., Cursor, VS Code, etc.)

- **Manual Reference**:  
  Open the relevant conventions file in a split window or tab for easy reference while coding.

- **AI Integration**:  
  If your IDE supports AI coding assistants (such as GitHub Copilot, Cursor, or others), you can:
  - Paste relevant sections of the conventions into the AI's context window or prompt.
  - Instruct the AI to follow the conventions by referencing the file directly in your prompt.

- **Team Onboarding**:  
  Share these conventions with your team to ensure consistent code style and best practices across your project.

---

## Contributing

Contributions and suggestions are welcome! Please open an issue or submit a pull request to propose changes or additions.

---

## License

This repository is provided under the MIT License. See [LICENSE](LICENSE) for details.

