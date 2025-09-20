# Manual Installation Guide for Hyprland Dotfiles

This guide provides step-by-step instructions to manually install the illogical-impulse Hyprland dotfiles setup without using any scripts. This setup uses **Quickshell** (not AGS) for the widgets and UI components.

## Prerequisites

- Arch Linux or Arch-based distribution
- Basic terminal knowledge
- Internet connection

## Overview

This setup includes:
- **Hyprland**: Wayland compositor
- **Quickshell**: QtQuick-based widget system for UI components
- **Custom fonts and themes**: Material Design-based theming
- **Audio/media controls**: PipeWire/WirePlumber integration
- **System integrations**: Notifications, screen capture, etc.

---

## Phase 1: System Preparation

### 1.1 Update System
```bash
sudo pacman -Syu
```

### 1.2 Install Base Development Tools
```bash
sudo pacman -S --needed base-devel git
```

### 1.3 Install AUR Helper (yay)
```bash
git clone https://aur.archlinux.org/yay-bin.git /tmp/buildyay
cd /tmp/buildyay
makepkg -si --noconfirm
cd ~
rm -rf /tmp/buildyay
```

### 1.4 Set Environment Variables
Create directories that will be used:
```bash
export XDG_BIN_HOME=${XDG_BIN_HOME:-$HOME/.local/bin}
export XDG_CACHE_HOME=${XDG_CACHE_HOME:-$HOME/.cache}
export XDG_CONFIG_HOME=${XDG_CONFIG_HOME:-$HOME/.config}
export XDG_DATA_HOME=${XDG_DATA_HOME:-$HOME/.local/share}
export XDG_STATE_HOME=${XDG_STATE_HOME:-$HOME/.local/state}

mkdir -p "$XDG_BIN_HOME" "$XDG_CACHE_HOME" "$XDG_CONFIG_HOME" "$XDG_DATA_HOME" "$XDG_STATE_HOME"
```

### 1.5 Create Backup (Optional but Recommended)
```bash
# Backup existing configs
BACKUP_DIR="$HOME/backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"
cp -r "$XDG_CONFIG_HOME" "$BACKUP_DIR/config" 2>/dev/null || true
cp -r "$HOME/.local" "$BACKUP_DIR/local" 2>/dev/null || true
echo "Backup created at: $BACKUP_DIR"
```

---

## Phase 2: Install Core Dependencies

### 2.1 Hyprland and Related Packages
```bash
sudo pacman -S --needed --noconfirm \
    hypridle \
    hyprcursor \
    hyprland \
    hyprland-qtutils \
    hyprland-qt-support \
    hyprlang \
    hyprlock \
    hyprpicker \
    hyprsunset \
    hyprutils \
    hyprwayland-scanner \
    xdg-desktop-portal-hyprland \
    wl-clipboard
```

### 2.2 Widget System (Quickshell)
```bash
yay -S --needed --noconfirm quickshell-git
```

### 2.3 Basic Utilities
```bash
sudo pacman -S --needed --noconfirm \
    axel \
    bc \
    coreutils \
    cliphist \
    cmake \
    curl \
    rsync \
    wget \
    ripgrep \
    jq \
    meson \
    xdg-user-dirs
```

### 2.4 Audio System
```bash
sudo pacman -S --needed --noconfirm \
    cava \
    pavucontrol-qt \
    wireplumber \
    libdbusmenu-gtk3 \
    playerctl
```

### 2.5 Backlight Control
```bash
sudo pacman -S --needed --noconfirm \
    geoclue \
    brightnessctl \
    ddcutil
```

### 2.6 Screenshot and Recording
```bash
yay -S --needed --noconfirm \
    hyprshot \
    slurp \
    swappy \
    tesseract \
    tesseract-data-eng \
    wf-recorder
```

### 2.7 System Integration
```bash
sudo pacman -S --needed --noconfirm \
    fuzzel \
    glib2 \
    nm-connection-editor \
    translate-shell \
    wlogout
```

