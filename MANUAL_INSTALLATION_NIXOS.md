# Manual Installation Guide for Hyprland Dotfiles (NixOS)

This guide provides step-by-step instructions to manually install the illogical-impulse Hyprland dotfiles setup on NixOS without using any scripts. This setup uses **Quickshell** (not AGS) for the widgets and UI components.

## Prerequisites

- NixOS with flakes enabled (recommended) or traditional configuration
- Basic terminal knowledge and NixOS configuration experience
- Internet connection
- Root/sudo access for system configuration

## Overview

This setup includes:
- **Hyprland**: Wayland compositor
- **Quickshell**: QtQuick-based widget system for UI components
- **Custom fonts and themes**: Material Design-based theming
- **Audio/media controls**: PipeWire/WirePlumber integration
- **System integrations**: Notifications, screen capture, etc.

---

## Phase 1: NixOS System Preparation

### 1.1 Enable Flakes (if not already enabled)
Add to your `/etc/nixos/configuration.nix`:
```nix
{
  nix.settings.experimental-features = [ "nix-command" "flakes" ];
}
```

Then rebuild:
```bash
sudo nixos-rebuild switch
```

### 1.2 Create Working Directory Structure
```bash
export XDG_BIN_HOME=${XDG_BIN_HOME:-$HOME/.local/bin}
export XDG_CACHE_HOME=${XDG_CACHE_HOME:-$HOME/.cache}
export XDG_CONFIG_HOME=${XDG_CONFIG_HOME:-$HOME/.config}
export XDG_DATA_HOME=${XDG_DATA_HOME:-$HOME/.local/share}
export XDG_STATE_HOME=${XDG_STATE_HOME:-$HOME/.local/state}

mkdir -p "$XDG_BIN_HOME" "$XDG_CACHE_HOME" "$XDG_CONFIG_HOME" "$XDG_DATA_HOME" "$XDG_STATE_HOME"
```

### 1.3 Create Backup (Optional but Recommended)
```bash
# Backup existing configs
BACKUP_DIR="$HOME/backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"
cp -r "$XDG_CONFIG_HOME" "$BACKUP_DIR/config" 2>/dev/null || true
cp -r "$HOME/.local" "$BACKUP_DIR/local" 2>/dev/null || true
echo "Backup created at: $BACKUP_DIR"
```

---

## Phase 2: NixOS Configuration Setup

### 2.1 Basic NixOS Configuration
Create or modify your `/etc/nixos/configuration.nix` to include:

```nix
{ config, pkgs, ... }:

{
  # Enable the X11 windowing system and Hyprland
  services.xserver.enable = true;
  services.displayManager.sddm.enable = true;
  programs.hyprland = {
    enable = true;
    xwayland.enable = true;
  };

  # Audio with PipeWire
  sound.enable = true;
  hardware.pulseaudio.enable = false;
  security.rtkit.enable = true;
  services.pipewire = {
    enable = true;
    alsa.enable = true;
    alsa.support32Bit = true;
    pulse.enable = true;
    wireplumber.enable = true;
  };

  # Enable Bluetooth
  hardware.bluetooth.enable = true;
  services.blueman.enable = true;

  # Enable NetworkManager
  networking.networkmanager.enable = true;

  # XDG Desktop Portals
  xdg.portal = {
    enable = true;
    wlr.enable = true;
    extraPortals = with pkgs; [
      xdg-desktop-portal-hyprland
      xdg-desktop-portal-kde
      xdg-desktop-portal-gtk
    ];
  };

  # Fonts
  fonts.packages = with pkgs; [
    rubik
    (nerdfonts.override { fonts = [ "JetBrainsMono" ]; })
    material-symbols
    twemoji-color-font
    space-grotesk
  ];

  # System packages needed for the setup
  environment.systemPackages = with pkgs; [
    # Core utilities
    git
    wget
    curl
    jq
    ripgrep
    rsync
    bc
    axel
    cmake
    meson
    
    # Wayland/Hyprland essentials
    wl-clipboard
    hyprpicker
    hypridle
    hyprlock
    hyprshot
    slurp
    swappy
    wf-recorder
    
    # Audio control
    pavucontrol
    playerctl
    cava
    
    # System tools
    brightnessctl
    geoclue2
    ddcutil
    cliphist
    fuzzel
    wlogout
    nm-tray
    
    # Screenshot and OCR
    tesseract
    
    # Development tools
    nodejs
    python312
    python312Packages.pip
    python312Packages.virtualenv
    
    # Qt and theme tools
    libsForQt5.qt5ct
    qt6ct
    qt6.qttools
    
    # File managers and utilities
    dolphin
    systemsettings
    kdialog
    
    # Terminal and shell
    kitty
    fish
    starship
    eza
    
    # Fonts
    fontconfig
  ];

  # Enable services
  services.udisks2.enable = true;
  services.gnome.gnome-keyring.enable = true;
  security.polkit.enable = true;
  
  # Enable dconf for settings
  programs.dconf.enable = true;
  
  # Enable fish shell
  programs.fish.enable = true;
}
```

