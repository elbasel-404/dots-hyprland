# NixOS Manual Installation Guide for end-4's Hyprland Dotfiles (illogical-impulse)

This guide provides step-by-step instructions to manually set up the **illogical-impulse** Hyprland dotfiles on NixOS. Instead of copying configuration files, this guide includes all configuration content with detailed explanations of what each section does.

## Prerequisites

- **NixOS system** with flakes enabled
- **Internet connection** for downloading packages
- **Basic NixOS configuration knowledge**
- **Backup your existing configurations** before proceeding

### ⚠️ Important Warnings

1. **This will modify your NixOS configuration**: You'll need to rebuild your system configuration.
2. **Home Manager recommended**: This guide assumes you're using Home Manager for user configurations.
3. **Large rebuild**: The complete setup includes many packages and will require a full system rebuild.
4. **GPU compatibility**: Some features require proper GPU drivers configuration in NixOS.

### Backup Your Configurations

```bash
# Backup current configurations
sudo cp /etc/nixos/configuration.nix /etc/nixos/configuration.nix.backup
cp ~/.config/home-manager/home.nix ~/.config/home-manager/home.nix.backup 2>/dev/null || true
```

## Table of Contents

1. [NixOS System Configuration](#1-nixos-system-configuration)
2. [Home Manager Setup](#2-home-manager-setup)
3. [Hyprland Configuration](#3-hyprland-configuration)
4. [Quickshell Configuration](#4-quickshell-configuration)
5. [Terminal Configuration (Fish & Kitty)](#5-terminal-configuration-fish--kitty)
6. [Application Configurations](#6-application-configurations)
7. [Theme and Styling](#7-theme-and-styling)
8. [Final Steps](#8-final-steps)

---

## 1. NixOS System Configuration

First, modify your `/etc/nixos/configuration.nix` to include Hyprland and required system packages:

```nix
{ config, pkgs, ... }:

{
  # Enable flakes (required for Hyprland)
  nix.settings.experimental-features = [ "nix-command" "flakes" ];

  # Hyprland configuration
  programs.hyprland = {
    enable = true;
    xwayland.enable = true;
  };

  # XDG portal for screen sharing and file picker
  xdg.portal = {
    enable = true;
    extraPortals = with pkgs; [
      xdg-desktop-portal-hyprland  # Hyprland-specific portal
      xdg-desktop-portal-gtk       # GTK file picker
    ];
  };

  # Audio configuration
  security.rtkit.enable = true;  # Required for pipewire
  services.pipewire = {
    enable = true;
    alsa.enable = true;
    alsa.support32Bit = true;
    pulse.enable = true;
    wireplumber.enable = true;
  };

  # Bluetooth support
  hardware.bluetooth = {
    enable = true;
    powerOnBoot = true;
  };
  services.blueman.enable = true;

  # Graphics drivers (choose based on your hardware)
  hardware.opengl = {
    enable = true;
    driSupport = true;
    driSupport32Bit = true;
  };

  # System packages required for the dotfiles
  environment.systemPackages = with pkgs; [
    # Core utilities
    wget curl git rsync ripgrep jq bc coreutils
    
    # Hyprland ecosystem
    hyprpicker        # Color picker
    hypridle          # Idle daemon
    hyprlock          # Screen locker
    hyprshot          # Screenshot tool
    
    # Media and capture
    slurp             # Region selector
    swappy            # Screenshot annotation
    wf-recorder       # Screen recorder
    
    # Color and theming
    matugen           # Material color generator
    
    # Utilities
    cliphist          # Clipboard manager
    wl-clipboard      # Wayland clipboard
    brightnessctl     # Brightness control
    playerctl         # Media player control
    
    # Development tools
    python312         # Required Python version
    uv                # Python package manager
    
    # Fonts
    (nerdfonts.override { fonts = [ "JetBrainsMono" ]; })
    noto-fonts
    noto-fonts-emoji
  ];

  # User groups for hardware access
  users.users.${config.users.users.yourUsername.name} = {
    extraGroups = [ "video" "input" "i2c" ];
  };

  # i2c support for external monitor brightness
  boot.kernelModules = [ "i2c-dev" ];
  services.udev.extraRules = ''
    SUBSYSTEM=="i2c-dev", GROUP="i2c", MODE="0660"
  '';

  # Font configuration
  fonts = {
    packages = with pkgs; [
      # Add the custom fonts used in the dotfiles
      rubik
      gabarito
    ];
    fontconfig = {
      enable = true;
      defaultFonts = {
        sansSerif = [ "Rubik" "Noto Sans" ];
        monospace = [ "JetBrainsMono Nerd Font" "Noto Sans Mono" ];
        emoji = [ "Noto Color Emoji" ];
      };
    };
  };
}
```

**What this configuration does:**
- **Flakes**: Enables experimental flakes feature needed for Hyprland
- **Hyprland**: Enables the compositor with Xwayland support for legacy apps
- **XDG Portal**: Provides desktop integration for file pickers and screen sharing
- **Audio**: Sets up PipeWire with ALSA and PulseAudio compatibility
- **Bluetooth**: Enables Bluetooth with GUI manager
- **Graphics**: Enables hardware acceleration and 32-bit support
- **System packages**: Installs all required command-line tools and utilities
- **User groups**: Adds your user to hardware access groups
- **i2c support**: Enables external monitor brightness control
- **Fonts**: Configures the fonts used in the dotfiles

After making these changes, rebuild your system:
```bash
sudo nixos-rebuild switch
```

## 2. Home Manager Setup

Install and configure Home Manager if you haven't already:

```bash
# Add Home Manager channel
nix-channel --add https://github.com/nix-community/home-manager/archive/master.tar.gz home-manager
nix-channel --update

# Install Home Manager
nix-shell '<home-manager>' -A install
```

Create or modify your `~/.config/home-manager/home.nix`:

```nix
{ config, pkgs, ... }:

{
  # Basic Home Manager configuration
  home.username = "yourUsername";
  home.homeDirectory = "/home/yourUsername";
  home.stateVersion = "23.11";

  # Let Home Manager manage itself
  programs.home-manager.enable = true;

  # GUI applications
  home.packages = with pkgs; [
    # Terminal and shell
    kitty               # GPU-accelerated terminal
    fish                # User-friendly shell
    starship            # Shell prompt
    
    # Launcher and widgets
    fuzzel              # Application launcher
    
    # Development and utilities
    dolphin             # File manager
    pavucontrol         # Audio control GUI
    
    # Media and graphics
    cava                # Audio visualizer
    
    # Qt/KDE theming
    qt6.qtbase
    qt6.qtwayland
    libsForQt5.qt5.qtwayland
    
    # Additional utilities
    translate-shell     # Translation tool
    tesseract           # OCR engine
    wlogout             # Logout menu
  ];

  # Python environment for Quickshell
  home.file.".local/state/quickshell/.venv" = {
    source = pkgs.python312.withPackages (ps: with ps; [
      pillow
      material-color-utilities
      pycairo
      pygobject3
      requests
      setuptools
      wheel
    ]);
    recursive = true;
  };

  # Environment variables
  home.sessionVariables = {
    # XDG directories
    XDG_CACHE_HOME = "$HOME/.cache";
    XDG_CONFIG_HOME = "$HOME/.config";
    XDG_DATA_HOME = "$HOME/.local/share";
    XDG_STATE_HOME = "$HOME/.local/state";
    
    # Quickshell Python environment
    ILLOGICAL_IMPULSE_VIRTUAL_ENV = "$HOME/.local/state/quickshell/.venv";
    
    # Qt/Wayland configuration
    QT_QPA_PLATFORM = "wayland";
    QT_WAYLAND_DISABLE_WINDOWDECORATION = "1";
  };
}
```

**What this Home Manager configuration does:**
- **User packages**: Installs GUI applications that don't need system-level access
- **Python environment**: Creates the Python virtual environment needed by Quickshell
- **Environment variables**: Sets up XDG directories and Qt/Wayland configuration
- **Terminal setup**: Configures Fish shell and Kitty terminal

Apply the Home Manager configuration:
```bash
home-manager switch
```

## 3. Hyprland Configuration

Create the Hyprland configuration directory and files:

```bash
mkdir -p ~/.config/hypr/{hyprland,custom}
```

### Main Hyprland Configuration

Create `~/.config/hypr/hyprland.conf`:

```bash
# Main Hyprland configuration file
# This file sources other configuration files for modularity

# Quickshell configuration identifier
$qsConfig = ii

# Initialize global submap (required for keybinds to work)
exec = hyprctl dispatch submap global
submap = global

# Source default configurations
source=hyprland/env.conf       # Environment variables
source=hyprland/execs.conf     # Startup applications  
source=hyprland/general.conf   # General settings (animations, input, etc.)
source=hyprland/rules.conf     # Window rules
source=hyprland/colors.conf    # Color scheme
source=hyprland/keybinds.conf  # Keybindings

# Source custom user configurations (these override defaults)
source=custom/env.conf         # Custom environment variables
source=custom/execs.conf       # Custom startup applications
source=custom/general.conf     # Custom general settings
source=custom/rules.conf       # Custom window rules  
source=custom/keybinds.conf    # Custom keybindings

# Monitor and workspace configurations
source=workspaces.conf         # Workspace setup
source=monitors.conf           # Monitor configuration
```

**What this does:**
- **$qsConfig**: Sets the Quickshell configuration name
- **submap**: Initializes global keymap for keybinds to work
- **Modular structure**: Separates concerns into different files
- **Custom overrides**: Allows personal customizations without modifying defaults

### General Settings

Create `~/.config/hypr/hyprland/general.conf`:

```bash
# Monitor configuration (adjust for your setup)
monitor=,preferred,auto,1
# monitor=,addreserved, 0, 0, 0, 0 # Custom reserved area for panels

# Touchpad gestures
gesture = 3, swipe, move,                           # 3-finger swipe to move windows
gesture = 4, horizontal, workspace                  # 4-finger horizontal swipe for workspaces  
gesture = 4, pinch, float                          # 4-finger pinch to toggle floating
gesture = 4, up, dispatcher, global, quickshell:overviewToggle    # 4-finger up for overview
gesture = 4, down, dispatcher, global, quickshell:overviewClose   # 4-finger down to close overview

gestures {
    workspace_swipe_distance = 700                  # Distance for workspace swipe
    workspace_swipe_cancel_ratio = 0.2             # Cancel ratio for incomplete swipes
    workspace_swipe_min_speed_to_force = 5         # Minimum speed to force swipe
    workspace_swipe_direction_lock = true          # Lock swipe direction
    workspace_swipe_direction_lock_threshold = 10  # Direction lock threshold
    workspace_swipe_create_new = true              # Create new workspace when swiping past last
}

general {
    # Window gaps and borders
    gaps_in = 4                    # Gap between windows
    gaps_out = 5                   # Gap between windows and screen edge
    gaps_workspaces = 50           # Gap between workspaces in overview

    border_size = 1                # Window border thickness
    col.active_border = rgba(0DB7D4FF)    # Active window border color (cyan)
    col.inactive_border = rgba(31313600)   # Inactive window border color (transparent)
    resize_on_border = true        # Allow resizing by dragging borders

    no_focus_fallback = true       # Don't focus random windows when active closes
    allow_tearing = true           # Allow screen tearing for low latency (gaming)

    # Window snapping
    snap {
        enabled = true             # Enable window snapping
        window_gap = 4            # Gap when snapping to windows
        monitor_gap = 5           # Gap when snapping to monitor edges
        respect_gaps = true       # Respect existing gap configuration
    }
}

# Window layout algorithm
dwindle {
    preserve_split = true          # Preserve split direction when opening new windows
    smart_split = false           # Disable smart split (predictable behavior)
    smart_resizing = false        # Disable smart resizing
}

# Window decorations
decoration {
    rounding = 18                 # Corner rounding radius

    # Blur effects
    blur {
        enabled = true            # Enable blur
        xray = true              # Blur behind transparent windows
        special = false          # Don't blur special workspaces
        new_optimizations = true  # Use new blur optimizations
        size = 14                # Blur radius
        passes = 3               # Number of blur passes (quality vs performance)
        brightness = 1           # Blur brightness multiplier
        noise = 0.04             # Add noise to blur (prevents banding)
        contrast = 1             # Blur contrast
        popups = true            # Blur popups/menus
        popups_ignorealpha = 0.6 # Ignore alpha for popup blur
        input_methods = true     # Blur input methods (keyboards)
        input_methods_ignorealpha = 0.8  # Ignore alpha for input method blur
    }

    # Drop shadows
    shadow {
        enabled = true           # Enable shadows
        ignore_window = true     # Shadow follows window shape, not decoration
        range = 30              # Shadow blur radius
        offset = 0 2            # Shadow offset (x, y)
        render_power = 4        # Shadow falloff curve (higher = sharper edge)
        color = rgba(00000010)  # Shadow color (semi-transparent black)
    }

    # Window dimming
    dim_inactive = true         # Dim inactive windows
    dim_strength = 0.025       # Dimming strength for inactive windows
    dim_special = 0.07         # Dimming strength for special workspaces
}

# Animations configuration
animations {
    enabled = true

    # Bezier curves for smooth animations
    bezier = expressiveFastSpatial, 0.42, 1.67, 0.21, 0.90      # Fast spatial movement
    bezier = expressiveSlowSpatial, 0.39, 1.29, 0.35, 0.98      # Slow spatial movement  
    bezier = expressiveDefaultSpatial, 0.38, 1.21, 0.22, 1.00   # Default spatial movement
    bezier = emphasizedDecel, 0.05, 0.7, 0.1, 1                 # Emphasized deceleration
    bezier = emphasizedAccel, 0.3, 0, 0.8, 0.15                 # Emphasized acceleration
    bezier = standardDecel, 0, 0, 0, 1                           # Standard deceleration
    bezier = menu_decel, 0.1, 1, 0, 1                           # Menu deceleration
    bezier = menu_accel, 0.52, 0.03, 0.72, 0.08                 # Menu acceleration

    # Window animations
    animation = windowsIn, 1, 3, emphasizedDecel, popin 80%     # Window open animation
    animation = windowsOut, 1, 2, emphasizedDecel, popin 90%    # Window close animation
    animation = windowsMove, 1, 3, emphasizedDecel, slide       # Window move animation
    animation = border, 1, 10, emphasizedDecel                  # Border color animation

    # Layer animations (panels, overlays)
    animation = layersIn, 1, 2.7, emphasizedDecel, popin 93%    # Layer open animation
    animation = layersOut, 1, 2.4, menu_accel, popin 94%       # Layer close animation

    # Fade animations
    animation = fadeLayersIn, 1, 0.5, menu_decel               # Layer fade in
    animation = fadeLayersOut, 1, 2.7, menu_accel              # Layer fade out

    # Workspace animations
    animation = workspaces, 1, 7, menu_decel, slide            # Workspace switch animation
    animation = specialWorkspaceIn, 1, 2.8, emphasizedDecel, slidevert   # Special workspace in
    animation = specialWorkspaceOut, 1, 1.2, emphasizedAccel, slidevert  # Special workspace out
}

# Input configuration
input {
    kb_layout = us                 # Keyboard layout
    numlock_by_default = true     # Enable numlock by default
    repeat_delay = 250            # Key repeat delay (ms)
    repeat_rate = 35              # Key repeat rate (per second)

    follow_mouse = 1              # Focus follows mouse
    off_window_axis_events = 2    # Handle off-window mouse events

    touchpad {
        natural_scroll = yes       # Natural scrolling (macOS style)
        disable_while_typing = true # Disable touchpad while typing
        clickfinger_behavior = true # Clickfinger instead of click zones
        scroll_factor = 0.5       # Scroll sensitivity
    }
}

# Miscellaneous settings
misc {
    disable_hyprland_logo = true          # Disable Hyprland logo on empty workspace
    disable_splash_rendering = true       # Disable splash screen
    vfr = 1                              # Variable refresh rate
    vrr = 1                              # Variable refresh rate (for gaming)
    mouse_move_enables_dpms = true       # Wake from DPMS on mouse movement
    key_press_enables_dpms = true        # Wake from DPMS on keypress
    animate_manual_resizes = false       # Don't animate manual window resizes
    animate_mouse_windowdragging = false # Don't animate mouse window dragging
    enable_swallow = false               # Disable window swallowing
    swallow_regex = (foot|kitty|allacritty|Alacritty)  # Apps that can be swallowed
    new_window_takes_over_fullscreen = 2 # New window behavior in fullscreen
    allow_session_lock_restore = true    # Allow restoring after session lock
    session_lock_xray = true            # See through session lock
    initial_workspace_tracking = false  # Don't track initial workspace
    focus_on_activate = true            # Focus windows when activated
}

# Keybind behavior
binds {
    scroll_event_delay = 0              # No delay for scroll events
    hide_special_on_workspace_change = true  # Hide special workspace when switching
}

# Cursor configuration
cursor {
    zoom_factor = 1                     # Cursor zoom factor
    zoom_rigid = false                  # Allow flexible cursor zooming
    hotspot_padding = 1                 # Padding around cursor hotspot
}

# Plugin configuration (if using hyprexpo for workspace overview)
plugin {
    hyprexpo {
        columns = 3                     # Number of columns in overview
        gap_size = 5                   # Gap between workspaces in overview
        bg_col = rgb(000000)           # Background color in overview
        workspace_method = first 1      # Workspace positioning method
        enable_gesture = false         # Disable touchpad gesture for overview
        gesture_distance = 300         # Gesture distance threshold
        gesture_positive = false       # Gesture direction
    }
}
```

**What this configuration does:**
- **Monitor setup**: Configures display with preferred resolution
- **Gestures**: Sets up touchpad gestures for workspace and window management
- **Window management**: Configures gaps, borders, and snapping behavior
- **Visual effects**: Sets up blur, shadows, and window dimming
- **Animations**: Defines smooth Material Design-inspired animations
- **Input handling**: Configures keyboard and touchpad behavior
- **Performance**: Optimizes for smooth experience with VRR support

### Environment Variables

Create `~/.config/hypr/hyprland/env.conf`:

```bash
# Environment variables for Hyprland session

# XDG directories
env = XDG_CURRENT_DESKTOP,Hyprland
env = XDG_SESSION_TYPE,wayland  
env = XDG_SESSION_DESKTOP,Hyprland

# Qt configuration for Wayland
env = QT_QPA_PLATFORM,wayland;xcb        # Prefer Wayland, fallback to X11
env = QT_WAYLAND_DISABLE_WINDOWDECORATION,1  # Disable Qt window decorations
env = QT_AUTO_SCREEN_SCALE_FACTOR,1      # Enable automatic DPI scaling

# GTK configuration
env = GDK_BACKEND,wayland,x11            # Prefer Wayland for GTK apps
env = GTK_THEME,Adw-dark                 # Use dark Adwaita theme

# Mozilla/Firefox Wayland support
env = MOZ_ENABLE_WAYLAND,1               # Enable native Wayland for Firefox

# NVIDIA-specific (uncomment if using NVIDIA GPU)
# env = LIBVA_DRIVER_NAME,nvidia
# env = __GLX_VENDOR_LIBRARY_NAME,nvidia
# env = GBM_BACKEND,nvidia-drm
# env = __GL_GSYNC_ALLOWED,1
# env = __GL_VRR_ALLOWED,1

# Quickshell Python environment
env = ILLOGICAL_IMPULSE_VIRTUAL_ENV,$HOME/.local/state/quickshell/.venv

# Cursor theme
env = XCURSOR_THEME,Bibata-Modern-Classic
env = XCURSOR_SIZE,24

# Java applications (fix for some Java apps)
env = _JAVA_AWT_WM_NONREPARENTING,1

# SDL games
env = SDL_VIDEODRIVER,wayland            # Use Wayland for SDL games
```

### Keybindings

Create `~/.config/hypr/hyprland/keybinds.conf`:

```bash
# Keybindings for illogical-impulse
# Lines ending with `# [hidden]` won't be shown on cheatsheet
# Lines starting with #! are section headings

#!
##! Shell and Overview
# Overview/launcher toggle (most important keybind)
bindid = Super, Super_L, Toggle overview, global, quickshell:overviewToggleRelease
bindid = Super, Super_R, Toggle overview, global, quickshell:overviewToggleRelease
bind = Super, Super_L, exec, qs -c $qsConfig ipc call TEST_ALIVE || pkill fuzzel || fuzzel # [hidden] Fallback launcher
bind = Super, Super_R, exec, qs -c $qsConfig ipc call TEST_ALIVE || pkill fuzzel || fuzzel # [hidden] Fallback launcher

# Overview interrupt binds (hidden from cheatsheet)
binditn = Super, catchall, global, quickshell:overviewToggleReleaseInterrupt # [hidden]
bind = Ctrl, Super_L, global, quickshell:overviewToggleReleaseInterrupt # [hidden]
bind = Ctrl, Super_R, global, quickshell:overviewToggleReleaseInterrupt # [hidden]
bind = Super, mouse:272, global, quickshell:overviewToggleReleaseInterrupt # [hidden]

# Workspace number display
bindit = ,Super_L, global, quickshell:workspaceNumber # [hidden]
bindit = ,Super_R, global, quickshell:workspaceNumber # [hidden]

# Specialized overlays
bindd = Super, V, Clipboard history, global, quickshell:overviewClipboardToggle
bindd = Super, Period, Emoji picker, global, quickshell:overviewEmojiToggle
bindd = Super, Tab, Toggle overview (alt), global, quickshell:overviewToggle

# Sidebars
bindd = Super, A, Toggle left sidebar, global, quickshell:sidebarLeftToggle
bind = Super+Alt, A, global, quickshell:sidebarLeftToggleDetach # [hidden]
bind = Super, B, global, quickshell:sidebarLeftToggle # [hidden]

#!
##! Applications
bind = Super, Return, Launch terminal, exec, kitty
bind = Super, E, Launch file manager, exec, dolphin
bind = Super, W, Launch browser, exec, firefox
bind = Super, X, Launch system monitor, exec, kitty -e htop

#!
##! Windows
# Window management
bind = Super, Q, Close window, killactive
bind = Super, F, Toggle fullscreen, fullscreen, 0
bind = Super+Shift, F, Toggle fake fullscreen, fullscreen, 1
bind = Super, Space, Toggle floating, togglefloating
bind = Super, P, Toggle pinned (sticky), pin

# Window focus
bind = Super, left, Focus left, movefocus, l
bind = Super, right, Focus right, movefocus, r
bind = Super, up, Focus up, movefocus, u
bind = Super, down, Focus down, movefocus, d
bind = Super, H, Focus left, movefocus, l
bind = Super, L, Focus right, movefocus, r
bind = Super, K, Focus up, movefocus, u
bind = Super, J, Focus down, movefocus, d

# Window movement
bind = Super+Shift, left, Move window left, movewindow, l
bind = Super+Shift, right, Move window right, movewindow, r
bind = Super+Shift, up, Move window up, movewindow, u
bind = Super+Shift, down, Move window down, movewindow, d
bind = Super+Shift, H, Move window left, movewindow, l
bind = Super+Shift, L, Move window right, movewindow, r
bind = Super+Shift, K, Move window up, movewindow, u
bind = Super+Shift, J, Move window down, movewindow, j

# Window resizing
bind = Super+Ctrl, left, Resize window, resizeactive, -60 0
bind = Super+Ctrl, right, Resize window, resizeactive, 60 0
bind = Super+Ctrl, up, Resize window, resizeactive, 0 -60
bind = Super+Ctrl, down, Resize window, resizeactive, 0 60
bind = Super+Ctrl, H, Resize window, resizeactive, -60 0
bind = Super+Ctrl, L, Resize window, resizeactive, 60 0
bind = Super+Ctrl, K, Resize window, resizeactive, 0 -60
bind = Super+Ctrl, J, Resize window, resizeactive, 0 60

#!
##! Workspaces
# Workspace switching
bind = Super, 1, Switch to workspace 1, workspace, 1
bind = Super, 2, Switch to workspace 2, workspace, 2
bind = Super, 3, Switch to workspace 3, workspace, 3
bind = Super, 4, Switch to workspace 4, workspace, 4
bind = Super, 5, Switch to workspace 5, workspace, 5
bind = Super, 6, Switch to workspace 6, workspace, 6
bind = Super, 7, Switch to workspace 7, workspace, 7
bind = Super, 8, Switch to workspace 8, workspace, 8
bind = Super, 9, Switch to workspace 9, workspace, 9
bind = Super, 0, Switch to workspace 10, workspace, 10

# Move window to workspace
bind = Super+Shift, 1, Move to workspace 1, movetoworkspace, 1
bind = Super+Shift, 2, Move to workspace 2, movetoworkspace, 2
bind = Super+Shift, 3, Move to workspace 3, movetoworkspace, 3
bind = Super+Shift, 4, Move to workspace 4, movetoworkspace, 4
bind = Super+Shift, 5, Move to workspace 5, movetoworkspace, 5
bind = Super+Shift, 6, Move to workspace 6, movetoworkspace, 6
bind = Super+Shift, 7, Move to workspace 7, movetoworkspace, 7
bind = Super+Shift, 8, Move to workspace 8, movetoworkspace, 8
bind = Super+Shift, 9, Move to workspace 9, movetoworkspace, 9
bind = Super+Shift, 0, Move to workspace 10, movetoworkspace, 10

# Special workspace (scratchpad)
bind = Super, grave, Toggle scratchpad, togglespecialworkspace
bind = Super+Shift, grave, Move to scratchpad, movetoworkspace, special

# Workspace scrolling
bind = Super, mouse_down, Next workspace, workspace, e+1
bind = Super, mouse_up, Previous workspace, workspace, e-1

#!
##! Media and System
# Audio control
bindle = , XF86AudioRaiseVolume, Volume up, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+
bindle = , XF86AudioLowerVolume, Volume down, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%-
bind = , XF86AudioMute, Toggle mute, exec, wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle
bind = , XF86AudioMicMute, Toggle mic mute, exec, wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle

# Media control
bind = , XF86AudioPlay, Play/pause, exec, playerctl play-pause
bind = , XF86AudioNext, Next track, exec, playerctl next
bind = , XF86AudioPrev, Previous track, exec, playerctl previous

# Brightness control
bindle = , XF86MonBrightnessUp, Brightness up, exec, brightnessctl set 5%+
bindle = , XF86MonBrightnessDown, Brightness down, exec, brightnessctl set 5%-

# Screenshot and recording
bind = , Print, Screenshot area, exec, hyprshot -m region --clipboard-only
bind = Shift, Print, Screenshot window, exec, hyprshot -m window --clipboard-only
bind = Super, Print, Screenshot full, exec, hyprshot -m output --clipboard-only
bind = Super+Shift, S, Screenshot area (save), exec, hyprshot -m region
bind = Super+Shift, R, Screen recording, exec, wf-recorder -g "$(slurp)" -f ~/Videos/recording_$(date +'%Y-%m-%d_%H-%M-%S').mp4

#!
##! System Controls
# Session management
bind = Super+Shift, E, Logout menu, exec, wlogout
bind = Super+Alt, L, Lock screen, exec, hyprlock
bind = Super+Shift, Q, Kill Hyprland, exit

# System utilities
bind = Super, R, Reload Hyprland, exec, hyprctl reload
bind = Super+Shift, T, Toggle idle inhibit, exec, hyprctl dispatch global "idleinhibit toggle"

# Color picker and theming
bind = Super+Ctrl, C, Color picker, exec, hyprpicker -a
bind = Super+Ctrl, T, Wallpaper selector, global, quickshell:wallpaperToggle

#!
##! Special Functions
# Window rules toggle
bind = Super+Shift, O, Toggle opacity, exec, hyprctl keyword windowrule "opacity 0.8 0.8,.*"
bind = Super+Alt, F, Toggle fake fullscreen, fakefullscreen

# Emergency fallback
bind = Super+Shift+Ctrl, R, Restart Quickshell, exec, pkill qs; qs -c ii &

# Mouse bindings
bindm = Super, mouse:272, Move window, movewindow
bindm = Super, mouse:273, Resize window, resizewindow
```

**What these keybindings do:**
- **Super key**: Primary modifier (Windows key)
- **Overview**: Super alone toggles the application launcher/overview
- **Window management**: Standard tiling window manager controls
- **Workspaces**: Number keys switch/move between workspaces
- **Media keys**: Hardware keys control volume, brightness, and media
- **Screenshots**: Various screenshot and recording options
- **System**: Logout, lock, reload, and system utilities

### Startup Applications

Create `~/.config/hypr/hyprland/execs.conf`:

```bash
# Startup applications and services

# Core system services
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
exec-once = systemctl --user import-environment WAYLAND_DISPLAY XDG_CURRENT_DESKTOP

# Authentication agent
exec-once = /usr/lib/polkit-kde-authentication-agent-1

# Clipboard manager
exec-once = wl-paste --type text --watch cliphist store
exec-once = wl-paste --type image --watch cliphist store

# Idle and lock daemon
exec-once = hypridle

# Quickshell (main interface)
exec-once = qs -c ii

# Notification daemon (if not using Quickshell notifications)
# exec-once = dunst

# Audio
exec-once = wpctl set-volume @DEFAULT_AUDIO_SINK@ 50%

# Network manager applet
exec-once = nm-applet --indicator

# Bluetooth manager
exec-once = blueman-applet

# Auto-mount removable media
exec-once = udisks2

# Wallpaper (fallback if Quickshell doesn't set one)
exec-once = hyprpaper

# Screen sharing portal fix
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP=Hyprland
```

**What these startup applications do:**
- **D-Bus**: Sets up desktop environment variables
- **Polkit**: Authentication agent for system operations
- **Clipboard**: Manages clipboard history with cliphist
- **Hypridle**: Handles screen locking and power management
- **Quickshell**: Starts the main interface (status bar, widgets)
- **Audio**: Sets default volume level
- **System tray**: Network and Bluetooth applets
- **Media**: Auto-mounting and screen sharing support

### Monitor and Workspace Configuration

Create `~/.config/hypr/monitors.conf`:

```bash
# Monitor configuration
# Adjust these settings for your specific monitors

# Primary monitor (laptop/main display)
monitor = eDP-1, preferred, 0x0, 1
# monitor = eDP-1, 1920x1080@60, 0x0, 1.25  # Example with specific resolution and scaling

# External monitor examples (uncomment and adjust as needed)
# monitor = HDMI-A-1, 1920x1080@60, 1920x0, 1        # External monitor to the right
# monitor = DP-1, 2560x1440@144, 1920x0, 1           # High refresh rate monitor
# monitor = HDMI-A-1, preferred, 0x0, 1, mirror, eDP-1  # Mirror main display

# Fallback for unknown monitors
monitor = , preferred, auto, 1

# Workspace assignments per monitor
# workspace = 1, monitor:eDP-1, default:true
# workspace = 2, monitor:eDP-1
# workspace = 3, monitor:eDP-1
# workspace = 4, monitor:HDMI-A-1, default:true
# workspace = 5, monitor:HDMI-A-1
```

Create `~/.config/hypr/workspaces.conf`:

```bash
# Workspace configuration

# Default workspace layout
workspace = 1, monitor:, default:true
workspace = 2, monitor:
workspace = 3, monitor:
workspace = 4, monitor:
workspace = 5, monitor:
workspace = 6, monitor:
workspace = 7, monitor:
workspace = 8, monitor:
workspace = 9, monitor:
workspace = 10, monitor:

# Special workspaces
workspace = special:scratchpad, gapsout:100
```

### Window Rules

Create `~/.config/hypr/hyprland/rules.conf`:

```bash
# Window rules for specific applications

# Floating windows
windowrulev2 = float, class:^(pavucontrol)$
windowrulev2 = float, class:^(blueman-manager)$
windowrulev2 = float, class:^(nm-connection-editor)$
windowrulev2 = float, class:^(kdialog)$
windowrulev2 = float, class:^(org.kde.polkit-kde-authentication-agent-1)$

# Picture-in-picture
windowrulev2 = float, title:^(Picture-in-Picture)$
windowrulev2 = pin, title:^(Picture-in-Picture)$
windowrulev2 = move 69% 4%, title:^(Picture-in-Picture)$
windowrulev2 = size 30% 30%, title:^(Picture-in-Picture)$

# Firefox specific rules
windowrulev2 = float, class:^(firefox)$, title:^(Library)$
windowrulev2 = float, class:^(firefox)$, title:^(Firefox — Sharing Indicator)$

# Gaming optimizations
windowrulev2 = immediate, class:^(cs2)$
windowrulev2 = immediate, class:^(steam_app_).*$

# Opacity rules
windowrulev2 = opacity 0.9 0.9, class:^(kitty)$
windowrulev2 = opacity 0.95 0.95, class:^(code)$
windowrulev2 = opacity 0.9 0.9, class:^(dolphin)$

# Workspace assignments
windowrulev2 = workspace 2, class:^(firefox)$
windowrulev2 = workspace 3, class:^(code)$
windowrulev2 = workspace 4, class:^(discord)$
windowrulev2 = workspace 5, class:^(steam)$

# Size and position rules
windowrulev2 = size 800 600, class:^(pavucontrol)$
windowrulev2 = center, class:^(pavucontrol)$

# Layer rules for overlays
layerrule = blur, waybar
layerrule = blur, rofi
layerrule = blur, notifications
layerrule = ignorezero, notifications
```

**What these rules do:**
- **Floating windows**: Makes utility windows float instead of tile
- **Picture-in-picture**: Pins and positions video overlays
- **Gaming**: Enables immediate mode for low latency
- **Opacity**: Makes terminals and editors slightly transparent
- **Workspace assignments**: Auto-assigns apps to specific workspaces
- **Layer rules**: Configures blur and behavior for overlays

### Color Configuration

Create `~/.config/hypr/hyprland/colors.conf`:

```bash
# Color configuration (automatically managed by Quickshell)
# These colors are generated from your wallpaper by the Material color system

# This file is automatically updated by Quickshell's color generation system
# Manual changes will be overwritten when wallpaper changes

# The colors follow Material Design 3 specifications
# and are generated to be accessible and cohesive

# Primary colors
$primary = rgb(0DB7D4)
$onPrimary = rgb(003238)
$primaryContainer = rgb(004A52)
$onPrimaryContainer = rgb(69F0FF)

# Secondary colors  
$secondary = rgb(B0CCCE)
$onSecondary = rgb(1B3437)
$secondaryContainer = rgb(324B4E)
$onSecondaryContainer = rgb(CCE8EA)

# Tertiary colors
$tertiary = rgb(B3C7EA)
$onTertiary = rgb(1D3048)
$tertiaryContainer = rgb(344860)
$onTertiaryContainer = rgb(D1E4FF)

# Error colors
$error = rgb(FFB4AB)
$onError = rgb(690005)
$errorContainer = rgb(93000A)
$onErrorContainer = rgb(FFDAD6)

# Surface colors
$surface = rgb(0E1415)
$onSurface = rgb(DDE3E4)
$surfaceVariant = rgb(3F484A)
$onSurfaceVariant = rgb(BFC8CA)
$surfaceDim = rgb(0E1415)
$surfaceBright = rgb(343A3B)
$surfaceContainerLowest = rgb(090F10)
$surfaceContainerLow = rgb(161D1E)
$surfaceContainer = rgb(1A2122)
$surfaceContainerHigh = rgb(252B2C)
$surfaceContainerHighest = rgb(303637)

# Background
$background = rgb(0E1415)
$onBackground = rgb(DDE3E4)

# Window border colors using the generated palette
col.active_border = $primary
col.inactive_border = rgba(31313600)
## 5. Terminal Configuration (Fish & Kitty)

### Fish Shell Configuration

Create `~/.config/fish/config.fish`:

```fish
# Fish shell configuration for illogical-impulse

# Check if running interactively
if status is-interactive
    # Disable fish greeting message
    set fish_greeting
    
    # Initialize Starship prompt
    starship init fish | source
    
    # Load terminal color sequences if available (generated by Quickshell)
    if test -f ~/.local/state/quickshell/user/generated/terminal/sequences.txt
        cat ~/.local/state/quickshell/user/generated/terminal/sequences.txt
    end
    
    # Useful aliases
    alias pamcan pacman           # Common typo fix
    alias ls 'eza --icons'        # Modern ls replacement with icons
    alias ll 'eza -la --icons'    # Long listing with icons
    alias la 'eza -a --icons'     # Show hidden files
    alias tree 'eza --tree --icons'  # Tree view with icons
    alias clear "printf '\033[2J\033[3J\033[1;1H'"  # True clear screen
    alias q 'qs -c ii'            # Quick Quickshell command
    alias cat 'bat'               # Better cat with syntax highlighting (if available)
    alias grep 'rg'               # Use ripgrep instead of grep
    
    # Git aliases
    alias g git
    alias ga 'git add'
    alias gc 'git commit'
    alias gp 'git push'
    alias gl 'git log --oneline'
    alias gs 'git status'
    alias gd 'git diff'
    
    # System aliases
    alias update 'sudo nixos-rebuild switch'
    alias hm 'home-manager switch'
    alias cleanup 'nix-collect-garbage -d'
    
    # Directory navigation
    alias .. 'cd ..'
    alias ... 'cd ../..'
    alias .... 'cd ../../..'
    
    # Process management
    alias ps 'ps aux | grep'
    alias htop 'htop -C'
    
    # Network
    alias myip 'curl -s https://ipinfo.io/ip'
    alias ports 'ss -tuln'
    
    # Make directories and navigate
    function mkcd
        mkdir -p $argv[1] && cd $argv[1]
    end
    
    # Extract various archive formats
    function extract
        switch $argv[1]
            case '*.tar.bz2'
                tar xjf $argv[1]
            case '*.tar.gz'
                tar xzf $argv[1]
            case '*.bz2'
                bunzip2 $argv[1]
            case '*.rar'
                unrar x $argv[1]
            case '*.gz'
                gunzip $argv[1]
            case '*.tar'
                tar xf $argv[1]
            case '*.tbz2'
                tar xjf $argv[1]
            case '*.tgz'
                tar xzf $argv[1]
            case '*.zip'
                unzip $argv[1]
            case '*.Z'
                uncompress $argv[1]
            case '*.7z'
                7z x $argv[1]
            case '*'
                echo "Unknown archive format: $argv[1]"
        end
    end
    
    # Quick file search
    function ff
        find . -type f -name "*$argv[1]*"
    end
    
    # Weather function
    function weather
        if test -n "$argv[1]"
            curl -s "wttr.in/$argv[1]"
        else
            curl -s "wttr.in"
        end
    end
end

# Environment variables
set -gx EDITOR nvim
set -gx BROWSER firefox
set -gx TERMINAL kitty

# Add ~/.local/bin to PATH if not already there
if not contains ~/.local/bin $PATH
    set -gx PATH ~/.local/bin $PATH
end

# Fish-specific settings
set -g fish_key_bindings fish_default_key_bindings

# Color scheme (will be overridden by terminal color sequences)
set -g fish_color_normal normal
set -g fish_color_command blue
set -g fish_color_quote yellow
set -g fish_color_redirection cyan
set -g fish_color_end green
set -g fish_color_error red
set -g fish_color_param normal
set -g fish_color_selection white --bold --background=brblack
set -g fish_color_search_match bryellow --background=brblack
set -g fish_color_history_current --bold
set -g fish_color_operator green
set -g fish_color_escape cyan
set -g fish_color_cwd green
set -g fish_color_cwd_root red
set -g fish_color_valid_path --underline
set -g fish_color_autosuggestion 555 brblack
set -g fish_color_user brgreen
set -g fish_color_host normal
set -g fish_color_cancel -r
set -g fish_pager_color_completion normal
set -g fish_pager_color_description B3A06D yellow
set -g fish_pager_color_prefix cyan --underline
set -g fish_pager_color_progress brwhite --background=cyan
```

**What this Fish configuration does:**
- **Starship prompt**: Modern, fast shell prompt with git integration
- **Color sequences**: Loads colors generated by Quickshell theme system
- **Useful aliases**: Modern replacements for common commands
- **Custom functions**: Utilities for archives, file search, and navigation
- **Environment setup**: Sets default editor, browser, and PATH
- **Color scheme**: Defines syntax highlighting colors

### Kitty Terminal Configuration

Create `~/.config/kitty/kitty.conf`:

```bash
# Kitty terminal configuration for illogical-impulse

# Font configuration
font_family      JetBrains Mono Nerd Font
bold_font        JetBrains Mono Nerd Font Bold
italic_font      JetBrains Mono Nerd Font Italic
bold_italic_font JetBrains Mono Nerd Font Bold Italic
font_size 11.0

# Font features
disable_ligatures never
font_features JetBrainsMonoNerdFont-Regular +zero +onum

# Cursor configuration
cursor_shape beam
cursor_beam_thickness 1.5
cursor_underline_thickness 2.0
cursor_blink_interval -1
cursor_stop_blinking_after 15.0

# Scrollback
scrollback_lines 10000
scrollback_pager less --chop-long-lines --RAW-CONTROL-CHARS +INPUT_LINE_NUMBER
scrollback_pager_history_size 0
wheel_scroll_multiplier 5.0
touch_scroll_multiplier 1.0

# Mouse
mouse_hide_wait 3.0
url_color #0087BD
url_style curly
open_url_modifiers kitty_mod
open_url_with default
url_prefixes http https file ftp
detect_urls yes
copy_on_select no
strip_trailing_spaces never
rectangle_select_modifiers ctrl+alt
terminal_select_modifiers shift
select_by_word_characters @-./_~?&=%+#

# Performance tuning
repaint_delay 10
input_delay 3
sync_to_monitor yes

# Bell
enable_audio_bell no
visual_bell_duration 0.0
window_alert_on_bell yes
bell_on_tab yes
command_on_bell none

# Window layout
remember_window_size  yes
initial_window_width  640
initial_window_height 400
enabled_layouts *
window_resize_step_cells 2
window_resize_step_lines 2
window_border_width 0.5pt
draw_minimal_borders yes
window_margin_width 0
single_window_margin_width -1
window_padding_width 21.75
placement_strategy center
active_border_color #00ff00
inactive_border_color #cccccc
bell_border_color #ff5a00
inactive_text_alpha 1.0

# Tab bar
tab_bar_edge bottom
tab_bar_margin_width 0.0
tab_bar_style fade
tab_bar_min_tabs 2
tab_activity_symbol none
tab_separator " ┇"
tab_title_template "{title}{' :{}:'.format(num_windows) if num_windows > 1 else ''}"
active_tab_title_template none

# Color scheme (overridden by generated colors)
foreground #dddddd
background #000000
selection_foreground #000000
selection_background #fffacd
cursor #cccccc
cursor_text_color #111111

# The 16 terminal colors
color0 #000000
color1 #cc0403
color2 #19cb00
color3 #cecb00
color4 #001cd1
color5 #cb1ed1
color6 #0dcdcd
color7 #e5e5e5
color8 #4d4d4d
color9 #3e0605
color10 #23fd00
color11 #fffd00
color12 #0026ff
color13 #fd28ff
color14 #14ffff
color15 #ffffff

# Dynamic background opacity
dynamic_background_opacity no
background_opacity 0.95

# Advanced
shell fish
editor nvim
close_on_child_death no
allow_remote_control yes
update_check_interval 24
startup_session none
clipboard_control write-clipboard write-primary
term xterm-kitty

# Keyboard shortcuts
kitty_mod ctrl+shift

# Clipboard
map kitty_mod+c copy_to_clipboard
map kitty_mod+v paste_from_clipboard
map kitty_mod+s paste_from_selection
map shift+insert paste_from_selection
map kitty_mod+o pass_selection_to_program

# Scrolling
map kitty_mod+up scroll_line_up
map kitty_mod+k scroll_line_up
map kitty_mod+down scroll_line_down
map kitty_mod+j scroll_line_down
map kitty_mod+page_up scroll_page_up
map kitty_mod+page_down scroll_page_down
map kitty_mod+home scroll_home
map kitty_mod+end scroll_end
map kitty_mod+h show_scrollback

# Window management
map kitty_mod+enter new_window
map kitty_mod+n new_os_window
map kitty_mod+w close_window
map kitty_mod+] next_window
map kitty_mod+[ previous_window
map kitty_mod+f move_window_forward
map kitty_mod+b move_window_backward
map kitty_mod+` move_window_to_top
map kitty_mod+r start_resizing_window
map kitty_mod+1 first_window
map kitty_mod+2 second_window
map kitty_mod+3 third_window
map kitty_mod+4 fourth_window
map kitty_mod+5 fifth_window
map kitty_mod+6 sixth_window
map kitty_mod+7 seventh_window
map kitty_mod+8 eighth_window
map kitty_mod+9 ninth_window
map kitty_mod+0 tenth_window

# Tab management
map kitty_mod+right next_tab
map kitty_mod+left previous_tab
map kitty_mod+t new_tab
map kitty_mod+q close_tab
map kitty_mod+. move_tab_forward
map kitty_mod+, move_tab_backward
map kitty_mod+alt+t set_tab_title

# Layout management
map kitty_mod+l next_layout

# Font sizes
map kitty_mod+equal change_font_size all +2.0
map kitty_mod+minus change_font_size all -2.0
map kitty_mod+backspace change_font_size all 0

# Miscellaneous
map kitty_mod+f11 toggle_fullscreen
map kitty_mod+f10 toggle_maximized
map kitty_mod+u kitten unicode_input
map kitty_mod+f2 edit_config_file
map kitty_mod+escape kitty_shell window

# Search
map kitty_mod+/ launch --location=hsplit --allow-remote-control kitty +kitten search.py @active-kitty-window-id
```

**What this Kitty configuration does:**
- **Font setup**: Uses JetBrains Mono Nerd Font with ligatures
- **Performance**: Optimized repaint and input delays
- **Visual**: Semi-transparent background with smooth scrolling
- **Shortcuts**: Comprehensive keyboard shortcuts for window/tab management
- **Integration**: Configured to work with Fish shell and the color system

### Starship Prompt Configuration

Create `~/.config/starship.toml`:

```toml
# Starship prompt configuration for illogical-impulse

# Get editor completions based on the config schema
"$schema" = 'https://starship.rs/config-schema.json'

# Inserts a blank line between shell prompts
add_newline = true

# Wait 10 milliseconds for starship to check files under the current directory.
scan_timeout = 10

# Disable the blank line at the start of the prompt
# add_newline = false

# Change command timeout from 500 to 1000 ms
command_timeout = 1000

# Change the default prompt format
format = """
[╭─ ](bold green)$all[╰─ ](bold green)$character"""

# Change the default prompt characters
[character]
success_symbol = "[➜](bold green)"
error_symbol = "[➜](bold red)"

# Shows an icon that should be included by zshrc script based on the distribution or os
[env_var.STARSHIP_DISTRO]
format = '[$env_value](bold white)'
variable = "STARSHIP_DISTRO"
disabled = false

# Shows the username
[username]
style_user = "bold dimmed blue"
show_always = false

[directory]
truncation_length = 3
truncation_symbol = "…/"
home_symbol = " ~"
read_only_style = "197"
read_only = "  "
format = "at [$path]($style)[$read_only]($read_only_style) "

[git_branch]
symbol = " "
format = "on [$symbol$branch]($style) "
truncation_length = 4
truncation_symbol = "…/"
style = "bold green"

[git_status]
format = '[\($all_status$ahead_behind\) ]($style)'
style = "bold green"
conflicted = "🏳"
up_to_date = " "
untracked = " "
ahead = "⇡${count}"
diverged = "⇕⇡${ahead_count}⇣${behind_count}"
behind = "⇣${count}"
stashed = " "
modified = " "
staged = '[++\($count\)](green)'
renamed = "襁 "
deleted = " "

[kubernetes]
format = 'via [ﴱ $context\($namespace\)](bold purple) '
disabled = false

[terraform]
format = "via [ terraform $version]($style) 壟 [$workspace]($style) "

[vagrant]
format = "via [ vagrant $version]($style) "

[docker_context]
format = "via [ $context](bold blue) "

[helm]
format = "via [ $version](bold purple) "

[python]
symbol = " "
python_binary = "python3"

[nodejs]
format = "via [🤖 $version](bold green) "

[ruby]
format = "via [ $version]($style) "

[kubernetes]
format = 'on [ﴱ $context\($namespace\)](dimmed green) '
disabled = false
[kubernetes.context_aliases]
"dev.local.cluster.k8s" = "dev"

[memory_usage]
disabled = false
threshold = -1
symbol = " "
style = "bold dimmed green"
format = "via $symbol [${ram}( | ${swap})]($style) "

[time]
disabled = false
format = '🕙[\[ $time \]]($style) '
time_format = "%T"
utc_time_offset = "local"
style = "bold blue"

[battery]
full_symbol = " "
charging_symbol = " "
discharging_symbol = " "
unknown_symbol = " "
empty_symbol = " "

[[battery.display]]
threshold = 15
style = "bold red"

[[battery.display]]
threshold = 50
style = "bold yellow"

[[battery.display]]
threshold = 80
style = "bold green"

[cmd_duration]
format = "underwent [$duration](bold yellow)"

[jobs]
format = "background [$symbol$number]($style) "
style = "bold red"
symbol = ""

[localip]
ssh_only = false
format = '@[$localipv4](bold red) '
disabled = false

[hostname]
ssh_only = false
format = 'on [$hostname](bold red) '
trim_at = "."
disabled = false

# Disable the package module, hiding it from the prompt completely
[package]
disabled = true
```

**What this Starship configuration does:**
- **Modern prompt**: Clean, informative prompt with git integration
- **Performance**: Fast loading with reasonable timeouts
- **Visual**: Uses nerd font icons and consistent color scheme
- **Information rich**: Shows git status, kubernetes context, battery, etc.
- **Customizable**: Easy to modify sections as needed

## 6. Application Configurations

### Fuzzel (Application Launcher)

Create `~/.config/fuzzel/fuzzel.ini`:

```ini
# Fuzzel application launcher configuration

[main]
# Terminal to launch terminal programs in
terminal=kitty

# Application launcher behavior
layer=overlay
exit-on-keyboard-focus-loss=yes

# Fonts
font=JetBrains Mono Nerd Font:size=12
dpi-aware=yes

# Prompt
prompt="❯ "
icon-theme=Papirus-Dark
icons=yes
show-actions=yes

# Matching
fuzzy=yes
show-match-count=yes
filter-desktop=yes

# Layout
width=50
horizontal-pad=20
vertical-pad=8
inner-pad=8

# Colors (will be overridden by generated theme)
background=000000dd
text=ffffffff
match=00ff00ff
selection=ffffff33
selection-text=000000ff
selection-match=000000ff
border=ffffffff

# Border
border-width=2
border-radius=18

# Key bindings
[key-bindings]
cancel=Escape Control+c
execute=Return KP_Enter
execute-or-next=Tab
cursor-left=Left
cursor-left-word=Control+Left
cursor-right=Right
cursor-right-word=Control+Right
cursor-home=Home
cursor-end=End
delete-prev=BackSpace
delete-prev-word=Control+BackSpace
delete-next=Delete
delete-next-word=Control+Delete
extend-to-word-boundary=Control+w
execute-or-next=Tab
prev=Up Control+p
next=Down Control+n
```

### Foot Terminal (Alternative)

If you prefer Foot terminal, create `~/.config/foot/foot.ini`:

```ini
# Foot terminal configuration (alternative to Kitty)

[main]
shell=fish
term=foot
login-shell=no
app-id=foot
title=foot
locked-title=no

[bell]
urgent=no
notify=no
command=
command-focused=no

[scrollback]
lines=10000
multiplier=3.0
indicator-position=relative
indicator-format=

[url]
launch=xdg-open ${url}
label-letters=sadfjklewcmpgh
osc8-underline=url-mode
protocols=http, https, ftp, ftps, file, gemini, gopher
uri-characters=abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_.,~:;/?#@!$&%*+="'()[]

[cursor]
style=beam
color=111111 cccccc
blink=no
beam-thickness=1.5
underline-thickness=1

[mouse]
hide-when-typing=no
alternate-scroll-mode=yes

[colors]
alpha=0.95
foreground=dddddd
background=111111

# Normal/regular colors (color palette 0-7)
regular0=222222  # black
regular1=cc241d  # red
regular2=98971a  # green
regular3=d79921  # yellow
regular4=458588  # blue
regular5=b16286  # magenta
regular6=689d6a  # cyan
regular7=a89984  # white

# Bright colors (color palette 8-15)
bright0=928374   # bright black
bright1=fb4934   # bright red
bright2=b8bb26   # bright green
bright3=fabd2f   # bright yellow
bright4=83a598   # bright blue
bright5=d3869b   # bright magenta
bright6=8ec07c   # bright cyan
bright7=ebdbb2   # bright white

[csd]
preferred=server
size=26
font=JetBrains Mono Nerd Font:size=11
color=rgba(16,16,16,0.9)
hide-when-maximized=no
double-click-to-maximize=yes
border-width=0
border-color=rgba(0,0,0,0)
button-width=26
button-color=rgba(0,0,0,0)
button-minimize-color=rgba(0,0,0,0)
button-maximize-color=rgba(0,0,0,0)
button-close-color=rgba(0,0,0,0)

[key-bindings]
scrollback-up-page=Shift+Page_Up
scrollback-up-half-page=none
scrollback-up-line=none
scrollback-down-page=Shift+Page_Down
scrollback-down-half-page=none
scrollback-down-line=none
clipboard-copy=Control+Shift+c XF86Copy
clipboard-paste=Control+Shift+v XF86Paste
primary-paste=Shift+Insert
search-start=Control+Shift+r
font-increase=Control+plus Control+equal Control+KP_Add
font-decrease=Control+minus Control+KP_Subtract
font-reset=Control+0 Control+KP_0
spawn-terminal=Control+Shift+n
minimize=none
maximize=none
fullscreen=F11
pipe-visible=[sh -c "xurls | fuzzel | xargs -r firefox"] none
pipe-scrollback=[sh -c "xurls | fuzzel | xargs -r firefox"] none
pipe-selected=[xargs -r firefox] none
show-urls-launch=Control+Shift+u
show-urls-copy=none

[search-bindings]
cancel=Control+g Control+c Escape
commit=Return
find-prev=Control+r
find-next=Control+s
cursor-left=Left Control+b
cursor-left-word=Control+Left Mod1+b
cursor-right=Right Control+f
cursor-right-word=Control+Right Mod1+f
cursor-home=Home Control+a
cursor-end=End Control+e
delete-prev=BackSpace
delete-prev-word=Mod1+BackSpace Control+BackSpace
delete-next=Delete
delete-next-word=Mod1+d Control+Delete
extend-to-word-boundary=Control+w
extend-to-next-whitespace=Control+Shift+w
extend-line-down=Shift+Down
extend-backward-char=Shift+Left
extend-forward-char=Shift+Right
extend-line-up=Shift+Up

[url-bindings]
cancel=Control+g Control+c Control+d Escape
toggle-url-visible=t
```

## 7. Theme and Styling

### GTK Theme Configuration

Create `~/.config/gtk-3.0/settings.ini`:

```ini
[Settings]
gtk-theme-name=Adw-dark
gtk-icon-theme-name=Papirus-Dark
gtk-font-name=Rubik 11
gtk-cursor-theme-name=Bibata-Modern-Classic
gtk-cursor-theme-size=24
gtk-toolbar-style=GTK_TOOLBAR_BOTH
gtk-toolbar-icon-size=GTK_ICON_SIZE_LARGE_TOOLBAR
gtk-button-images=1
gtk-menu-images=1
gtk-enable-event-sounds=1
gtk-enable-input-feedback-sounds=1
gtk-xft-antialias=1
gtk-xft-hinting=1
gtk-xft-hintstyle=hintfull
gtk-xft-rgba=rgb
gtk-application-prefer-dark-theme=1
```

Create `~/.config/gtk-4.0/settings.ini`:

```ini
[Settings]
gtk-theme-name=Adw-dark
gtk-icon-theme-name=Papirus-Dark
gtk-font-name=Rubik 11
gtk-cursor-theme-name=Bibata-Modern-Classic
gtk-cursor-theme-size=24
gtk-application-prefer-dark-theme=1
```

### Qt Theme Configuration

Create `~/.config/qt5ct/qt5ct.conf`:

```ini
[Appearance]
color_scheme_path=/usr/share/qt5ct/colors/airy.conf
custom_palette=false
icon_theme=Papirus-Dark
standard_dialogs=default
style=Breeze

[Fonts]
fixed=@Variant(\0\0\0@\0\0\0 \0J\0\x65\0t\0\x42\0r\0\x61\0i\0n\0s\0 \0M\0o\0n\0o@&\0\0\0\0\0\0\xff\xff\xff\xff\x5\x1\0K\x10)
general=@Variant(\0\0\0@\0\0\0\x12\0R\0u\0\x62\0i\0k@&\0\0\0\0\0\0\xff\xff\xff\xff\x5\x1\0\x32\x10)

[Interface]
activate_item_on_single_click=1
buttonbox_layout=0
cursor_flash_time=1000
dialog_buttons_have_icons=1
double_click_interval=400
gui_effects=@Invalid()
keyboard_scheme=2
menus_have_icons=true
show_shortcuts_in_context_menus=true
stylesheets=@Invalid()
toolbutton_style=4
underline_shortcut=1
wheel_scroll_lines=3

[SettingsWindow]
geometry=@ByteArray(\x1\xd9\xd0\xcb\0\x3\0\0\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37\0\0\0\0\0\0\0\0\b\0\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37)

[Troubleshooting]
force_raster_widgets=1
ignored_applications=@Invalid()
```

Create `~/.config/qt6ct/qt6ct.conf`:

```ini
[Appearance]
color_scheme_path=/usr/share/qt6ct/colors/airy.conf
custom_palette=false
icon_theme=Papirus-Dark
standard_dialogs=default
style=Breeze

[Fonts]
fixed=@Variant(\0\0\0@\0\0\0 \0J\0\x65\0t\0\x42\0r\0\x61\0i\0n\0s\0 \0M\0o\0n\0o@&\0\0\0\0\0\0\xff\xff\xff\xff\x5\x1\0K\x10)
general=@Variant(\0\0\0@\0\0\0\x12\0R\0u\0\x62\0i\0k@&\0\0\0\0\0\0\xff\xff\xff\xff\x5\x1\0\x32\x10)

[Interface]
activate_item_on_single_click=1
buttonbox_layout=0
cursor_flash_time=1000
dialog_buttons_have_icons=1
double_click_interval=400
gui_effects=@Invalid()
keyboard_scheme=2
menus_have_icons=true
show_shortcuts_in_context_menus=true
stylesheets=@Invalid()
toolbutton_style=4
underline_shortcut=1
wheel_scroll_lines=3

[SettingsWindow]
geometry=@ByteArray(\x1\xd9\xd0\xcb\0\x3\0\0\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37\0\0\0\0\0\0\0\0\b\0\0\0\x2\x80\0\0\x1\x90\0\0\x5\x7f\0\0\x4\x37)

[Troubleshooting]
force_raster_widgets=1
ignored_applications=@Invalid()
```

## 8. Final Steps

### Apply All Configurations

Rebuild your NixOS system and Home Manager:

```bash
# Rebuild NixOS system configuration
sudo nixos-rebuild switch

# Apply Home Manager configuration
home-manager switch

# Refresh font cache
fc-cache -fv
```

### Start Hyprland

1. **Logout** from your current session
2. **Select Hyprland** from your display manager (not "Hyprland with UWSM")
3. **Login** to start your new setup

### Initial Setup

Once logged in:

1. **Test basic functionality**:
   - Press `Super` to open the overview/launcher
   - Press `Super+Enter` to open terminal
   - Press `Super+/` to view keybinds

2. **Set a wallpaper**:
   - Press `Super+Ctrl+T` to open wallpaper selector
   - Or manually set one: `hyprpaper` or through the Quickshell interface

3. **Configure monitors** (if using multiple displays):
   - Edit `~/.config/hypr/monitors.conf`
   - Reload: `hyprctl reload`

4. **Test applications**:
   - Terminal: `Super+Enter`
   - File manager: `Super+E`
   - Browser: `Super+W`

### Customization

To customize the setup:

1. **Hyprland settings**: Edit files in `~/.config/hypr/custom/`
2. **Keybindings**: Add to `~/.config/hypr/custom/keybinds.conf`
3. **Startup apps**: Add to `~/.config/hypr/custom/execs.conf`
4. **Window rules**: Add to `~/.config/hypr/custom/rules.conf`

### Troubleshooting

If something doesn't work:

1. **Check logs**: `journalctl --user -f`
2. **Hyprland logs**: Check TTY output or `hyprctl log`
3. **Rebuild**: `sudo nixos-rebuild switch` and `home-manager switch`
4. **Restart**: Logout and login again

## What This Setup Provides

- **Modern Wayland desktop** with Hyprland compositor
- **Material Design theming** with automatic color generation
- **Efficient workflow** with keyboard-driven navigation
- **Beautiful visuals** with blur effects and smooth animations
- **Terminal-focused** environment with Fish shell and modern tools
- **Extensible configuration** that's easy to customize

The setup is minimal but complete, providing a solid foundation that you can extend based on your needs. The modular configuration makes it easy to modify individual components without affecting the rest of the system.

Enjoy your new NixOS Hyprland setup! 🎉
```
