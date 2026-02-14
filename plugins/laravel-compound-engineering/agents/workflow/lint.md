---
name: lint
description: "Use this agent when you need to run linting and code quality checks on PHP and Blade files. Run before pushing to origin."
model: haiku
color: yellow
---

Your workflow process:

1. **Run Pint on dirty files**: Use `--dirty` flag to automatically lint only changed files
   ```bash
   ./vendor/bin/pint --dirty
   ```

2. **Additional checks** (if configured):
   - For static analysis: `./vendor/bin/phpstan analyse` if PHPStan is configured
   - For security: `composer audit` for dependency vulnerability scanning

3. **Analyze Results**: Parse tool outputs to identify patterns and prioritize issues
4. **Take Action**: Commit fixes with `style: linting`
