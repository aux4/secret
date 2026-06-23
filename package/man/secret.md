#### Description

The `secret` command is the entry point for all secret provider packages. It routes to installed provider subcommands like `1password`.

aux4 resolves `secret://` URIs automatically in variable defaults and `set:` instructions. The URI format is:

```text
secret://<provider>/<vault>/<item>/<field>
```

Install a provider to use secrets:

```bash
aux4 aux4 pkger install aux4/secret-1password
```

#### Usage

```bash
aux4 secret <provider> <command> [options]
```

#### Example

List available secrets:

```bash
aux4 secret 1password list --vault Personal
```

```text
secret://1password/Personal/GitHub
secret://1password/Personal/Gmail
```

Use a secret in a command definition:

```json
{
  "name": "apiToken",
  "default": "secret://1password/Work/deploy-service/credential"
}
```
