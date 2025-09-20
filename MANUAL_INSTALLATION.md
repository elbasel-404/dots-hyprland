# Manual Installation Guide for end-4's Hyprland Dotfiles

This guide provides step-by-step instructions to manually install the **illogical-impulse** Hyprland dotfiles without using the automated installation script.

## Prerequisites

- **Arch Linux or Arch-based distribution** (this guide is specifically for Arch-based systems)
- **Internet connection** for downloading packages
- **Basic command line knowledge**
- **Backup your existing configurations** before proceeding

### ⚠️ Important Warnings

1. **This will replace existing configurations**: The installation process will overwrite many configuration files in `~/.config/`. Make sure to backup any important configurations first.

2. **Python 3.12 required**: The setup requires Python 3.12 for proper Pillow library support.

3. **Large download size**: The complete installation includes many packages and can be several GB in size.

4. **GPU compatibility**: While the setup works on most systems, some features may require specific GPU drivers (especially for Hyprland compositing effects).

### Backup Your Configurations

Before proceeding, create backups of your important configurations:

```bash
# Create backup directory
mkdir -p ~/backup-$(date +%Y%m%d)

# Backup important config directories
cp -r ~/.config/hypr ~/backup-$(date +%Y%m%d)/ 2>/dev/null || true
cp -r ~/.config/fish ~/backup-$(date +%Y%m%d)/ 2>/dev/null || true
cp -r ~/.config/kitty ~/backup-$(date +%Y%m%d)/ 2>/dev/null || true
cp ~/.zshrc ~/backup-$(date +%Y%m%d)/ 2>/dev/null || true
cp ~/.bashrc ~/backup-$(date +%Y%m%d)/ 2>/dev/null || true
```

## Table of Contents

