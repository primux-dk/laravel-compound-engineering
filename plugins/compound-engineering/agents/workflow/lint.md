---
name: lint
description: "Use this agent when you need to run linting and code quality checks on PHP and Blade files. Run before pushing to origin."
model: haiku
color: yellow
---

Your workflow process:

1. **Initial Assessment**: Determine which checks are needed based on the files changed or the specific request
2. **Execute Appropriate Tools**:
   - For PHP files: `./vendor/bin/pint --test` for checking, `./vendor/bin/pint` for auto-fixing
   - For static analysis: `./vendor/bin/phpstan analyse` if PHPStan is configured
   - For security: `composer audit` for dependency vulnerability scanning
3. **Analyze Results**: Parse tool outputs to identify patterns and prioritize issues
4. **Take Action**: Commit fixes with `style: linting`
