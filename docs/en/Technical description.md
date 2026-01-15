# Focus CMS – Technical Overview

## Project direction and openness

Focus CMS is a functional system that is actively used in real-world scenarios.
As more people increasingly appreciate clean and simple solutions, the idea emerged that the project could later become available in an open form.

According to the concept, the code could be released under the **MIT license**.
If a small developer community forms around it, Focus CMS could become a useful and enjoyable alternative for others as well.

Before such a step, however, it is necessary to:
- review the codebase,
- determine whether it is suitable as a shared foundation,
- and ensure that it does not contain structural or logical issues
  that would inherently prevent meaningful collaboration.

Before opening the project, support would be especially valuable in the following areas:
- writing automated tests,
- security auditing,
- long-term code optimization.

---

## Architecture – core principles

Focus CMS follows a **monolithic architecture** with a **modular mindset**.

All functionality is implemented directly as part of the system.
The goal is not a “plugin-heavy” approach (a WordPress-style Swiss army knife), but a clean, integrated solution.

The philosophy is closer to an *Apple-like* direction:
- fewer features,
- but stable,
- well thought out,
- and sustainable in the long term.

---

## Theme architecture and extensibility

Themes are not merely presentation layers.

### Installation and structure
- Themes are installed via Composer
- Through a dedicated module installer package
- Placed in their own directory, separated from the application core

Theme development is **completely independent** of the core application code.

---

### Service Provider–based approach

Each theme has its own Service Provider
(e.g. under the `Themes\FocusDefaultTheme\Providers` namespace).

As a result, a theme:
- does not only contain Blade views,
- but also dedicated PHP classes,
- responsible for handling layouts, components, and related logic.

```php
\Themes\FocusDefaultTheme\Classes\Layouts\Components\PublicDefault::class => 'public-default',
\Themes\FocusDefaultTheme\Classes\Layouts\Components\MaintenanceDefault::class => 'maintenance-default',
```

This allows that:
- layouts can be used as Blade components (`<x-public-default>`)
- each layout can optionally be backed by a concrete PHP class
- view handling is structured, extensible, and easy to follow
- the theme carries not only presentation, but also logical responsibility
- themes can be developed locally and installed via Composer from their own Git repositories

---

## Functional extension at theme level

Through its own Service Provider, a theme can independently extend the behavior of the application:

- defining custom routes
- creating custom controllers
- registering custom middleware
- adding helpers and service bindings

All of this is done **without modifying the core code**.

In this model, a theme is **optionally** not just a “skin”, but a **functional module** with its own responsibility.

---

## Independent theme development (separate Git + symlink)

Themes can be developed in separate Git repositories:
- with independent version control
- their own release cycles
- independently of the application codebase

During local development, a theme is symlinked into the appropriate application directory, allowing:
- immediate feedback on changes
- no need for constant reinstallation
- identical local and production behavior

---

## Complete separation of admin and frontend

The frontend theme is completely independent from the admin interface:

- separate view structures
- separate CSS and JavaScript assets
- separate Vite / NPM build processes
- no shared assets or UI dependencies

The admin and frontend:
- do not share views
- do not share styles
- do not share JavaScript logic

This ensures that the admin remains stable and predictable,
while the frontend can freely experiment with UI/UX,
avoiding any form of “cross-contamination”.

---

## Build system and tooling

- Tailwind CSS–based admin interface
- Frontend themes are also built on Tailwind CSS
- Alpine.js usage
- jQuery integrated in a compatible way
- Modular packages installed via NPM

### Build processes
- `npm build --all` → application + themes
- `npm build app` → application only
- theme switching handled via NPM + Vite configuration

---

## Features (non-exhaustive list)

- Web-based admin interface
- CLI commands (e.g. theme switching, maintenance tasks)
- Brevo integration (email sending)
- Google reCAPTCHA + Cloudflare protection
- Two-factor authentication (email, Authy-compatible)
- Taxonomies:
  - categories (hierarchical)
  - tags (non-hierarchical)
  - configurable extensibility
- Pages and posts
- Widget system:
  - widget areas defined at theme level
  - widgets insertable from the admin via shortcodes
  - widgets embeddable into content as full-featured posts
- Markdown-based content handling
- Custom image and file management:
  - mobile-optimized
  - sortable within albums
- Code blocks with syntax highlighting
- Email notifications for login events
- Currently used primarily in a single-user setup (by me)
- Architecture prepared for multi-user operation
- Database designed for multi-user support
- Multiple users can be created via CLI

---

## Development and deployment model

- Local development on localhost
- Docker-based virtualization
- Apache + PHP
- Node.js
- MariaDB

The Apache + PHP and Node.js containers point to a filesystem outside the containers,
allowing development (e.g. with VS Code) to take place directly on the host system.

Containers are configured on a shared network.

### Typical commands

Apache container:
- `php artisan serve`
- `composer update`

Node container:
- `npm update`
- `npm run build`

Host:
- Git operations

### Deployment and updates

- private Git repository
- deployment via SSH on the server:
  - `git pull`
  - `composer update`
  - `php artisan migrate`
  - `php artisan optimize:clear`
  - `php artisan optimize`
