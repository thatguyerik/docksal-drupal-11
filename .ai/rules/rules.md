# Project Development Guidelines

## Project Overview

This is a Drupal 11 project using the recommended project template with Docksal for local development. The project follows modern Drupal development practices with composer-based dependency management and symlinked project structure.

## Important Environment Note

**CRITICAL:** Always assume your terminal is a Linux environment. Never attempt to discern the host environment for yourself. All Docksal containers are Linux-based, and the production host will be a Linux environment. All terminal commands, paths, and operations should be treated as Linux-based, regardless of the development host platform.

## Build/Configuration Instructions

### Prerequisites
- Docksal installed and configured
- Docker environment running

### Initial Setup
1. **Clone the repository** and navigate to the project directory
2. **Start Docksal environment**:
   ```bash
   fin start
   ```
3. **Initialize the project** (full reset):
   ```bash
   fin init
   ```

### Project Initialization Process
The `fin init` command performs a two-step process:
1. **Stack initialization**: Resets the Docksal project stack (`fin project reset -f`).
2. **Site initialization**: Runs the `init-site` command which:
   - Installs Composer dependencies (`composer install`).
   - Fixes file permissions for the site directory.
   - Copies settings templates (from `example.settings.local.php`).
   - Installs Drupal. If `web/sites/default/settings.php` and `settings.local.php` already exist, it installs using existing configuration (`drush site-install minimal --existing-config`). Otherwise, it performs a fresh minimal install using the DB URL `mysql://root:root@db/default`.
   - Applies the default content recipe (`recipes/grantparish_default_content`).
   - Builds the theme assets (Vite build) via the Docksal command.
   - Generates sample users, media, and content using the Devel module.

### Environment Configuration
- **Virtual Host**: `docksal-drupal-11.docksal.site`
- **Document Root**: `web/`
- **LAMP Stack**: Default Docksal stack
- **XDebug**: Disabled by default (enable by adding `XDEBUG_ENABLED=1` to `docksal-local.env`)

### Project Structure
The project uses symlinks to map the `project/` directory structure to the runtime directories. Managed via Composer script `project-scaffold` and `kporras07/composer-symlinks`:
- `project/modules` → `web/modules/custom`
- `project/theme` → `web/themes/custom`
- `project/profiles` → `web/profiles/custom`
- `project/recipes` → `recipes`
- `project/sites/default/settings.php` → `web/sites/default/settings.php`
- `project/sites/default/settings.local.php` → `web/sites/default/settings.local.php`
- `project/sites/default/services.yml` → `web/sites/default/services.yml`
- `project/sites/default/development.services.yml` → `web/sites/default/development.services.yml`
- `project/config` → `config`

### Composer Scripts
- **project-scaffold**: Sets up symlinks and manages settings files
- **post-autoload-dump**: Automatically runs project-scaffold after composer operations

### Theme Frontend (Vite/Sass/Bootstrap)

The project’s frontend assets live inside the custom theme directory at `${THEME_PATH}` (defaults to `project/theme/grantparish`). The theme uses Vite and Sass with Bootstrap 5.

- Tooling:
  - Vite 7 for build and dev server
  - Sass for stylesheet compilation
  - Bootstrap 5 for UI components

#### Build Process

Primary build command (recommended):
```bash
fin theme/build
```
This Docksal command runs inside the CLI container and performs:
1. `npm install` in `${THEME_PATH}`
2. `npm run build` (Vite production build)

Notes:
- Prefer `fin theme/build` over running npm locally to ensure a consistent containerized environment.
- The Vite output is written under the theme directory (e.g., `${THEME_PATH}/dist`).

#### Local Development

- Start the Vite dev server:
```bash
fin exec "cd ${THEME_PATH} && npm run dev"
```
- One‑off production build:
```bash
fin theme/build
```

#### Integration with Drupal

- The theme’s libraries should reference the built assets from the Vite output directory.
- Attach the appropriate theme library in Twig templates using `attach_library()`.

There is no separate web‑components design system or Storybook in this repository at this time. If you re‑introduce one, add corresponding Docksal commands and document them here.

## Testing Information

### Testing Framework
The project uses **PHPUnit 10+** for testing, integrated with Drupal's testing infrastructure.

