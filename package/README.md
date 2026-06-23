# aux4 secret

Secret manager for aux4. This package provides the `secret` command group and dispatches to provider packages like `aux4/secret-1password` that handle the actual secret storage.

aux4 resolves `secret://` URIs at runtime so credentials never appear in `.aux4` configuration files.

## Installation

```bash
aux4 aux4 pkger install aux4/secret
```

## How Secrets Work

### The `secret://` URI

```text
secret://<provider>/<vault>/<item>/<field>
```

| Segment    | Description                                                                 |
|------------|-----------------------------------------------------------------------------|
| `provider` | The secret provider name (e.g., `1password`). Must match an installed `aux4/secret-<provider>` package |
| `vault`    | The vault or namespace where the secret lives                               |
| `item`     | The item name. Can contain `/` for nested paths                             |
| `field`    | The field to retrieve (e.g., `password`, `username`, `credential`)          |

Examples:

```text
secret://1password/Personal/GitHub/password
secret://1password/Work/AWS/access-key
```

### Using Secrets

In variable defaults:

```json
{
  "name": "apiToken",
  "default": "secret://1password/Work/deploy-service/credential"
}
```

In `set:` instructions:

```json
"execute": [
  "set:password=secret://1password/Work/database/password",
  "connect.sh --password ${password}"
]
```

Via `--var` flag:

```bash
aux4 my-command --var apiKey=secret://1password/Work/service/api-key
```

The user can always override a secret default with an explicit `--varName <value>`.

### Batching

When multiple variables reference the same provider and item, aux4 fetches them in a single call:

```json
{
  "name": "username",
  "default": "secret://1password/Work/MyApp/username"
},
{
  "name": "password",
  "default": "secret://1password/Work/MyApp/password"
}
```

### OTP (One-Time Password)

Use `otp` as the field name to retrieve a TOTP code:

```text
secret://1password/Work/GitHub/otp
```

## Available Providers

| Provider | Package | Backend |
|----------|---------|---------|
| `1password` | `aux4/secret-1password` | [1Password CLI](https://developer.1password.com/docs/cli/) |

```bash
aux4 aux4 pkger install aux4/secret-1password
```

## Provider Command Contract

Every secret provider must implement the following commands under the `secret:<provider>` profile. The commands and their interfaces are the same across all providers.

### get (required)

Retrieves secret fields as a JSON object.

```bash
aux4 secret <provider> get --ref <vault/item> --fields <field1,field2,...> [--otp <true|false>]
```

| Variable | Description | Default |
|----------|-------------|---------|
| `ref`    | Secret reference path (e.g., `Personal/GitHub`) | required |
| `fields` | Comma-separated field names to retrieve | required |
| `otp`    | Include one-time password | `false` |

Output:

```json
{
  "username": "john",
  "password": "abc123"
}
```

### list (optional)

Lists secrets as `secret://` references. Users can copy these directly into `.aux4` files.

```bash
aux4 secret <provider> list [--vault <name>] [--withFields <true|false>]
```

| Variable | Description | Default |
|----------|-------------|---------|
| `vault`  | Filter by vault name | all vaults |
| `withFields` | Show field names for each item | `false` |

Output:

```text
secret://provider/Vault/ItemName
secret://provider/Vault/AnotherItem
```

### search (optional)

Searches secrets by name (case-insensitive).

```bash
aux4 secret <provider> search --query <text> [--vault <name>] [--withFields <true|false>]
```

| Variable | Description | Default |
|----------|-------------|---------|
| `query`  | Search query | required |
| `vault`  | Filter by vault name | all vaults |
| `withFields` | Show field names for each item | `false` |

### create (optional)

Creates a new secret.

```bash
aux4 secret <provider> create --vault <name> --item <title> --fields <key=val,key=val,...> [--category <type>]
```

| Variable | Description | Default |
|----------|-------------|---------|
| `vault`  | Target vault | required |
| `item`   | Item title | required |
| `fields` | Comma-separated `key=value` pairs | required |
| `category` | Item category | `Login` |

Output: the `secret://` reference for the created item.

### set (optional)

Updates a single field of an existing secret.

```bash
aux4 secret <provider> set --ref <vault/item> --field <name> --value <value>
```

| Variable | Description | Default |
|----------|-------------|---------|
| `ref`    | Secret reference path | required |
| `field`  | Field name to update | required |
| `value`  | New value | required |

## Building a Secret Provider

A secret provider is a standard aux4 package that adds commands under the `secret` profile. The package must:

1. Declare `aux4/secret` as a dependency
2. Route `secret` -> `<provider>` -> `secret:<provider>` using profile routing
3. Implement at least the `get` command following the contract above

See the `aux4/secret-1password` package source for a complete provider implementation example.
