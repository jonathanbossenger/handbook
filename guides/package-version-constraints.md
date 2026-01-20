# Package Version Constraints

When installing WP-CLI packages using the `wp package install` command, you can specify version constraints to control which version of a package is installed.

Version constraints are specified by appending a colon (`:`) followed by the constraint to the package name:

```bash
wp package install <package-name>:<version-constraint>
```

## Common Constraints

WP-CLI uses Composer's version constraint syntax. Here are some commonly used examples:

```bash
# Install a specific version
wp package install wp-cli/server-command:2.0.0

# Install with caret operator (allows non-breaking updates)
wp package install wp-cli/server-command:^1.0

# Install with tilde operator (allows patch-level updates)
wp package install wp-cli/server-command:~1.2

# Install the latest stable release
wp package install wp-cli/server-command:@stable

# Install from a development branch
wp package install wp-cli/server-command:dev-main
```

For complete documentation on version constraint syntax and operators, see the [Composer documentation on versions and constraints](https://getcomposer.org/doc/articles/versions.md).