### 2.8 Desktop Portals
```bash
sudo pacman -S --needed --noconfirm \
    xdg-desktop-portal \
    xdg-desktop-portal-kde \
    xdg-desktop-portal-gtk \
    xdg-desktop-portal-hyprland
```

### 2.9 KDE Integration
```bash
sudo pacman -S --needed --noconfirm \
    bluedevil \
    gnome-keyring \
    networkmanager \
    plasma-nm \
    polkit-kde-agent \
    dolphin \
    systemsettings
```

### 2.10 Qt/GTK Toolkit Dependencies
```bash
sudo pacman -S --needed --noconfirm \
    kdialog \
    qt6-5compat \
    qt6-avif-image-plugin \
    qt6-base \
    qt6-declarative \
    qt6-imageformats \
    qt6-multimedia \
    qt6-positioning \
    qt6-quicktimeline \
    qt6-sensors \
    qt6-svg \
    qt6-tools \
    qt6-translations \
    qt6-virtualkeyboard \
    qt6-wayland \
    syntax-highlighting \
    upower \
    wtype \
    ydotool
```

---

## Phase 3: Install Fonts and Themes

### 3.1 Install Fonts via Package Manager
```bash
yay -S --needed --noconfirm \
    otf-space-grotesk \
    ttf-jetbrains-mono-nerd \
    ttf-material-symbols-variable-git \
    ttf-readex-pro \
    ttf-rubik-vf \
    ttf-twemoji \
    ttf-gabarito-git
```

### 3.2 Install Rubik Font (Manual Method if needed)
```bash
mkdir -p /tmp/fonts/Rubik
cd /tmp/fonts/Rubik
git clone https://github.com/googlefonts/rubik.git .
sudo mkdir -p /usr/local/share/fonts/TTF/
sudo cp fonts/variable/Rubik*.ttf /usr/local/share/fonts/TTF/
sudo mkdir -p /usr/local/share/licenses/ttf-rubik/
sudo cp OFL.txt /usr/local/share/licenses/ttf-rubik/LICENSE
fc-cache -fv
```

### 3.3 Install Gabarito Font (Manual Method if needed)
```bash
mkdir -p /tmp/fonts/Gabarito
cd /tmp/fonts/Gabarito
git clone https://github.com/naipefoundry/gabarito.git .
sudo mkdir -p /usr/local/share/fonts/TTF/
sudo cp fonts/ttf/Gabarito*.ttf /usr/local/share/fonts/TTF/
sudo mkdir -p /usr/local/share/licenses/ttf-gabarito/
sudo cp OFL.txt /usr/local/share/licenses/ttf-gabarito/LICENSE
fc-cache -fv
```

### 3.4 Install Themes
```bash
yay -S --needed --noconfirm \
    adw-gtk-theme-git \
    breeze \
    breeze-plus \
    darkly-bin \
    kde-material-you-colors \
    matugen-bin
```

### 3.5 Install OneUI Icons
```bash
yay -S --needed --noconfirm illogical-impulse-oneui4-icons-git
```

Or manually:
```bash
mkdir -p /tmp/icons/OneUI4-Icons
cd /tmp/icons/OneUI4-Icons
git clone https://github.com/end-4/OneUI4-Icons.git .
sudo mkdir -p /usr/local/share/icons
sudo cp -r OneUI /usr/local/share/icons
sudo cp -r OneUI-dark /usr/local/share/icons
sudo cp -r OneUI-light /usr/local/share/icons
```

### 3.6 Install Bibata Cursor Theme
```bash
yay -S --needed --noconfirm illogical-impulse-bibata-modern-classic-bin
```

