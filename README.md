# Docksal + Drupal 11 — Quick Start

A minimal Drupal 11 starter that runs on Docksal. This project lets you spin up a clean Drupal site with one command, using Composer-driven workflows and a simple `project/` workspace that is symlinked into runtime paths.

## Prerequisites
- Docker running on your machine
- Docksal installed (see Linux install below)
- `fin` available in your shell (installed with Docksal)

## Install Docksal on Linux (quick)
1. Ensure Docker is installed and running (Docker Engine + Compose plugin).
2. Install Docksal via the official installer script:
   ```bash
   curl -fsSL https://get.docksal.io | bash
   ```
3. Start Docksal system services:
   ```bash
   fin system start
   ```

For detailed instructions and supported distributions, see: https://docs.docksal.io/installation/linux/

## Setup
1. Edit `.docksal/docksal.env` with settings specific to your project.
2. Optional: create `.docksal/docksal-local.env` to enable Xdebug (set `XDEBUG_ENABLED=1`) and/or add config for AI coding assistants.
3. Initialize the project (first run or to reinitialize):
   ```bash
   fin init
   ```

## Site URL (VIRTUAL_HOST)
- Docksal uses the `VIRTUAL_HOST` variable to define the site URL.
- By default, if `VIRTUAL_HOST` is not set, Docksal derives it from the project directory name: `<folder-name>.docksal.site`.
  - Example: if your project folder is `my-project`, the default URL will be `http://my-project.docksal.site`.
- To change the URL, set `VIRTUAL_HOST` in `.docksal/docksal.env` (or `.docksal/docksal-local.env`) and restart:
  ```bash
  # .docksal/docksal.env
  VIRTUAL_HOST=my-site.docksal.site

  fin project restart
  ```

## Result
- Running `fin init` produces a minimal Drupal installation.
- Add contrib or custom modules or themes according to your project's needs (put your code under `project/`; composer scripts symlink to runtime paths).
  - Use the `fin theme/new` command to generate a new theme using Drupal's Theme Starter Kit.
  - Example: `fin theme/new --name="My Theme" --machine-name="my_theme" --description="A custom theme for myself."`
  - See `.docksal/commands/theme/new` for more details, or run `fin help theme/new`.

## Defaults
- Document root: `web/`
- Example default virtual host: `http://docksal-drupal-11.docksal.site` (if your folder name is `docksal-drupal-11`).

## Notes
- All commands are intended to run inside the Docksal (Linux) environment.
