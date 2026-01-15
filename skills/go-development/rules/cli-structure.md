---
title: Use Cobra for CLI Applications
impact: MEDIUM
impactDescription: Standardizes CLI structure and argument parsing
tags: cli, cobra
---

## Use Cobra for CLI Applications

**Impact: MEDIUM**

Use `spf13/cobra` to build powerful and modern CLI applications. It provides a standard structure for commands, flags, and help text generation.

**CLI Directory Structure:**

- Single project can have multiple cli applications separated by the `cmd` directory.
- Each command and subcommand is a separate file.

```
.
├── cmd
    ├── app-name
        └── commands
        │   ├── server.go
        │   ├── server_start.go
        │   ├── server_migrate.go
        │   └── version.go
        │   └── root.go
        └── main.go
```

**Structured Cobra commands:**

Root command will not run any logic, it will only add subcommands and flags.

```go
package commands

import (
    "github.com/spf13/cobra"
)

type opts struct {
	configPath string
}

func NewCommand() *cobra.Command {
	opts := &opts{}
	command := &cobra.Command{
		Use:   "app-name",
		Short: "Application Description",
        // Root command doesn't typically run logic if there are subcommands
		Run: func(c *cobra.Command, args []string) {
			c.HelpFunc()(c, args)
		},
	}

    // Add subcommands
	command.AddCommand(
		newServer(opts),
		newVersion(),
	)

    // Persistent flags available to all commands
	command.PersistentFlags().StringVar(&opts.configPath, "config", "config.yaml", "path to config file")

	return command
}
```

**Subcommand Example:**

```go
func newServer(opts *opts) *cobra.Command {

	type config struct {
		port int
	}
	var cfg config
	command := &cobra.Command{
		Use:   "server",
		Short: "Start the server",
		Run: func(c *cobra.Command, args []string) {
			// Server logic here
		},
	}
	// flag available to current command
	command.Flags().IntVar(&cfg.port, "port", 8080, "port to listen on")
	return command
}
```

Reference: [Cobra Documentation](https://github.com/spf13/cobra)