### Test Types Available
1. **Unit Tests**: For testing individual classes and services
2. **Kernel Tests**: For testing with minimal Drupal bootstrap
3. **Functional Tests**: For full browser-based testing
4. **FunctionalJavascript Tests**: For JavaScript-enabled browser testing

### Running Tests

#### Using Docksal
Tests should be run within the Docksal CLI container:
```bash
fin exec "vendor/bin/phpunit --configuration web/core/phpunit.xml.dist"
```

#### Running Specific Test Groups
```bash
fin exec "vendor/bin/phpunit --group [group_name]"
```

#### Running Tests for Custom Modules
```bash
fin exec "vendor/bin/phpunit web/modules/custom/[module_name]/tests/"
```

### Adding New Tests

#### Directory Structure
Custom module tests should follow this structure:
```
project/modules/[module_name]/
├── tests/
│   └── src/
│       ├── Unit/           # Unit tests
│       ├── Kernel/         # Kernel tests
│       ├── Functional/     # Functional tests
│       └── FunctionalJavascript/  # JS tests
```

#### Test Example
A complete test example has been created in `project/modules/test_example/`:

**Service Class** (`src/Calculator.php`):
```php
<?php
namespace Drupal\test_example;

class Calculator {
  public function add($a, $b) {
    return $a + $b;
  }

  public function divide($a, $b) {
    if ($b == 0) {
      throw new \InvalidArgumentException('Division by zero is not allowed.');
    }
    return $a / $b;
  }
}
```

**Unit Test** (`tests/src/Unit/CalculatorTest.php`):
```php
<?php
namespace Drupal\Tests\test_example\Unit;

use Drupal\test_example\Calculator;
use Drupal\Tests\UnitTestCase;

/**
 * @group test_example
 */
class CalculatorTest extends UnitTestCase {

  protected $calculator;

  protected function setUp(): void {
    parent::setUp();
    $this->calculator = new Calculator();
  }

  public function testAdd() {
    $this->assertEquals(5, $this->calculator->add(2, 3));
  }

  public function testDivideByZero() {
    $this->expectException(\InvalidArgumentException::class);
    $this->calculator->divide(10, 0);
  }
}
```

#### Test Execution
```bash
fin exec "vendor/bin/phpunit --group test_example"
```

### Test Configuration
- **PHPUnit Configuration**: `web/core/phpunit.xml.dist`
- **Test Bootstrap**: `web/core/tests/bootstrap.php`
- **Environment Variables**: Tests require `SIMPLETEST_BASE_URL` for functional tests

## Code Style and Development Standards

### PHP CodeSniffer (PHPCS)
The project uses Drupal coding standards enforced by PHPCS:
- **Configuration**: `web/core/phpcs.xml.dist`
- **Standards**: Drupal and DrupalPractice coding standards
- **Usage**: Extensive use of `phpcs:ignore` comments for specific rule exceptions

### Code Style Guidelines
1. **Follow Drupal Coding Standards**: Use proper indentation, naming conventions, and documentation
2. **PHPDoc Comments**: All classes, methods, and properties should have proper documentation
3. **Namespace Structure**: Follow PSR-4 autoloading standards
4. **Exception Handling**: Use appropriate exception types and messages

### Development Tools
- **Drush**: Command-line tool for Drupal operations (`vendor/bin/drush`)
- **Composer**: Dependency management and project scaffolding
- **Docksal**: Local development environment with Docker

### File Permissions
- Site directory permissions are managed automatically by the init scripts
- Custom modules and themes should be placed in the `project/` directory structure

### Configuration Management
- **Config Directory**: `project/config/sync/` (symlinked to `config/sync/`)
- **Settings Files**: Managed through symlinks from `project/sites/default/`
- **Local Settings**: Use `settings.local.php` for environment-specific configurations

### Custom Development
- **Custom Modules**: Place in `project/modules/`
- **Custom Themes**: Place in `project/theme/`
- **Custom Profiles**: Place in `project/profiles/`

### Debugging
- **XDebug**: Available but disabled by default
- **Error Logging**: Configure in `settings.local.php`
- **Development Mode**: Enable in local settings for verbose error reporting

## Additional Notes

- The project uses Drupal 11.2 with admin_toolbar module
- Symlinks are automatically managed by composer scripts
- Database connection uses root/root credentials in the Docksal environment
- The site can be accessed at `http://grantparish.docksal.site` after initialization
- Configuration can be imported/exported using Drush commands within the Docksal environment
