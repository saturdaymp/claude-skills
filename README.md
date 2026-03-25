# SaturdayMP Claude Plugins

[![Sponsor](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=fe8e86)](https://github.com/sponsors/saturdaymp)

A Claude Code Marketplace plugins that Saturday Morning Productions finds useful and hope you do as well.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed

## Installation

### From GitHub

Add this repo as a plugin marketplace:

```
/plugin marketplace add saturdaymp/claude-plugins
```

### From a local clone

Clone the repo:

```bash
git clone https://github.com/saturdaymp/claude-plugins.git
```

Load the local marketplace:

```
/plugin marketplace add /path/to/cloned/repo
```

## Plugins

Install plugins:

```
/plugin install smp-apply-github-pr-feedback@saturdaymp/claude-plugins
```

Don't forget to restart Claude Code or try reloading the plugins:

```
/reload-plugins
```

### Plugins Available

| Plugin | Description |
|--------|-------------|
| [smp-github](plugins/smp-github/) | GitHub skills — PR review feedback workflow |

## License

[MIT](LICENSE)