Or manually:
```bash
mkdir -p /tmp/cursors/bibata-cursor
cd /tmp/cursors/bibata-cursor
CURSOR_VERSION="2.0.6"
axel "https://github.com/ful1e5/Bibata_Cursor/releases/download/v${CURSOR_VERSION}/Bibata-Modern-Classic.tar.xz"
tar -xf Bibata-Modern-Classic.tar.xz
sudo mkdir -p /usr/local/share/icons
sudo cp -r Bibata-Modern-Classic /usr/local/share/icons
```

### 3.7 Terminal and Shell Tools
```bash
sudo pacman -S --needed --noconfirm \
    eza \
    fish \
    fontconfig \
    kitty \
    starship
```

---

## Phase 4: Python Environment and MicroTeX

### 4.1 Install Python Dependencies
```bash
sudo pacman -S --needed --noconfirm \
    clang \
    gtk4 \
    libadwaita \
    libsoup3 \
    libportal-gtk4 \
    gobject-introspection \
    sassc \
    python-opencv
```

### 4.2 Install uv (Python Package Manager)
```bash
curl -LJs "https://astral.sh/uv/install.sh" | bash
# Add to PATH if needed
export PATH="$HOME/.local/bin:$PATH"
```

### 4.3 Create Python Virtual Environment
```bash
export ILLOGICAL_IMPULSE_VIRTUAL_ENV="$XDG_STATE_HOME/quickshell/.venv"
mkdir -p "$(dirname "$ILLOGICAL_IMPULSE_VIRTUAL_ENV")"

# Clone the repo temporarily to get requirements
git clone https://github.com/elbasel-404/dots-hyprland.git /tmp/dots-hyprland
cd /tmp/dots-hyprland

# Create virtual environment and install Python packages
uv venv --prompt .venv "$ILLOGICAL_IMPULSE_VIRTUAL_ENV" -p 3.12
source "$ILLOGICAL_IMPULSE_VIRTUAL_ENV/bin/activate"
uv pip install -r scriptdata/requirements.txt
deactivate
```

### 4.4 Install MicroTeX (Optional - for LaTeX rendering)
```bash
yay -S --needed --noconfirm illogical-impulse-microtex-git
```

Or manually:
```bash
sudo pacman -S --needed --noconfirm tinyxml2 gtkmm3 gtksourceviewmm cairomm
mkdir -p /tmp/MicroTeX
cd /tmp/MicroTeX
git clone https://github.com/NanoMichael/MicroTeX.git .
mkdir -p build
cd build
cmake ..
make -j$(nproc)
sudo mkdir -p /opt/MicroTeX
sudo cp ./LaTeX /opt/MicroTeX/
sudo cp -r ./res /opt/MicroTeX/
```

---

## Phase 5: Configure System Settings

### 5.1 Add User to Required Groups
```bash
sudo usermod -aG video,i2c,input "$(whoami)"
```

### 5.2 Enable System Services
```bash
# Enable i2c-dev module for brightness control
echo i2c-dev | sudo tee /etc/modules-load.d/i2c-dev.conf

# Enable ydotool for automated input
systemctl --user enable ydotool --now

# Enable Bluetooth
sudo systemctl enable bluetooth --now
```

### 5.3 Configure Font and Theme Settings
```bash
# Set default font
gsettings set org.gnome.desktop.interface font-name 'Rubik 11'

# Set dark theme preference
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

# Configure KDE theme
kwriteconfig6 --file kdeglobals --group KDE --key widgetStyle Darkly
```

### 5.4 Configure XDG User Directories
```bash
xdg-user-dirs-update
```

---

## Phase 6: Install Configuration Files

### 6.1 Clone the Dotfiles Repository
```bash
cd /tmp
git clone https://github.com/elbasel-404/dots-hyprland.git
cd dots-hyprland
```