1. [System Update and Basic Setup](#1-system-update-and-basic-setup)
2. [Install AUR Helper (yay)](#2-install-aur-helper-yay)
3. [Install Core Dependencies](#3-install-core-dependencies)
4. [Install Meta-packages](#4-install-meta-packages)
5. [Install Python Dependencies](#5-install-python-dependencies)
6. [User Groups and System Configuration](#6-user-groups-and-system-configuration)
7. [Directory Structure Setup](#7-directory-structure-setup)
8. [Configuration Files Installation](#8-configuration-files-installation)
9. [Theme and Desktop Integration](#9-theme-and-desktop-integration)
10. [Final Steps and Verification](#10-final-steps-and-verification)

---

## 1. System Update and Basic Setup

First, ensure your system is up to date:

```bash
sudo pacman -Syu
```

Set up XDG environment variables (add these to your shell profile if not already set):

```bash
export XDG_BIN_HOME=${XDG_BIN_HOME:-$HOME/.local/bin}
export XDG_CACHE_HOME=${XDG_CACHE_HOME:-$HOME/.cache}
export XDG_CONFIG_HOME=${XDG_CONFIG_HOME:-$HOME/.config}
export XDG_DATA_HOME=${XDG_DATA_HOME:-$HOME/.local/share}
export XDG_STATE_HOME=${XDG_STATE_HOME:-$HOME/.local/state}
```

## 2. Install AUR Helper (yay)

If you don't have `yay` installed:

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay-bin.git /tmp/buildyay
cd /tmp/buildyay
makepkg -si
cd -
rm -rf /tmp/buildyay
```

## 3. Install Core Dependencies

### Basic Utilities
```bash
yay -S --needed axel bc coreutils cliphist cmake curl rsync wget ripgrep jq meson xdg-user-dirs
```

### Audio Dependencies
```bash
yay -S --needed cava pavucontrol-qt wireplumber libdbusmenu-gtk3 playerctl
```

### Backlight Control
```bash
yay -S --needed geoclue brightnessctl ddcutil
```

### Hyprland and Related Packages
```bash
yay -S --needed hypridle hyprcursor hyprland hyprland-qtutils hyprland-qt-support hyprlang hyprlock hyprpicker hyprsunset hyprutils hyprwayland-scanner xdg-desktop-portal-hyprland wl-clipboard
```

### Widget System Dependencies
```bash
yay -S --needed fuzzel glib2 hypridle hyprutils hyprlock hyprpicker nm-connection-editor quickshell-git translate-shell wlogout
```

### Fonts and Themes
```bash
yay -S --needed adw-gtk-theme-git breeze breeze-plus darkly-bin eza fish fontconfig kde-material-you-colors kitty matugen-bin otf-space-grotesk starship ttf-gabarito-git ttf-jetbrains-mono-nerd ttf-material-symbols-variable-git ttf-readex-pro ttf-rubik-vf ttf-twemoji
```

### KDE Integration
```bash
yay -S --needed bluedevil gnome-keyring networkmanager plasma-nm polkit-kde-agent dolphin systemsettings
```

### Portal Integration
```bash
yay -S --needed xdg-desktop-portal xdg-desktop-portal-kde xdg-desktop-portal-gtk xdg-desktop-portal-hyprland
```

### Screen Capture Tools
```bash
yay -S --needed hyprshot slurp swappy tesseract tesseract-data-eng wf-recorder
```

### GTK/Qt Toolkit
```bash
yay -S --needed kdialog qt6-5compat qt6-avif-image-plugin qt6-base qt6-declarative qt6-imageformats qt6-multimedia qt6-positioning qt6-quicktimeline qt6-sensors qt6-svg qt6-tools qt6-translations qt6-virtualkeyboard qt6-wayland syntax-highlighting upower wtype ydotool
```

### Python Dependencies
```bash
yay -S --needed clang uv gtk4 libadwaita libsoup3 libportal-gtk4 gobject-introspection sassc python-opencv
```

## 4. Install Meta-packages

Clone the dotfiles repository if you haven't already:

```bash
git clone https://github.com/end-4/dots-hyprland.git
cd dots-hyprland
```

Install the custom meta-packages:

### MicroTeX (LaTeX rendering)
```bash
cd arch-packages/illogical-impulse-microtex-git
makepkg -si
cd ../..
```

### Bibata Cursor Theme (if not already installed)
```bash
if [[ ! -f /usr/share/icons/Bibata-Modern-Classic/index.theme ]]; then
    cd arch-packages/illogical-impulse-bibata-modern-classic-bin
    makepkg -si
    cd ../..
fi
```

### OneUI4 Icons (optional - currently commented out in install script)
```bash
# cd arch-packages/illogical-impulse-oneui4-icons-git
# makepkg -si
# cd ../..
```

## 5. Install Python Dependencies

Install Python packages using uv (with specific versions from requirements):

```bash
# Create virtual environment
uv venv ~/.local/state/quickshell/.venv --prompt .venv -p 3.12

# Activate the virtual environment
source ~/.local/state/quickshell/.venv/bin/activate

# Install packages from the dotfiles requirements
cd dots-hyprland
uv pip install -r scriptdata/requirements.txt

# Deactivate virtual environment
deactivate
```

The main Python dependencies include:
- **build** - Build system interface
- **pillow** - Image processing library  
- **material-color-utilities** - Material Design color generation
- **materialyoucolor** - Material You color theming
- **pycairo** - Cairo graphics bindings
- **pygobject** - GObject bindings for GTK
- **libsass** - Sass CSS preprocessor
- **pywayland** - Wayland protocol bindings
- **click**, **loguru**, **psutil**, **tqdm** - Various utilities

## 6. User Groups and System Configuration

### Add user to required groups:
```bash
sudo usermod -aG video,i2c,input "$(whoami)"
```

### Configure i2c module loading:
```bash
echo "i2c-dev" | sudo tee /etc/modules-load.d/i2c-dev.conf
```

### Enable and start services:
```bash
systemctl --user enable ydotool
systemctl --user start ydotool
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

### Configure GNOME/GTK settings:
```bash
gsettings set org.gnome.desktop.interface font-name 'Rubik 11'
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

### Configure KDE settings:
```bash
kwriteconfig6 --file kdeglobals --group KDE --key widgetStyle Darkly
```

## 7. Directory Structure Setup

Create necessary directories:

```bash
mkdir -p "$XDG_BIN_HOME" "$XDG_CACHE_HOME" "$XDG_CONFIG_HOME" "$XDG_DATA_HOME" "$XDG_STATE_HOME"
```

## 8. Configuration Files Installation

### Copy all configuration files (except fish and hypr):
```bash
cd dots-hyprland

# Copy all config directories except fish and hypr
for dir in .config/*/; do
    dirname=$(basename "$dir")
    if [[ "$dirname" != "fish" && "$dirname" != "hypr" ]]; then
        echo "Copying .config/$dirname"
        if [[ -d ".config/$dirname" ]]; then
            rsync -av --delete ".config/$dirname/" "$XDG_CONFIG_HOME/$dirname/"
        elif [[ -f ".config/$dirname" ]]; then
            rsync -av ".config/$dirname" "$XDG_CONFIG_HOME/$dirname"
        fi
    fi
done
```

### Copy Fish configuration:
```bash
rsync -av --delete .config/fish/ "$XDG_CONFIG_HOME/fish/"
```

### Copy Hyprland configuration:
```bash
# Copy all hypr config except certain files
rsync -av --delete --exclude '/custom' --exclude '/hyprlock.conf' --exclude '/hypridle.conf' --exclude '/hyprland.conf' .config/hypr/ "$XDG_CONFIG_HOME/hypr/"

# Handle main Hyprland config
if [[ -f "$XDG_CONFIG_HOME/hypr/hyprland.conf" ]]; then
    echo "Backing up existing hyprland.conf"
    mv "$XDG_CONFIG_HOME/hypr/hyprland.conf" "$XDG_CONFIG_HOME/hypr/hyprland.conf.old"
fi
cp .config/hypr/hyprland.conf "$XDG_CONFIG_HOME/hypr/hyprland.conf"

# Handle hypridle config
if [[ -f "$XDG_CONFIG_HOME/hypr/hypridle.conf" ]]; then
    echo "Creating hypridle.conf.new (existing config preserved)"
    cp .config/hypr/hypridle.conf "$XDG_CONFIG_HOME/hypr/hypridle.conf.new"
else
    cp .config/hypr/hypridle.conf "$XDG_CONFIG_HOME/hypr/hypridle.conf"
fi

# Handle hyprlock config
if [[ -f "$XDG_CONFIG_HOME/hypr/hyprlock.conf" ]]; then
    echo "Creating hyprlock.conf.new (existing config preserved)"
    cp .config/hypr/hyprlock.conf "$XDG_CONFIG_HOME/hypr/hyprlock.conf.new"
else
    cp .config/hypr/hyprlock.conf "$XDG_CONFIG_HOME/hypr/hyprlock.conf"
fi

# Handle custom config directory
if [[ ! -d "$XDG_CONFIG_HOME/hypr/custom" ]]; then
    rsync -av --delete .config/hypr/custom/ "$XDG_CONFIG_HOME/hypr/custom/"
fi
```

### Copy local data files:
```bash
# Copy icons
rsync -av .local/share/icons/ "$XDG_DATA_HOME/icons/"

# Copy konsole profiles
rsync -av .local/share/konsole/ "$XDG_DATA_HOME/konsole/"
```

## 9. Theme and Desktop Integration

### Optional: Install plasma-browser-integration (for Firefox media controls)
```bash
# This package is ~600MB, install only if you want Firefox media controls
sudo pacman -S --needed plasma-browser-integration
```

### Set up virtual environment for Quickshell
Make sure the Python virtual environment is properly configured. The environment variable `ILLOGICAL_IMPULSE_VIRTUAL_ENV` should be set to `~/.local/state/quickshell/.venv` (this is configured in the Hyprland config files).

## 10. Final Steps and Verification

### Make scripts executable:
```bash
cd dots-hyprland
find .config/quickshell/ii/scripts -name "*.sh" -type f -exec chmod +x {} \;
```

### Refresh font cache:
```bash
fc-cache -fv
```

### Set up Zsh integration (if using Zsh):
If you use Zsh, add this line to your `~/.zshrc`:
```bash
source ${XDG_CONFIG_HOME:-~/.config}/zshrc.d/dots-hyprland.zsh
```

### Reboot or Re-login:
```bash
# Reboot to ensure all changes take effect
sudo reboot
```

### Start Hyprland:
When logging in, select **Hyprland** (NOT "Hyprland with UWSM") from your display manager.

### Initial Setup After Login:
1. **Select a wallpaper**: Press `Ctrl+Super+T`
2. **View keybinds**: Press `Super+/`
3. **Open terminal**: Press `Super+Enter`
4. **Test Quickshell**: The status bar and widgets should be visible immediately

## Important Notes

### Environment Variables
The dotfiles require the `ILLOGICAL_IMPULSE_VIRTUAL_ENV` environment variable to be set. This should be automatically configured through the Hyprland config, but if you encounter issues, ensure it's set to: `~/.local/state/quickshell/.venv`

### Configuration Conflicts
If you had existing configurations:
- `hyprland.conf.old` - Your old Hyprland config (if it existed)
- `hypridle.conf.new` - New hypridle config (compare with your existing one)
- `hyprlock.conf.new` - New hyprlock config (compare with your existing one)

### Quickshell Development
For Quickshell development and testing:
1. Open `~/.config/quickshell/ii` in your code editor
2. Run `pkill qs; qs -c ii` in terminal to start Quickshell with logs
3. Changes are reloaded live

## Troubleshooting

### If Quickshell doesn't start:
1. Check that `quickshell-git` is installed: `yay -Q quickshell-git`
2. Verify Python virtual environment: `ls ~/.local/state/quickshell/.venv/`
3. Check Quickshell config: `qs -c ii` in terminal for error messages
4. Ensure the virtual environment has all required packages:
   ```bash
   source ~/.local/state/quickshell/.venv/bin/activate
   python -c "import material_color_utilities, PIL, cairo; print('All imports successful')"
   deactivate
   ```

### If themes don't apply:
1. Verify Qt/GTK theme packages are installed
2. Check KDE settings: `systemsettings5`
3. Restart applications or re-login
4. Verify Kvantum theme: `kvantummanager`

### If hotkeys don't work:
1. Check Hyprland config: `~/.config/hypr/hyprland.conf`
2. Verify `hyprland.conf` includes all necessary config files
3. Reload Hyprland: `hyprctl reload`
4. Check for conflicting keybinds in other applications

### Permission errors:
1. Ensure user is in correct groups: `groups $(whoami)`
2. Check service status: `systemctl --user status ydotool`
3. Verify i2c module loading: `lsmod | grep i2c_dev`

### Performance issues:
1. Check GPU drivers are properly installed
2. Verify Hyprland is using hardware acceleration: `hyprctl systeminfo`
3. Reduce animation settings if needed in Hyprland config
4. Monitor system resources: `htop`

### If packages fail to install:
1. Update package databases: `yay -Sy`
2. Clear package cache: `yay -Yc`
3. Install problematic packages individually
4. Check AUR package build logs for specific errors

### Font rendering issues:
1. Refresh font cache: `fc-cache -fv`
2. Verify font installation: `fc-list | grep -i rubik`
3. Check fontconfig settings: `~/.config/fontconfig/fonts.conf`

### Screen capture not working:
1. Verify screen capture tools: `which hyprshot slurp swappy`
2. Check Hyprland screen sharing: `hyprctl monitors`
3. Test individual tools: `slurp` (should show selection overlay)

## Post-Installation Resources

- **Wiki**: https://end-4.github.io/dots-hyprland-wiki/en/ii-qs/01setup/#post-installation
- **Keybinds**: Press `Super+/` in Hyprland
- **Discord**: https://discord.gg/GtdRBXgMwq for community support
- **Issues**: https://github.com/end-4/dots-hyprland/issues for bug reports

## What This Setup Includes

- **Hyprland**: Wayland compositor with custom configuration
- **Quickshell**: Modern widget system for status bar and sidebars
- **Fish**: User-friendly shell with custom configuration
- **Kitty**: GPU-accelerated terminal emulator
- **Fuzzel**: Application launcher
- **Material Design**: Automatic color generation from wallpapers
- **AI Integration**: Gemini API and Ollama support
- **Media Controls**: Integration with audio/video players
- **Screen Capture**: Screenshot and recording tools
- **Custom Themes**: OneUI and Material Design inspired themes

## Customization

### Custom Hyprland Configuration
Your personal customizations should go in `~/.config/hypr/custom/`:
- `custom/keybinds.conf` - Additional keybindings
- `custom/rules.conf` - Window rules
- `custom/execs.conf` - Startup applications
- `custom/general.conf` - General settings
- `custom/env.conf` - Environment variables

### Quickshell Customization
- **Main config**: `~/.config/quickshell/ii/`
- **Widgets**: `~/.config/quickshell/ii/modules/`
- **Themes**: Colors are automatically generated from wallpapers
- **Scripts**: `~/.config/quickshell/ii/scripts/`

### Wallpaper and Colors
- **Change wallpaper**: `Ctrl+Super+T` or use the settings panel
- **Custom wallpapers**: Place images in your Pictures directory
- **Color generation**: Automatic Material Design colors from wallpaper
- **Manual color override**: Edit generated color files in `~/.local/state/quickshell/user/generated/`

### AI Configuration
1. **Gemini API**: Configure API key in Quickshell settings
2. **Ollama**: Install ollama and download models for local AI
3. **Custom prompts**: Add prompts to `~/.config/illogical-impulse/ai/prompts/`

### Fish Shell Customization
- **Config**: `~/.config/fish/config.fish`
- **Functions**: `~/.config/fish/functions/`
- **Completions**: Fish completions are automatically generated
- **Starship prompt**: Configure in `~/.config/starship.toml`

Enjoy your new Hyprland setup! 🎉