### 2.2 User Configuration
Add user-specific packages in your user configuration or home-manager:

```nix
# If using home-manager, add to home.nix
home.packages = with pkgs; [
  # Additional user packages
  translate-shell
  matugen
  
  # Cursor theme
  bibata-cursors
];
```

### 2.3 Rebuild System
```bash
sudo nixos-rebuild switch
```

---

## Phase 3: Install Quickshell

Since Quickshell might not be available in nixpkgs stable, you may need to build it manually or use a flake.

### 3.1 Method 1: Using Flake (Recommended)
Create a `flake.nix` in your NixOS configuration directory:

```nix
{
  description = "NixOS configuration with Quickshell";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    quickshell = {
      url = "github:outfoxxed/quickshell";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, quickshell, ... }: {
    nixosConfigurations = {
      # Replace 'hostname' with your actual hostname
      hostname = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          ./configuration.nix
          {
            environment.systemPackages = [ quickshell.packages.x86_64-linux.default ];
          }
        ];
      };
    };
  };
}
```

Then rebuild with flakes:
```bash
sudo nixos-rebuild switch --flake .
```

### 3.2 Method 2: Manual Build
If flakes aren't available, build Quickshell manually:

```bash
# Install build dependencies
nix-shell -p cmake qt6.qtbase qt6.qtdeclarative qt6.qttools pkg-config

# Clone and build
git clone https://github.com/outfoxxed/quickshell.git /tmp/quickshell
cd /tmp/quickshell
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
sudo cmake --install build
```

---

## Phase 4: Install Fonts and Themes

### 4.1 Install Additional Fonts
Add to your NixOS configuration:

```nix
fonts.packages = with pkgs; [
  # Core fonts already added above
  gabarito
  readex-pro
  # If not available in nixpkgs, we'll install manually below
];
```

### 4.2 Manual Font Installation (if needed)
For fonts not in nixpkgs:

```bash
# Gabarito font
mkdir -p /tmp/fonts/Gabarito
cd /tmp/fonts/Gabarito
git clone https://github.com/naipefoundry/gabarito.git .
sudo mkdir -p /usr/local/share/fonts/TTF/
sudo cp fonts/ttf/Gabarito*.ttf /usr/local/share/fonts/TTF/
fc-cache -fv
```

### 4.3 Icon Themes
Install OneUI icons manually:
```bash
mkdir -p /tmp/icons/OneUI4-Icons
cd /tmp/icons/OneUI4-Icons
git clone https://github.com/end-4/OneUI4-Icons.git .
sudo mkdir -p /usr/local/share/icons
sudo cp -r OneUI /usr/local/share/icons
sudo cp -r OneUI-dark /usr/local/share/icons
sudo cp -r OneUI-light /usr/local/share/icons
```