### 6.2 Copy Hyprland Configuration
```bash
# Copy Hyprland configs (excluding custom directory)
rsync -av --delete --exclude '/custom' --exclude '/hyprlock.conf' --exclude '/hypridle.conf' --exclude '/hyprland.conf' .config/hypr/ "$XDG_CONFIG_HOME"/hypr/

# Handle main config files
if [ -f "$XDG_CONFIG_HOME/hypr/hyprland.conf" ]; then
    echo "Backing up existing hyprland.conf"
    mv "$XDG_CONFIG_HOME/hypr/hyprland.conf" "$XDG_CONFIG_HOME/hypr/hyprland.conf.old"
fi
cp .config/hypr/hyprland.conf "$XDG_CONFIG_HOME/hypr/hyprland.conf"

# Handle hypridle config
if [ -f "$XDG_CONFIG_HOME/hypr/hypridle.conf" ]; then
    echo "Backing up existing hypridle.conf"
    mv "$XDG_CONFIG_HOME/hypr/hypridle.conf" "$XDG_CONFIG_HOME/hypr/hypridle.conf.old"
fi
cp .config/hypr/hypridle.conf "$XDG_CONFIG_HOME/hypr/hypridle.conf"

# Handle hyprlock config
if [ -f "$XDG_CONFIG_HOME/hypr/hyprlock.conf" ]; then
    echo "Backing up existing hyprlock.conf"
    mv "$XDG_CONFIG_HOME/hypr/hyprlock.conf" "$XDG_CONFIG_HOME/hypr/hyprlock.conf.old"
fi
cp .config/hypr/hyprlock.conf "$XDG_CONFIG_HOME/hypr/hyprlock.conf"

# Copy custom directory if it doesn't exist
if [ ! -d "$XDG_CONFIG_HOME/hypr/custom" ]; then
    rsync -av --delete .config/hypr/custom/ "$XDG_CONFIG_HOME/hypr/custom/"
fi
```

### 6.3 Copy Quickshell Configuration
```bash
rsync -av --delete .config/quickshell/ "$XDG_CONFIG_HOME"/quickshell/
```

### 6.4 Copy Other Configuration Files
```bash
# Skip fish and handle miscellaneous configs
rsync -av --delete --exclude 'fish' --exclude 'hypr' --exclude 'quickshell' .config/ "$XDG_CONFIG_HOME"/
```

### 6.5 Copy Local Data Files
```bash
rsync -av .local/share/icons/ "$XDG_DATA_HOME"/icons/
rsync -av .local/share/konsole/ "$XDG_DATA_HOME"/konsole/
```

### 6.6 Setup Fish Shell Configuration (Optional)
```bash
# Only if you want to use fish shell
if command -v fish >/dev/null 2>&1; then
    rsync -av .config/fish/ "$XDG_CONFIG_HOME"/fish/
fi
```

### 6.7 Setup ZSH Configuration (Optional)
```bash
# Setup zsh config if using zsh
if [ -f ~/.zshrc ]; then
    if ! grep -q 'source ${XDG_CONFIG_HOME:-~/.config}/zshrc.d/dots-hyprland.zsh' ~/.zshrc; then
        echo 'source ${XDG_CONFIG_HOME:-~/.config}/zshrc.d/dots-hyprland.zsh' >> ~/.zshrc
    fi
fi
```

---

## Phase 7: Final Configuration

### 7.1 Setup Quickshell LSP Support
```bash
touch "$XDG_CONFIG_HOME/quickshell/ii/.qmlls.ini"
```

### 7.2 Configure Plasma Browser Integration (Optional)
```bash
# Install if you want media controls to work with Firefox
sudo pacman -S --needed plasma-browser-integration
```

### 7.3 Set Environment Variables for Hyprland
```bash
# The environment variables are already set in the Hyprland config
# Make sure the virtual environment path is correctly set
echo "export ILLOGICAL_IMPULSE_VIRTUAL_ENV=\"$ILLOGICAL_IMPULSE_VIRTUAL_ENV\"" >> ~/.profile
```

---

## Phase 8: Launch and Verify

