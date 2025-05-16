# WordPress Development Conventions

This document outlines comprehensive conventions for WordPress development, synthesizing best practices from WordPress Core, VIP, 10up, and industry standards. It covers PHP, JavaScript, CSS, HTML, security, performance, accessibility, project structure, and documentation.

---

## Table of Contents

1. [General Philosophy](#general-philosophy)
2. [Project Structure](#project-structure)
3. [PHP](#php)
4. [JavaScript](#javascript)
5. [CSS](#css)
6. [HTML & Markup](#html--markup)
7. [Accessibility](#accessibility)
8. [Security](#security)
9. [Performance](#performance)
10. [Internationalization](#internationalization)
11. [Documentation](#documentation)
12. [Migrations](#migrations)
13. [Third-Party Integrations & API Keys](#third-party-integrations--api-keys)
14. [Testing](#testing)
15. [Version Control](#version-control)

---

## General Philosophy

- Prioritize maintainability, readability, and collaboration.
- Follow the principle of least surprise: code should be easy to understand for any WordPress developer.
- Prefer convention over configuration.
- Emphasize progressive enhancement, accessibility, and performance.
- Write modular, decoupled, and reusable code.
- Favor verbose, clear documentation and code comments, especially for "why" and "what".

---

## Project Structure

Follow the [10up recommended structure](https://10up.github.io/Engineering-Best-Practices/structure/):

```
|- assets/
|  |- css/
|  |- fonts/
|  |- images/
|  |- js/
|  |- svg/
|- bin/
|- gulp-tasks/
|- includes/
|    |- classes/
|- languages/
|- node_modules/
|- partials/
|- templates/
|- tests/
|  |- php/
|  |- js/
|- vendor/
|- .babelrc
|- .editorconfig
|- .eslintrc
|- composer.json
|- gulpfile.babel.js
|- package.json
|- webpack.config.babel.js
```

- Use Composer for PHP dependencies and npm for JS/CSS.
- Document all third-party integrations in `INTEGRATIONS.md`.
- Never commit secrets or production API keys to the repository.
- All projects must include a `.editorconfig` file and use tabs for indentation in PHP, JS, HTML, and CSS.

---

## PHP

- **Coding Standards:** Follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/) and [VIP Coding Standards](https://github.com/Automattic/VIP-Coding-Standards/).
- **Namespaces:** All code outside theme templates should use unique, project-specific namespaces.
- **Class Naming:** Use `Camel_Snake_Case` for classes, all-caps with underscores for constants.
- **Functions & Variables:** Use lowercase with underscores. Avoid abbreviations.
- **Visibility:** Always declare visibility (`public`, `protected`, `private`).
- **No Shorthand Tags:** Always use `<?php ... ?>`.
- **Yoda Conditions:** Use Yoda conditions for equality checks.
- **Array Syntax:** Use long array syntax: `array( ... )`.
- **Late Escaping:** Escape output as late as possible, using the appropriate WordPress escaping function.
- **Sanitization & Validation:** Always sanitize and validate input before saving to the database.
- **Internationalization:** All user-facing strings must be translatable and use a literal text domain.
- **No Sessions:** Avoid PHP sessions; use cookies or client-side storage.
- **No Heredoc/Nowdoc:** Avoid for output; use templates and late escaping.
- **No extract(), eval(), or goto.**
- **Hooks:** Register hooks outside of constructors for testability.
- **Composer:** All projects must have a `composer.json`.
- **No direct database queries:** Use WordPress APIs and `$wpdb->prepare()` for custom queries.
- **No direct file writes:** Use the WordPress Filesystem API.
- **No direct output:** Use return values and template parts for output, not echo/print in logic code.

---

## JavaScript

- **Coding Standards:** Follow [WordPress JS Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/) and [10up JS Best Practices](https://10up.github.io/Engineering-Best-Practices/javascript/).
- **Linting:** Use [10up's eslint-config](https://github.com/10up/eslint-config).
- **Modern JS:** Use ES6+ features, classes, arrow functions, destructuring, and modules.
- **Single Quotes:** Use single quotes for strings.
- **Semicolons:** Always use semicolons.
- **Tabs:** Use tabs for indentation.
- **Naming:** Use camelCase for variables and functions, UpperCamelCase for classes, SCREAMING_SNAKE_CASE for constants.
- **No window pollution:** Avoid adding to the global namespace; use closures or modules.
- **Security:** Never use `innerHTML` with untrusted data; prefer `textContent` or DOM APIs.
- **Performance:** Cache DOM queries, use event delegation, debounce/throttle expensive operations.
- **Accessibility:** Ensure dynamic content is accessible (e.g., ARIA updates).
- **Documentation:** Use JSDoc for functions, classes, and modules.
- **No barrel files:** Import only what you use, avoid index.js wildcards.
- **No direct DOM manipulation in React:** Use state and props.

---

## CSS

- **Coding Standards:** Follow [WordPress CSS Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/) and [10up CSS Best Practices](https://10up.github.io/Engineering-Best-Practices/css/).
- **Mobile First:** Write mobile-first, responsive CSS using min-width media queries.
- **Syntax:** One selector and one declaration per line. Use double quotes for font names.
- **Nesting:** Only for pseudo-classes, pseudo-elements, component states, and media queries.
- **Naming:** Use lowercase with hyphens. Use BEM or similar for components.
- **No !important:** Avoid unless absolutely necessary.
- **Performance:** Minify, concatenate, and purge unused CSS. Avoid base64 images and @import.
- **Accessibility:** Use `prefers-reduced-motion` for animations.
- **Documentation:** Use comment blocks for sections and complex rules.
- **No CSS-in-JS for WordPress themes/plugins:** Use external stylesheets.

---

## HTML & Markup

- **Coding Standards:** Follow [WordPress HTML Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/) and [10up Markup Best Practices](https://10up.github.io/Engineering-Best-Practices/markup/).
- **Semantic HTML:** Use semantic elements (`header`, `nav`, `main`, `article`, etc.).
- **Accessibility:** Use ARIA roles, skip links, and ensure forms are accessible.
- **Tabs:** Use tabs for indentation.
- **Minimal & Valid:** Write minimal, valid markup. Test with the W3C validator.
- **Schema.org:** Use JSON-LD for structured data where appropriate.
- **SVG:** Optimize SVGs, use ARIA for accessibility, and prefer inline SVG for icons.
- **No presentational markup:** Use CSS for presentation, not HTML.

---

## Accessibility

- **WCAG Compliance:** All projects must meet at least WCAG 2.1 Level A; Level AA for global/enterprise projects.
- **Forms:** All fields must have labels; use fieldsets for grouping.
- **Bypass Blocks:** Implement skip links and ARIA landmarks.
- **Keyboard Navigation:** All interactive elements must be keyboard accessible.
- **ARIA:** Use ARIA attributes to describe states and properties.
- **Testing:** Use automated tools (e.g., pa11y) and manual testing.
- **Accessible media:** Provide alt text for images, captions for video, and transcripts for audio.
- **No autoplay audio:** If used, provide controls.

---

## Security

- **Passwords:** Enforce strong passwords and MFA for admin accounts.
- **File Edits:** Set `DISALLOW_FILE_MODS` to true.
- **Remove Unused Code:** Delete unused plugins, themes, and code.
- **Vulnerability Scanning:** Regularly scan for vulnerable packages.
- **X-Frame-Options:** Set to `SAMEORIGIN`.
- **Disable XML-RPC:** Unless explicitly needed.
- **Updates:** Keep WordPress, plugins, and themes up to date.
- **No 'admin' Username:** Avoid common usernames.
- **Disable User Enumeration:** Limit user data exposure via REST API.
- **API Keys:** Store in `wp-config.php` or options, never in source code.
- **No Sessions:** Avoid PHP sessions; use cookies or client-side storage.
- **No direct output of user data:** Always escape output.
- **No direct file uploads:** Use WordPress APIs and validate file types.

---

## Performance

- **Core Web Vitals:** Optimize for LCP, CLS, and FID.
- **Images:** Use responsive images, modern formats (WebP), lazy loading, and CDNs.
- **JS/CSS:** Minify, compress, code-split, and defer non-critical assets.
- **Fonts:** Use `font-display: swap`, serve from CDN, subset, and preload critical fonts.
- **Resource Hints:** Use `preconnect`, `preload`, and `dns-prefetch` for critical resources.
- **Caching:** Use object cache, transients, and page caching appropriately.
- **Database:** Avoid unbounded queries, use `no_found_rows`, and cache results.
- **Third-Party Scripts:** Audit, defer, and lazy-load where possible.
- **No posts_per_page = -1:** Always set reasonable limits.
- **No post__not_in for large queries:** Filter in PHP if possible.

---

## Internationalization

- **Text Domains:** Use a unique, literal text domain for all strings.
- **Escaping:** Always escape translated strings on output.
- **No Variable Strings:** Never use variables/constants for translation strings or text domains.
- **Pluralization:** Use `_n()` and context functions as needed.
- **No string interpolation in translation functions:** Use `sprintf` with placeholders.

---

## Documentation

- **PHP:** Use PHPDoc for all classes, methods, and functions. Include long descriptions for "why" and "what".
- **JS:** Use JSDoc for all functions, classes, and modules.
- **CSS:** Use comment blocks for sections and complex rules.
- **README:** Every project must have a `README.md` with setup, build, and deployment instructions.
- **INTEGRATIONS.md:** Document all third-party integrations and API keys.
- **Changelogs:** Maintain a clear changelog for releases.

---

## Migrations

- **Plan:** Document a detailed migration plan, including data mapping, risks, and rollback procedures.
- **Scripts:** Use WP-CLI for migration scripts; test thoroughly in all environments.
- **Media:** Plan for media migration and redirects.
- **Backups:** Always backup before running migrations.
- **No Social Triggers:** Prevent migration scripts from triggering social/email integrations.
- **Meta Tracking:** Save meta on migrated content for traceability.
- **No test users on production:** Remove test users after migration.

---

## Third-Party Integrations & API Keys

- **Documentation:** List all integrations in `INTEGRATIONS.md`.
- **No Hard-Coding:** Never hard-code production credentials.
- **Environment Awareness:** Use `WP_ENVIRONMENT_TYPE` to prevent production actions in non-prod environments.
- **UI for Keys:** Prefer a UI for managing API keys; otherwise, use constants or filters.
- **No secrets in version control:** Never commit secrets or keys.

---

## Testing

- **Automated Testing:** Use PHPUnit for PHP, Jest or similar for JS.
- **Accessibility Testing:** Use pa11y, axe, or similar tools.
- **Performance Testing:** Use Lighthouse, WebPageTest, or similar.
- **Manual Testing:** Always supplement automated tests with manual QA.
- **Test Coverage:** Strive for high coverage, especially for distributed code.

---

## Version Control

- **Git:** Use git for all projects.
- **Commits:** Write clear, descriptive commit messages (e.g., `feat:`, `fix:`, `docs:`, `refactor:`).
- **Branches:** Use feature branches and pull requests for all changes.
- **Lock Files:** Commit `composer.lock` and `package-lock.json` for reproducible builds.
- **No secrets:** Never commit secrets, API keys, or credentials.

---

## References

- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)
- [VIP Coding Standards](https://github.com/Automattic/VIP-Coding-Standards/)
- [10up Engineering Best Practices](https://10up.github.io/Engineering-Best-Practices/)
- [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)
- [WordPress Theme Handbook](https://developer.wordpress.org/themes/)

