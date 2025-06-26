# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| Latest  | :white_check_mark: |

## Reporting a Vulnerability

If you discover a security vulnerability, please report it by:

1. **Email**: Create an issue with the "security" label
2. **Response Time**: We aim to respond within 48 hours
3. **Process**: We will investigate and provide updates on the resolution timeline

## Security Best Practices

When using this environment setup:

- **Never commit credentials** - Use environment variables
- **GCP Authentication** - Use `gcloud auth application-default login`
- **Project IDs** - Set via `export PROJECT_ID="your-project"`
- **Dependencies** - Keep packages updated via `mamba update --all`

## Security Features

This repository includes:
- Vulnerability alerts enabled
- Dependabot security updates
- Regular security scanning