### 8.1 Logout and Login to Hyprland
1. Logout from your current session
2. Select "Hyprland" from your login manager (not "uwsm-managed Hyprland")
3. Login

### 8.2 Start Quickshell
Once in Hyprland, the Quickshell should start automatically. If not:
```bash
pkill qs; qs -c ii
```

### 8.3 Verify Installation
- Press `Super + /` to open the keybind cheat sheet
- Press `Super + Enter` to open terminal
- Try `Super + V` for clipboard history
- Test `Super + A` for application launcher

---

## Important Keybinds

### Shell/UI Controls
| Keybind | Action |
|---------|--------|
| `Super` (tap) | Toggle overview/launcher |
| `Super + /` | Toggle keybind cheat sheet |
| `Super + V` | Clipboard history |
| `Super + .` (Period) | Emoji picker |
| `Super + A` | Toggle left sidebar |
| `Super + N` | Toggle right sidebar |
| `Super + K` | Toggle on-screen keyboard |
| `Super + M` | Toggle media controls |
| `Super + J` | Toggle bar |
| `Ctrl + Alt + Delete` | Session menu (logout/reboot/shutdown) |

### Window Management
| Keybind | Action |
|---------|--------|
| `Super + Enter` | Terminal |
| `Super + E` | File manager |
| `Super + W` | Browser |
| `Super + C` | Code editor |
| `Super + Q` | Close window |
| `Super + F` | Fullscreen |
| `Super + P` | Pin window |
| `Alt + F` | Fullscreen spoof |
| `Super + D` | Maximize |
| `Alt + Space` | Float/Tile toggle |

### Workspace Management  
| Keybind | Action |
|---------|--------|
| `Super + [1-9]` | Focus workspace # |
| `Super + Shift + [1-9]` | Send to workspace # |
| `Super + Alt + [1-9]` | Send to workspace # (silent) |
| `Super + S` | Toggle scratchpad |
| `Super + Page_Up/Page_Down` | Focus left/right workspace |
| `Ctrl + Super + [←/→]` | Focus left/right workspace |
| `Super + Shift + Page_Up/Page_Down` | Send to left/right workspace |

### Window Movement & Resizing
| Keybind | Action |
|---------|--------|
| `Super + [←/↑/↓/→]` | Move focus |
| `Super + Shift + [←/↑/↓/→]` | Move window |
| `Super + Alt + [←/↑/↓/→]` | Resize window |
| `Super + [C/T/3/L]` | Focus in direction |
| `Super + Shift + [C/T/3/L]` | Move in direction |

### Session & Screen
| Keybind | Action |
|---------|--------|
| `Super + L` | Lock screen |
| `Super + Shift + L` | Sleep |
| `Super + Minus` | Zoom out |
| `Super + Equal` | Zoom in |

### Media Controls
| Keybind | Action |
|---------|--------|
| `Super + Shift + N` | Next track |
| `Super + Shift + B` | Previous track |  
| `Super + Shift + P` | Play/pause media |
| `Volume Up/Down` | Audio volume |
| `Super + Shift + M` | Toggle mute |
| `Super + Alt + M` | Toggle microphone |

### System Utilities
| Keybind | Action |
|---------|--------|
| `Super + Shift + S` | Screen snip |
| `Super + Shift + C` | Pick color (Hex) |
| `Super + Shift + D` | Pick color |
| `Super + Shift + Alt + R` | Record region (no sound) |
| `Ctrl + Super + Shift + R` | Record region (with sound) |
| `Super + Alt + R` | Record screen (with sound) |
| `Ctrl + Shift + Print` | Screenshot & clipboard |
| `Ctrl + Alt + R` | Record screen (no sound) |
| `Print` | Screenshot >> clipboard & file |

### Advanced Features
| Keybind | Action |
|---------|--------|
| `Super + X` | Text editor |
| `Super + I` | Settings app |
| `Ctrl + Shift + Escape` | Task manager |
| `Ctrl + Super + V` | Volume mixer |
| `Super + Shift + Alt + RMB` | AI summary for selected text |

