---
title: SQL Database Configuration
impact: MEDIUM
impactDescription: Standardizes database connection parameters and string generation
tags: sql, configuration, postgres
---

## SQL Database Configuration

Define a standard `SQL` configuration struct to manage database connection parameters consistently across the application. This struct should include helper methods to generate the `DatabaseUrl` and `DataSourceName` required by various database drivers and libraries.

**Configuration Struct:**

```go
type SQL struct {
	Host        string `yaml:"host"`
	Port        int    `yaml:"port"`
	User        string `yaml:"user"`
	Password    string `yaml:"password"`
	Database    string `yaml:"database"`
	SSLMode     string `yaml:"sslmode"`
	MaxIdleConn int    `yaml:"maxIdleConn"`
	MaxOpenConn int    `yaml:"maxOpenConn"`
}

func (s SQL) DatabaseUrl() string {
	return fmt.Sprintf("postgres://%s:%s@%s:%d/%s?sslmode=%s",
		s.User, s.Password, s.Host, s.Port, s.Database, s.SSLMode)
}

func (s SQL) DataSourceName() string {
	return fmt.Sprintf("user=%s password=%s host=%s port=%d dbname=%s sslmode=%s",
		s.User, s.Password, s.Host, s.Port, s.Database, s.SSLMode)
}
```

