---
title: Use Viper for Configuration
impact: HIGH
impactDescription: Enables flexible configuration from multiple sources (env, file, flags)
tags: config, viper
---

## Use Viper for Configuration

**Impact: HIGH**

Use `spf13/viper` together with `yaml` tags to load configuration. This allows reading from config files (YAML/JSON), environment variables, and flags in a layered approach, while unmarshaling into strongly-typed structs.

**Directory Structure:**

`config` directory is used for configuration files and it is stored in `internal` packages.

```
.
├── internal
    └── config
        ├── config.go
        └── config.yaml
```

**Incorrect (hardcoded or direct environment access):**

```go
type Config struct {
    Port int
}

func Load() *Config {
    return &Config{
        Port: 8080, // Hardcoded
    }
}

// Or manually reading every env var
func LoadFromEnv() *Config {
    portStr := os.Getenv("PORT")
    // manual conversion logic...
    return &Config{...}
}
```

**Correct (Viper with struct tags):**

Each configuration struct should be in its own package. The `config.go` should be the configuration container.

```go
import (
    "strings"
    "github.com/spf13/viper"
)

// Use struct tags for mapping
type Config struct {
    DB     pg.Config    `yaml:"db"`
    Redis  redis.Config `yaml:"redis"`
}

func Load(configFile string) (*Config, error) {
	v := viper.New()
    // Replace dots with underscores for env vars (e.g. SERVER_PORT -> server.port)
	v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
	v.AutomaticEnv()
	v.SetEnvPrefix("APP_PREFIX")
	v.SetConfigType("yaml")

    // Read from file if provided
	if configFile != "" {
        v.SetConfigFile(configFile)
        if err := v.ReadInConfig(); err != nil {
            return nil, err
        }
    }

	var cfg Config
	if err := v.Unmarshal(&cfg); err != nil {
		return nil, err
	}

	return &cfg, nil
}
```

Reference: [Viper Documentation](https://github.com/spf13/viper)