### 4.4 GTK Themes
Add to your NixOS configuration:
```nix
environment.systemPackages = with pkgs; [
  adwaita-qt
  adwaita-qt6
  breeze-gtk
  breeze-qt5
  kde-material-you-colors  # If available
];
```

---

## Phase 5: Python Environment Setup

### 5.1 Create Python Virtual Environment
```bash
# Set up virtual environment location
export ILLOGICAL_IMPULSE_VIRTUAL_ENV="$XDG_STATE_HOME/quickshell/.venv"
mkdir -p "$(dirname "$ILLOGICAL_IMPULSE_VIRTUAL_ENV")"

# Clone the repo temporarily to get requirements
git clone https://github.com/elbasel-404/dots-hyprland.git /tmp/dots-hyprland
cd /tmp/dots-hyprland

# Create and populate virtual environment
python -m venv "$ILLOGICAL_IMPULSE_VIRTUAL_ENV"
source "$ILLOGICAL_IMPULSE_VIRTUAL_ENV/bin/activate"
pip install -r scriptdata/requirements.txt
deactivate
```

### 5.2 Install Python Dependencies Globally (Alternative)
Add to your NixOS configuration:
```nix
environment.systemPackages = with pkgs; [
  python312Packages.pillow
  python312Packages.pycairo
  python312Packages.pygobject3
  python312Packages.psutil
  python312Packages.click
  python312Packages.loguru
  python312Packages.tqdm
  python312Packages.pywayland
  python312Packages.setproctitle
  python312Packages.wheel
  python312Packages.build
];
```

---

## Phase 6: Install MicroTeX (Optional)

### 6.1 Using Nix
Add to your configuration:
```nix
environment.systemPackages = with pkgs; [
  # MicroTeX might not be in nixpkgs, so build manually if needed
];
```

### 6.2 Manual Build
```bash
nix-shell -p cmake tinyxml-2 gtkmm3 gtksourceviewmm cairo

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

## Phase 7: System Configuration

### 7.1 User Groups and Services
Add to your NixOS configuration:
```nix
# Add your user to required groups
users.users.yourusername = {
  extraGroups = [ "video" "audio" "networkmanager" "wheel" "input" "i2c" ];
};

# Enable required services
services.ydotool.enable = true;
hardware.i2c.enable = true;

# Environment variables
environment.sessionVariables = {
  ILLOGICAL_IMPULSE_VIRTUAL_ENV = "$XDG_STATE_HOME/quickshell/.venv";
};
```

### 7.2 Configure Font Settings
```bash
# Set default font
gsettings set org.gnome.desktop.interface font-name 'Rubik 11'

# Set dark theme preference
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

### 7.3 Rebuild System
```bash
sudo nixos-rebuild switch
```

---

## Phase 8: Install Configuration Files

### 8.1 Clone the Dotfiles Repository
```bash
cd /tmp
git clone https://github.com/elbasel-404/dots-hyprland.git
cd dots-hyprland
```

### 8.2 Copy Hyprland Configuration
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

### 8.3 Copy Quickshell Configuration
```bash
rsync -av --delete .config/quickshell/ "$XDG_CONFIG_HOME"/quickshell/
```

### 8.4 Copy Other Configuration Files
```bash
# Skip fish and handle miscellaneous configs
rsync -av --delete --exclude 'fish' --exclude 'hypr' --exclude 'quickshell' .config/ "$XDG_CONFIG_HOME"/

# Copy local data files
rsync -av .local/share/icons/ "$XDG_DATA_HOME"/icons/
rsync -av .local/share/konsole/ "$XDG_DATA_HOME"/konsole/
```

### 8.5 Setup Fish Shell Configuration (Optional)
```bash
# Only if you want to use fish shell
if command -v fish >/dev/null 2>&1; then
    rsync -av .config/fish/ "$XDG_CONFIG_HOME"/fish/
fi
```

