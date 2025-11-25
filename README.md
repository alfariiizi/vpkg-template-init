# VPKG Namespace Template

This is a complete template for creating your own VPKG namespace/registry. Use this to publish your own reusable Vandor packages.

## What You Get

This template provides a complete namespace structure:

```
your-namespace/
├── index.yaml              # Namespace index (auto-generated)
├── README.md              # This file
├── example-http/          # Example package
│   ├── meta.yaml          # Package metadata
│   ├── config.go.tmpl     # Configuration
│   └── module.go.tmpl     # FX module
└── example-cache/         # Another example
    ├── meta.yaml
    └── handler.go.tmpl
```

## Quick Start

### 1. Initialize Your Namespace

```bash
# Initialize from template
vandor vpkg init ./my-namespace

cd my-namespace
```

### 2. Customize Your Namespace

**Update `index.yaml`:**
```yaml
namespace: my-namespace        # Change this!
description: My awesome packages
packages:
  - name: example-http
    version: v1.0.0
    description: HTTP server implementation
    tags:
      - transport
      - http
    type: fx-module
```

### 3. Create Your First Package

**Option A: Modify existing examples**
```bash
# Rename example-http to your package name
mv example-http my-http-server

# Update the meta.yaml
nano my-http-server/meta.yaml
```

**Option B: Create from scratch**
```bash
# Create new package directory
mkdir my-auth-service

# Create meta.yaml
vandor vpkg meta ./my-auth-service

# Add your code (.go.tmpl files)
```

### 4. Sync Index

After adding/updating packages, sync the index:

```bash
vandor vpkg index .
```

This will scan all `meta.yaml` files and update `index.yaml` automatically.

### 5. Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit: my-namespace packages"

# Create repo on GitHub: https://github.com/your-username/my-namespace
git remote add origin https://github.com/your-username/my-namespace.git
git branch -M main
git push -u origin main
```

### 6. Register with Vandor (Optional)

To make your packages publicly available through Vandor:

1. Fork https://github.com/alfariiizi/vandor
2. Edit `registry.yaml`:
   ```yaml
   namespaces:
     my-namespace:
       repository: https://github.com/your-username/my-namespace
       packages_path: ""
       type: free
       official: false
   ```
3. Create Pull Request
4. Once merged, users can install: `vandor vpkg add my-namespace/my-http-server`

## Package Structure

Each package should have:

### Required Files

- **`meta.yaml`** - Package metadata (name, version, description)
- **`*.go.tmpl`** - Go template files (use `{{.Module}}` for imports)

### Example Package Structure

```
my-package/
├── meta.yaml           # Package metadata
├── config.go.tmpl      # Configuration
├── module.go.tmpl      # FX module registration
├── handler.go.tmpl     # HTTP handlers
└── README.md           # Package docs (optional)
```

## Template Variables

Your `.go.tmpl` files can use:

- **`{{.Module}}`** - Target project's module name
  ```go
  import "{{.Module}}/internal/pkg/logger"
  ```

## Commands Reference

```bash
# Initialize namespace from template
vandor vpkg init ./my-namespace

# Create package metadata
vandor vpkg meta ./my-namespace/my-package

# Generate/update index.yaml
vandor vpkg index ./my-namespace

# Test package installation
vandor vpkg add my-namespace/my-package --test-dir=./my-namespace/my-package

# Create package from existing code
vandor vpkg create --from=./src --to=./my-namespace/my-package --with-meta
```

## Workflow Example

### Adding a New Package

```bash
cd my-namespace

# 1. Create package directory
mkdir redis-cache

# 2. Create metadata
vandor vpkg meta ./redis-cache

# 3. Add your code
cat > redis-cache/client.go.tmpl << 'EOF'
package redis

import (
    "context"
    "github.com/redis/go-redis/v9"
    "{{.Module}}/internal/pkg/logger"
)

type Client struct {
    client *redis.Client
    logger logger.Logger
}

func NewClient(config Config, logger logger.Logger) *Client {
    rdb := redis.NewClient(&redis.Options{
        Addr: config.Addr,
    })
    return &Client{client: rdb, logger: logger}
}
EOF

# 4. Update index
vandor vpkg index .

# 5. Commit and push
git add redis-cache/
git add index.yaml
git commit -m "Add redis-cache package"
git push
```

## Best Practices

### Package Naming
- Use lowercase with hyphens: `http-server`, `redis-cache`
- Be descriptive: `auth-jwt` not just `auth`
- Avoid redundant namespace: `cache` not `my-namespace-cache`

### Versioning
- Follow semantic versioning: `v1.0.0`, `v1.2.3`
- Update version in `meta.yaml` when changing code
- Run `vandor vpkg index .` after version updates

### Code Organization
- One package = one responsibility
- Use FX modules for dependency injection
- Follow Vandor's hexagonal architecture patterns
- Include configuration structs

### Testing
```bash
# Test locally before pushing
vandor vpkg add your-namespace/your-package --test-dir=./your-package

# Test in real project
cd /path/to/test-project
vandor vpkg add your-namespace/your-package --test-dir=/path/to/your-namespace/your-package
```

## Maintenance

### Updating Packages

```bash
# 1. Update package code/metadata
nano my-package/meta.yaml  # Bump version

# 2. Sync index
vandor vpkg index .

# 3. Commit
git add my-package/ index.yaml
git commit -m "Update my-package to v1.1.0"
git push
```

### Adding Dependencies

If your package depends on other packages, document in `meta.yaml`:

```yaml
dependencies:
  - vandor/logger
  - vandor/config
```

## Support

- **Vandor Docs**: https://github.com/alfariiizi/vandor
- **Issues**: https://github.com/alfariiizi/vandor/issues
- **Examples**: See official packages at https://github.com/alfariiizi/vandor/tree/main/vpkg-packages/vandor

## License

MIT License - See LICENSE file