---

## Troubleshooting

### Issue: Quickshell not starting
**Solution**: 
```bash
# Check if quickshell is installed
which qs

# Start manually with debug output
pkill qs; qs -c ii

# Check config path
ls -la "$XDG_CONFIG_HOME/quickshell/ii/"

# Check if QML modules are accessible
export QML_IMPORT_PATH="$XDG_CONFIG_HOME/quickshell/ii/modules"
```

### Issue: Python scripts not working
**Solution**:
```bash
# Verify virtual environment
ls -la "$ILLOGICAL_IMPULSE_VIRTUAL_ENV"

# Check if environment variable is set
echo $ILLOGICAL_IMPULSE_VIRTUAL_ENV

# Re-activate and install packages if needed
source "$ILLOGICAL_IMPULSE_VIRTUAL_ENV/bin/activate"
pip list
```

### Issue: Fonts not applied
**Solution**:
```bash
# Refresh font cache
fc-cache -fv

# Check if fonts are installed
fc-list | grep -i rubik
fc-list | grep -i gabarito

# Set font manually if needed
gsettings set org.gnome.desktop.interface font-name 'Rubik 11'
```

### Issue: Missing icons
**Solution**:
```bash
# Check icon installation
ls /usr/share/icons/ | grep -i oneui
ls /usr/local/share/icons/ | grep -i oneui

# Update icon cache
sudo gtk-update-icon-cache /usr/share/icons/hicolor
```

### Issue: Audio controls not working
**Solution**:
```bash
# Check if PipeWire is running
systemctl --user status pipewire pipewire-pulse wireplumber

# Install missing audio packages
sudo pacman -S pipewire pipewire-pulse pipewire-alsa wireplumber

# Check audio devices
wpctl status
```

### Issue: Screen capture not working
**Solution**:
```bash
# Check if screen capture tools are installed
which hyprshot slurp swappy

# Check if xdg-desktop-portal-hyprland is running
systemctl --user status xdg-desktop-portal-hyprland

# Test manual screenshot
hyprshot -m region
```

### Issue: Virtual environment path not found
**Solution**:
```bash
# Check if environment variable is set in hyprland config
grep -r "ILLOGICAL_IMPULSE_VIRTUAL_ENV" ~/.config/hypr/

# Set manually if needed
echo 'env = ILLOGICAL_IMPULSE_VIRTUAL_ENV,'"$XDG_STATE_HOME/quickshell/.venv" >> ~/.config/hypr/custom/env.conf
```

### Issue: Material You colors not generating
**Solution**:
```bash
# Check if matugen is installed
which matugen

# Manually generate colors from wallpaper
matugen image ~/Pictures/your-wallpaper.jpg -m hex

# Clear color cache
rm -rf ~/.cache/illogical-impulse/
```

### Issue: Widgets showing errors
**Solution**:
```bash
# Check Quickshell logs
journalctl --user -u quickshell -f

# Validate QML syntax
qmlscene ~/.config/quickshell/ii/shell.qml

# Check for missing dependencies
qs -c ii --debug
```

### Issue: Network manager not working
**Solution**:
```bash
# Enable NetworkManager service
sudo systemctl enable NetworkManager --now

# Check if nm-applet or plasma-nm is running
ps aux | grep -E "(nm-applet|plasma-nm)"

# Restart network services
sudo systemctl restart NetworkManager
systemctl --user restart plasma-nm
```

### Issue: Brightness controls not working
**Solution**:
```bash
# Check if user is in video group
groups $USER | grep video

# Add user to video group if missing
sudo usermod -aG video $USER

# Check brightness devices
ls /sys/class/backlight/

# Test manual brightness control
brightnessctl set 50%
```

---

## Post-Installation Customization

