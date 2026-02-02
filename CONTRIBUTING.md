# Contributing to XposedOrNot Sentinel

First off, thank you for considering contributing to XposedOrNot Sentinel! 🎉

## How Can I Contribute?

### Reporting Bugs

- Check if the bug has already been reported in [Issues](https://github.com/XposedOrNot/XposedOrNot-Sentinel/issues)
- If not, create a new issue with:
  - Clear title and description
  - Steps to reproduce
  - Expected vs actual behavior
  - Azure/Sentinel version info

### Suggesting Features

- Open an issue with the `enhancement` label
- Describe the feature and its use case
- Explain why it would be useful

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test your changes in an Azure environment
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## Development Guidelines

### ARM Templates

- Validate templates with `az deployment group validate`
- Use parameters for configurable values
- Include descriptions in parameter metadata
- Follow Azure naming conventions

### KQL Queries

- Test queries in Log Analytics before adding to rules/workbooks
- Use comments to explain complex logic
- Optimize for performance (avoid full table scans)

### Documentation

- Update README.md if adding new features
- Include examples for new functionality
- Keep language clear and concise

## Code of Conduct

Be respectful, inclusive, and constructive. We're all here to make security better for everyone.

## Questions?

Feel free to open an issue or reach out to the maintainers.

---

Thank you for contributing! 🙏
