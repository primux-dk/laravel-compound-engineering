---
name: doctor
description: Check plugin requirements and install missing dependencies
---

# Doctor Command

Check if all plugin requirements are installed and offer to install missing dependencies.

## Step 1: Check Core Requirements

Run these checks and collect results:

```bash
echo "=== Laravel Compound Engineering - Doctor ==="
echo ""

# PHP
echo -n "PHP 8.3+: "
if command -v php &> /dev/null; then
  PHP_VERSION=$(php -v | head -1 | cut -d' ' -f2)
  echo "✓ $PHP_VERSION"
else
  echo "✗ Not found"
fi

# Composer
echo -n "Composer: "
if command -v composer &> /dev/null; then
  COMPOSER_VERSION=$(composer --version 2>/dev/null | head -1)
  echo "✓ $COMPOSER_VERSION"
else
  echo "✗ Not found"
fi

# Node.js
echo -n "Node.js: "
if command -v node &> /dev/null; then
  NODE_VERSION=$(node --version)
  echo "✓ $NODE_VERSION"
else
  echo "✗ Not found"
fi

# npm
echo -n "npm: "
if command -v npm &> /dev/null; then
  NPM_VERSION=$(npm --version)
  echo "✓ v$NPM_VERSION"
else
  echo "✗ Not found"
fi

# Python
echo -n "Python 3: "
if command -v python3 &> /dev/null; then
  PYTHON_VERSION=$(python3 --version)
  echo "✓ $PYTHON_VERSION"
else
  echo "✗ Not found"
fi

# pip
echo -n "pip: "
if command -v pip3 &> /dev/null; then
  PIP_VERSION=$(pip3 --version 2>/dev/null | cut -d' ' -f2)
  echo "✓ v$PIP_VERSION"
else
  echo "✗ Not found"
fi
```

## Step 2: Check Skill-Specific Requirements

```bash
echo ""
echo "=== Skill Requirements ==="
echo ""

# agent-browser (for browser automation)
echo -n "agent-browser: "
if command -v agent-browser &> /dev/null; then
  echo "✓ Installed"
else
  echo "✗ Not installed (needed for /test-browser, /feature-video)"
fi

# rclone (for cloud uploads)
echo -n "rclone: "
if command -v rclone &> /dev/null; then
  RCLONE_VERSION=$(rclone version 2>/dev/null | head -1)
  echo "✓ $RCLONE_VERSION"
else
  echo "✗ Not installed (needed for rclone skill)"
fi

# Gemini API key
echo -n "GEMINI_API_KEY: "
if [ -n "$GEMINI_API_KEY" ]; then
  echo "✓ Set"
else
  echo "✗ Not set (needed for gemini-imagegen skill)"
fi

# Python packages for gemini-imagegen
echo -n "google-genai (Python): "
if python3 -c "import google.genai" 2>/dev/null; then
  echo "✓ Installed"
else
  echo "✗ Not installed (needed for gemini-imagegen skill)"
fi

echo -n "pillow (Python): "
if python3 -c "from PIL import Image" 2>/dev/null; then
  echo "✓ Installed"
else
  echo "✗ Not installed (needed for gemini-imagegen skill)"
fi
```

## Step 3: Check Laravel Project Requirements (if in Laravel project)

```bash
echo ""
echo "=== Laravel Project (if applicable) ==="
echo ""

if [ -f "composer.json" ] && grep -q "laravel/framework" composer.json 2>/dev/null; then
  echo "Laravel project detected"
  echo ""

  # Laravel Pint
  echo -n "Laravel Pint: "
  if [ -f "vendor/bin/pint" ]; then
    echo "✓ Installed"
  else
    echo "✗ Not installed (run: composer require laravel/pint --dev)"
  fi

  # PEST
  echo -n "PEST: "
  if [ -f "vendor/bin/pest" ]; then
    echo "✓ Installed"
  else
    echo "✗ Not installed (run: composer require pestphp/pest --dev)"
  fi

  # PHPStan
  echo -n "PHPStan: "
  if [ -f "vendor/bin/phpstan" ]; then
    echo "✓ Installed"
  else
    echo "○ Not installed (optional, run: composer require phpstan/phpstan --dev)"
  fi
else
  echo "Not in a Laravel project directory"
fi
```

## Step 4: Summarize and Offer Installation

After running the checks, summarize what's missing and ask the user:

**For missing global tools, offer to install:**

| Tool | Install Command |
|------|-----------------|
| agent-browser | `npm install -g agent-browser` |
| rclone | macOS: `brew install rclone`, Linux: `curl https://rclone.org/install.sh \| sudo bash` |
| google-genai | `pip3 install google-genai` |
| pillow | `pip3 install pillow` |

**For Laravel project tools:**

| Tool | Install Command |
|------|-----------------|
| Laravel Pint | `composer require laravel/pint --dev` |
| PEST | `composer require pestphp/pest --dev --with-all-dependencies` |
| PHPStan | `composer require phpstan/phpstan --dev` |

## Step 5: Interactive Installation

Use the AskUserQuestion tool to ask which missing dependencies to install. Group them logically:

1. **Browser automation** (agent-browser)
2. **Image generation** (google-genai, pillow, GEMINI_API_KEY setup)
3. **Cloud storage** (rclone)
4. **Laravel development** (Pint, PEST, PHPStan)

Run the installation commands for selected items.

## Output Format

```
=== Laravel Compound Engineering - Doctor ===

Core Requirements:
  ✓ PHP 8.3.1
  ✓ Composer 2.7.1
  ✓ Node.js v20.11.0
  ✓ npm v10.2.4
  ✓ Python 3.12.0
  ✓ pip v24.0

Skill Requirements:
  ✗ agent-browser - Not installed
  ✓ rclone - v1.65.0
  ✗ GEMINI_API_KEY - Not set
  ✗ google-genai - Not installed
  ✓ pillow - Installed

Laravel Project:
  ✓ Laravel Pint
  ✓ PEST
  ○ PHPStan (optional)

Missing: agent-browser, GEMINI_API_KEY, google-genai

Would you like to install the missing dependencies?
```