### Configuration File Structure
The configuration is organized as follows:
```
~/.config/hypr/
├── hyprland.conf          # Main config (sources other files)
├── hypridle.conf          # Idle management
├── hyprlock.conf          # Lock screen
├── monitors.conf          # Monitor configuration
├── workspaces.conf        # Workspace rules
├── hyprland/              # Default configurations
│   ├── env.conf           # Environment variables
│   ├── execs.conf         # Startup applications
│   ├── general.conf       # General Hyprland settings
│   ├── keybinds.conf      # Keybindings
│   ├── rules.conf         # Window rules
│   └── colors.conf        # Color scheme
└── custom/                # Your custom overrides
    ├── env.conf           # Custom environment variables
    ├── execs.conf         # Custom startup applications
    ├── general.conf       # Custom general settings
    ├── keybinds.conf      # Custom keybindings
    └── rules.conf         # Custom window rules
```

### Quickshell Configuration
```
~/.config/quickshell/
├── ii/                    # Main Quickshell config
│   ├── shell.qml         # Main shell definition
│   ├── modules/          # QML modules
│   ├── services/         # Background services
│   ├── widgets/          # UI widgets
│   └── .qmlls.ini        # LSP configuration
└── translations/         # Internationalization
```

### Custom Wallpapers
Place wallpapers in `~/Pictures/` and they will be automatically detected by the theme system.

### Custom Keybinds
Edit `~/.config/hypr/custom/keybinds.conf` to add your custom keybinds.

### Custom Rules
Edit `~/.config/hypr/custom/rules.conf` for window rules.

### Monitor Setup
Run `nwg-displays` to configure monitors, or edit `~/.config/hypr/monitors.conf` manually.

### Theme Customization
The theme system uses Material You colors generated from your wallpaper:
- Colors are automatically generated using `matugen`
- Main theme file: `~/.config/kde-material-you-colors/`
- To force regenerate colors: remove the cache in `~/.cache/illogical-impulse/`

---

## Notes

- This setup uses **Quickshell**, not AGS (Aylur's Gtk Shell)
- All configurations support Material You theming based on wallpaper
- The Python virtual environment is required for various scripts to work
- Make sure to restart after installation to apply all changes
- Keep the `/tmp/dots-hyprland` directory until you're sure everything works

## Additional Tips

### First Launch Checklist
After completing installation and logging into Hyprland:

1. **Verify Quickshell is running**: You should see a status bar at the top
2. **Test keybinds**: Press `Super + /` to see the cheat sheet
3. **Set wallpaper**: Place a wallpaper in `~/Pictures/` and use the wallpaper selector
4. **Configure monitors**: Run `nwg-displays` if you have multiple monitors
5. **Test audio**: Try volume keys and media controls
6. **Test clipboard**: Use `Super + V` for clipboard history

### Performance Tips
- The Quickshell version is more performant than the old AGS version
- If experiencing lag, check system resources with `htop`
- Disable animations in `~/.config/hypr/custom/general.conf` if needed:
  ```
  animations {
      enabled = false
  }
  ```

### Updating
To update the configuration in the future:
```bash
cd /tmp/dots-hyprland
git pull origin main
# Then selectively copy updated configs as needed
```

### Development/Customization
- Edit Quickshell configs in `~/.config/quickshell/ii/`
- Changes to QML files are live-reloaded automatically
- Use VSCode with the Qt QML extension for better editing experience
- Check `~/.config/quickshell/ii/welcome.qml` for the welcome screen

### Backup & Restore
Before making major changes:
```bash
# Create configuration backup
tar -czf "hyprland-backup-$(date +%Y%m%d).tar.gz" ~/.config/hypr ~/.config/quickshell

# Restore if needed
tar -xzf hyprland-backup-YYYYMMDD.tar.gz -C ~/
```

This completes the manual installation. The setup should now be fully functional with all features of the illogical-impulse Hyprland dotfiles.