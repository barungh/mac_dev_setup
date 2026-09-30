# mac_dev_setup

Here is a complete, copy-paste-ready Markdown guide formatted for a GitHub Gist or documentation file.

---

```markdown
# Fresh macOS Setup Guide: Python, `uv`, and LazyVim

A minimal, high-performance developer environment setup for macOS (Apple Silicon & Intel) using **uv** for Python tooling and **Neovim (LazyVim)** as the primary editor.

---

## 1. System Essentials & Xcode Command Line Tools

Open the built-in **Terminal** and install the macOS Developer Tools (compiler tools, headers, and basic Git):

```bash
xcode-select --install
```

Follow the GUI prompt to complete the installation.

---

## 2. Homebrew (macOS Package Manager)

Install Homebrew to manage CLI utilities and graphical applications:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to your shell environment (critical for Apple Silicon Macs):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

*(If you are on an Intel Mac, replace `/opt/homebrew` with `/usr/local` if prompted).*

---

## 3. Terminal & Nerd Font

LazyVim requires a **Nerd Font** to render file tree icons, diagnostics, and statusline symbols correctly.

### 3.1 Install a Nerd Font
```bash
brew install --cask font-jetbrains-mono-nerd-font
```

### 3.2 Choose a Modern Terminal (Optional but Recommended)
While macOS Terminal works, modern GPU-accelerated terminal emulators offer a much smoother Neovim experience:

```bash
# Options: Ghostty (recommended), WezTerm, or iTerm2
brew install --cask ghostty
# OR
brew install --cask iterm2
```

> **Important**: Open your terminal's settings (e.g., Ghostty/iTerm2 Preferences -> Profiles -> Text) and set the font to **JetBrainsMono Nerd Font**.

---

## 4. LazyVim Prerequisites & Dependencies

LazyVim relies on Neovim (>= 0.10.0) and several modern CLI tools for fuzzy-finding, grep, and Git workflows:

```bash
brew install \
  neovim \
  ripgrep \
  fd \
  fzf \
  git \
  lazygit
```

Set up your basic Git identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
```

---

## 5. Install & Setup `uv`

[`uv`](https://github.com/astral-sh/uv) replaces `pyenv`, `pip`, `pip-tools`, `poetry`, and `virtualenv`. It can manage Python runtimes, create isolated environments, and run scripts at blazingly fast speeds.

### 5.1 Install `uv`
```bash
brew install uv
```
*(Or use Astral's standalone installer: `curl -LsSf https://astral.sh/uv/install.sh | sh`)*

### 5.2 Install Python Runtimes
You do not need Homebrew-installed Python or `pyenv`. Let `uv` manage standalone Python versions:

```bash
# Install the latest Python version
uv python install

# Or install specific versions
uv python install 3.12 3.11
```

### 5.3 Shell Autocompletion
Add `uv` autocomplete to your `.zshrc`:

```bash
echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
source ~/.zshrc
```

---

## 6. Install LazyVim

LazyVim provides a structured, modular starter configuration for Neovim.

### 6.1 Back up existing Neovim configs (if any)
```bash
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null
mv ~/.local/share/nvim ~/.local/share/nvim.bak 2>/dev/null
mv ~/.local/state/nvim ~/.local/state/nvim.bak 2>/dev/null
mv ~/.cache/nvim ~/.cache/nvim.bak 2>/dev/null
```

### 6.2 Clone the LazyVim Starter
```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

### 6.3 First Launch
Launch Neovim to trigger automatic plugin installation:

```bash
nvim
```

Lazy will download and compile all default plugins. When finished, press `q` to exit.

---

## 7. Configure LazyVim for Python

LazyVim includes built-in "Extras" for Python, enabling LSP (`pyright` or `basedpyright`), linting/formatting (`ruff`), and debugging (`debugpy`).

### 7.1 Enable the Python Extra
Open `~/.config/nvim/lua/config/lazy.lua` and locate the `spec` table:

```lua
  spec = {
    -- add LazyVim and import its plugins
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },
    -- import/override with your plugins
    { import = "lazyvim.plugins.extras.lang.python" }, -- <--- ADD THIS LINE
    { import = "plugins" },
  },
```

*(Alternatively, inside Neovim press `<Space>ux` to open LazyExtras, search for `lang.python`, and press `x` to toggle it).*

### 7.2 (Optional) Make Ruff the Primary Formatter
LazyVim automatically pairs Ruff with Pyright. If you want Ruff to format on save, ensure your `~/.config/nvim/lua/plugins/formatting.lua` exists with:

```lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        python = { "ruff_format", "ruff_organize_imports" },
      },
    },
  },
}
```

---

## 8. Recommended Daily Workflow

Here is how `uv` and LazyVim work together in practice:

### 8.1 Start a New Project
```bash
mkdir my-python-project
cd my-python-project

# Initialize a new uv managed project
uv init

# Create and activate a local virtualenv
uv venv
source .venv/bin/activate

# Add packages (e.g., fastapi, pydantic)
uv add fastapi pydantic
```

### 8.2 Open in LazyVim
```bash
nvim .
```

* **LSP Resolution**: LazyVim automatically discovers the `.venv` folder in your project root. Autocompletion and type-checking will resolve your installed packages immediately.
* **Format on Save**: Enabled out-of-the-box (`<Space>cf` to manually format).
* **Code Actions**: Press `<Space>ca` to trigger Ruff fixes or import cleanups.
* **Terminal**: Press `<Ctrl-/>` inside Neovim to toggle an embedded terminal running your virtual environment.

### 8.3 Run and Test
```bash
# Run code directly via uv
uv run main.py

# Run tests
uv add --dev pytest
uv run pytest
```
```
