# Node.js Application Conventions and Best Practices

This document outlines standardized conventions for Node.js application development, focusing on project structure, coding standards, and best practices for AI-assisted development. Following these guidelines will help maintain consistency, readability, and maintainability across projects.

## Table of Contents

- [Project Structure](#project-structure)
- [JavaScript Coding Conventions](#javascript-coding-conventions)
- [Error Handling](#error-handling)
- [Node.js Specific Practices](#nodejs-specific-practices)
- [Tailwind CSS Conventions](#tailwind-css-conventions)
- [AI-Assisted Development Guidelines](#ai-assisted-development-guidelines)
- [Documentation Standards](#documentation-standards)


## Project Structure

A consistent project structure helps maintain organization and improves developer onboarding. Use the following structure as a starting point:

```
my-node-app/
├── src/
│   ├── controllers/    # Request handlers
│   ├── models/         # Data structures
│   ├── routes/         # API endpoints
│   ├── services/       # Business logic
│   ├── middleware/     # Express middleware
│   ├── utils/          # Helper functions
│   ├── config/         # Configuration files
│   └── index.js        # Application entry point
├── tests/              # Test files
├── public/             # Static assets
├── .env                # Environment variables
├── .gitignore          # Git ignore file
├── package.json        # Dependencies and scripts
└── README.md           # Project documentation
```


### Component Organization

Every component should have a clear structure:

- Every component should have an `index.js` file that exports the module's functionality[^1]
- Group related functionality (controllers, models, routes) by feature or domain[^15]
- Use consistent naming conventions across all files[^15]


## JavaScript Coding Conventions

Follow these JavaScript conventions for consistency and readability:

### Variables and References

- Use `const` for all references that are not reassigned[^11][^20]
- Use `let` only for references that will be reassigned[^11][^20]
- Never use `var`[^11][^20]
- Declare one variable per declaration statement[^5]

```js
// Good
const user = getUser();
let count = 0;

// Bad
var user = getUser(),
    count = 0;
```


### Objects and Arrays

- Use the literal syntax for object and array creation[^11]

```js
// Good
const items = [];
const user = {};

// Bad
const items = new Array();
const user = new Object();
```

- Use property value shorthand when possible[^11]

```js
// Good
const name = 'John';
const user = { name };

// Bad
const user = { name: name };
```

- Use array methods like `map`, `filter`, and `reduce` instead of loops when appropriate[^3]


### Functions

- Use named function expressions instead of function declarations[^4]
- Prefer arrow functions for anonymous functions[^4]
- Keep functions small and focused on a single task[^5]
- Return early from functions to avoid deep nesting[^5]

```js
// Good
function validateUser(user) {
    if (!user) {
        return false;
    }
    
    if (!user.name) {
        return false;
    }
    
    return true;
}

// Bad
function validateUser(user) {
    let isValid = false;
    if (user) {
        if (user.name) {
            isValid = true;
        }
    }
    return isValid;
}
```


### Asynchronous Code

- Use async/await for asynchronous operations instead of callbacks[^6]
- Always properly handle promises with try/catch blocks[^6]

```js
// Good
async function fetchUser(id) {
    try {
        const user = await getUserById(id);
        return user;
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}

// Bad
function fetchUser(id) {
    return getUserById(id)
        .then(user => user)
        .catch(error => {
            console.error('Failed to fetch user:', error);
            throw error;
        });
}
```


## Error Handling

Proper error handling is critical for robust applications:

- Create custom error classes for different error types[^6]
- Use middleware for centralized error handling in Express applications[^6]
- Log errors with appropriate severity levels[^6]
- Include contextual information in error messages[^6]

```js
app.get('/user/:id', async (req, res, next) => {
    try {
        const user = await getUserById(req.params.id);
        if (!user) {
            return res.status(404).send('User not found');
        }
        res.send(user);
    } catch (error) {
        next(error); // Pass to error handling middleware
    }
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something went wrong!');
});
```


## Node.js Specific Practices

### Environment Variables

- Store configuration in environment variables, not in code[^6]
- Use `.env` files for local development, but never commit them[^6]
- Load environment variables using `dotenv` package[^6]

```js
// config.js
require('dotenv').config();

module.exports = {
    dbHost: process.env.DB_HOST,
    dbUser: process.env.DB_USER,
    dbPass: process.env.DB_PASS,
    port: process.env.PORT || 3000
};
```


### Module Exports

- Export a single responsibility per file[^1]
- Use named exports for multiple functions/classes[^1]
- Organize related functionality in a directory with an `index.js` file[^1]


## Tailwind CSS Conventions

When generating HTML with Tailwind CSS, follow these practices:

### Class Order

- Maintain a consistent order for utility classes following the "Concentric CSS" approach[^9]:

1. Positioning/visibility
2. Box model
3. Borders
4. Backgrounds
5. Typography
6. Other visual adjustments


### Class Usage

- Use fewer utility classes when possible (use `mx-2` instead of `ml-2 mr-2`)[^9]
- Prefix responsive utility classes with their breakpoint (e.g., `lg:flex`)[^9]
- Group related utilities for readability[^9]

```html
<!-- Good -->
<div class="flex flex-col items-center justify-center p-4 lg:p-8 text-center">
  <!-- Content -->
</div>

<!-- Bad (mixed order, verbose) -->
<div class="text-center p-4 flex items-center justify-center flex-col lg:p-8">
  <!-- Content -->
</div>
```


## AI-Assisted Development Guidelines

Leverage AI tools effectively for Node.js development:

### Task Selection

- Assign well-defined, specific tasks to AI assistants[^8]
- Reserve architecture and complex design decisions for human developers[^8]
- Use AI for repetitive coding tasks, boilerplate code, and routine refactoring[^8]


### Prompting Strategies

1. **Clear Task Definitions**: Provide explicit context and examples when prompting AI[^8]
```
# Good Prompt
"Create a controller function that handles user registration with the following requirements:
- Validate email and password inputs
- Hash password using bcrypt
- Return appropriate error responses for validation failures
- Return JWT token on success
Here's our User model structure: [model details]"

# Poor Prompt
"Write a user registration function"
```

2. **Implement Guardrails**: Use unit tests, interfaces, and type definitions to keep AI-generated code on track[^8]
3. **Test-Driven Development**: Have AI write tests first, then implement features[^8]
    - Ask AI to write failing tests
    - Implement features to make tests pass
    - Iteratively run tests to confirm functionality
4. **Incremental Complexity**: Break complex tasks into smaller chunks[^8]
    - Build functionality incrementally rather than all at once
    - Focus on one abstraction layer at a time

### Code Review and Quality

- Always review AI-generated code for:
    - Security vulnerabilities
    - Performance issues
    - Adherence to project conventions
    - Edge cases and error handling


## Documentation Standards

### Code Comments

- Use JSDoc comments for functions and classes[^3]
- Include examples in documentation when helpful[^3]
- Only comment complex logic, not obvious operations[^3]

```js
/**
 * Authenticates a user and generates a JWT token
 * @param {string} email - User email address
 * @param {string} password - User password
 * @returns {Promise<string>} JWT token
 * @throws {AuthError} If authentication fails
 */
async function authenticateUser(email, password) {
    // Implementation
}
```


### Markdown Documentation

- Use GitHub-Flavored Markdown for all documentation[^12]
- Structure documentation with clear headings and sections[^12]
- Use code fences with language identifiers for syntax highlighting[^12]

```markdown
## Authentication API

### POST /api/auth/login

Authenticates a user and returns a JWT token.

```

// Request body
{
"email": "user@example.com",
"password": "password123"
}

```

#### Response

```

{
"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
"user": {
"id": "123",
"name": "John Doe",
"email": "user@example.com"
}
}

```
```

By following these conventions, teams can maintain consistency across projects and leverage AI tools effectively while maintaining high code quality and readability.

[^1]: https://dreamix.eu/insights/node-js-project-structure-a-short-guide/

[^2]: https://github.com/millermedeiros/mdoc

[^3]: https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Code_style_guide/JavaScript

[^4]: https://github.com/airbnb/javascript

[^5]: https://github.com/felixge/node-style-guide

[^6]: https://dev.to/mehedihasan2810/nodejs-best-practices-a-guide-for-developers-4d65

[^7]: https://github.com/nodejs/node/blob/main/doc/README.md

[^8]: https://engineering.axur.com/2025/05/09/best-practices-for-ai-assisted-coding.html

[^9]: https://gist.github.com/sandren/0f22e116f01611beab2b1195ab731b63

[^10]: https://dev.to/mr_ali3n/folder-structure-for-nodejs-expressjs-project-435l

[^11]: https://www.npmjs.com/package/airbnb-style

[^12]: https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Howto/Markdown_in_MDN

[^13]: https://www.reddit.com/r/node/comments/nv59cq/how_to_structure_your_nodejs_project_to_fit/

[^14]: https://www.reddit.com/r/javascript/comments/30hyan/airbnb_javascript_style_guide_a_mostly_reasonable/

[^15]: https://reintech.io/blog/structuring-a-nodejs-project-a-comprehensive-guide-for-software-developers

[^16]: http://airbnb.io/javascript/react/

[^17]: https://gist.github.com/tracker1/59f2c13044315f88bee9

[^18]: https://news.ycombinator.com/item?id=9822975

[^19]: https://wearedev.hashnode.dev/1-best-practices-nodejs-backend-folder-structure

[^20]: https://github.com/mitsuruog/airbnb.javascript

[^21]: https://www.markdowntoolbox.com/blog/markdown-best-practices-for-documentation/

[^22]: https://github.com/felixge/node-style-guide

[^23]: https://www.w3schools.com/js/js_conventions.asp

[^24]: https://www.reddit.com/r/node/comments/1itprhz/nodejs_best_practices_guide/

[^25]: https://www.reddit.com/r/javascript/comments/1hjh8lj/i_made_a_markdown_documentation_hosting_server_in/

[^26]: https://www.mkdocs.org/user-guide/writing-your-docs/

[^27]: https://forum.godotengine.org/t/node-js-script-for-generating-markdown-docs/54431

[^28]: https://www.markdownguide.org/basic-syntax/

[^29]: https://github.com/millermedeiros/mdoc

[^30]: https://experienceleague.adobe.com/en/docs/contributor/contributor-guide/writing-essentials/markdown