### 8.6 Setup ZSH Configuration (Optional)
```bash
# Setup zsh config if using zsh
if [ -f ~/.zshrc ]; then
    if ! grep -q 'source ${XDG_CONFIG_HOME:-~/.config}/zshrc.d/dots-hyprland.zsh' ~/.zshrc; then
        echo 'source ${XDG_CONFIG_HOME:-~/.config}/zshrc.d/dots-hyprland.zsh' >> ~/.zshrc
    fi
fi
```

---

## Phase 9: Final Configuration

### 9.1 Setup Quickshell LSP Support
```bash
touch "$XDG_CONFIG_HOME/quickshell/ii/.qmlls.ini"
```

### 9.2 Set Environment Variables
```bash
# Add to your shell profile
echo "export ILLOGICAL_IMPULSE_VIRTUAL_ENV=\"$ILLOGICAL_IMPULSE_VIRTUAL_ENV\"" >> ~/.profile
```

---

## Phase 10: Launch and Verify

### 10.1 Logout and Login to Hyprland
1. Logout from your current session
2. Select "Hyprland" from SDDM login manager
3. Login

### 10.2 Start Quickshell
Once in Hyprland, the Quickshell should start automatically. If not:
```bash
quickshell -c ii
```

### 10.3 Verify Installation
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
which quickshell

# Start manually with debug output
pkill quickshell; quickshell -c ii

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

# Rebuild NixOS to ensure fonts are properly installed
sudo nixos-rebuild switch
```

### Issue: Missing packages
**Solution**:
```bash
# Check if package is available in nixpkgs
nix search nixpkgs packagename

# Add missing packages to configuration.nix and rebuild
sudo nixos-rebuild switch
```

### Issue: Audio controls not working
**Solution**:
```bash
# Check if PipeWire is running
systemctl --user status pipewire pipewire-pulse wireplumber

# Ensure PipeWire is enabled in configuration.nix
# Then rebuild system
sudo nixos-rebuild switch
```

### Issue: Screen capture not working
**Solution**:
```bash
# Check if screen capture tools are available
which hyprshot slurp swappy

# Make sure xdg-desktop-portal-hyprland is in your config
# and rebuild system
```

### Issue: Virtual environment path not found
**Solution**:
```bash
# Check if environment variable is set in hyprland config
grep -r "ILLOGICAL_IMPULSE_VIRTUAL_ENV" ~/.config/hypr/

# Add to NixOS configuration:
# environment.sessionVariables.ILLOGICAL_IMPULSE_VIRTUAL_ENV = "$XDG_STATE_HOME/quickshell/.venv";
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

## NixOS-Specific Notes

### Managing Updates
```bash
# Update your NixOS configuration
sudo nixos-rebuild switch

# If using flakes
sudo nixos-rebuild switch --flake .

# Update flake inputs
nix flake update
```

### Package Management
- Use `nix search nixpkgs packagename` to find packages
- All packages should be declared in your `configuration.nix`
- User packages can be managed via home-manager

### Configuration Management
- All system configuration should be in `/etc/nixos/configuration.nix`
- Consider using home-manager for user-specific configurations
- Use git to track your NixOS configuration files

### Performance Tips
- NixOS builds can be resource-intensive; ensure adequate disk space
- Consider using binary caches to speed up builds
- The Quickshell version is more performant than the old AGS version

### Development/Customization
- Edit Quickshell configs in `~/.config/quickshell/ii/`
- Changes to QML files are live-reloaded automatically
- Use VSCode with the Qt QML extension for better editing experience

### Backup & Restore
```bash
# Create configuration backup
sudo cp -r /etc/nixos /etc/nixos.backup-$(date +%Y%m%d)
tar -czf "hyprland-backup-$(date +%Y%m%d).tar.gz" ~/.config/hypr ~/.config/quickshell

# Restore configs
tar -xzf hyprland-backup-YYYYMMDD.tar.gz -C ~/
```

This completes the NixOS-specific manual installation. The setup should now be fully functional with all features of the illogical-impulse Hyprland dotfiles on NixOS.