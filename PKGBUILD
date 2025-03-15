pkgname=claude-desktop
pkgver=0.8.1
pkgrel=1
pkgdesc="Claude Desktop for Linux – an AI assistant from Anthropic"
arch=('x86_64')
url="https://github.com/aaddrick/claude-desktop-debian"
license=('custom')
depends=('nodejs' 'npm' 'electron' 'p7zip' 'icoutils' 'imagemagick')
makedepends=('wget')
source=("https://storage.googleapis.com/osprey-downloads-c02f6a0d-347c-492b-a752-3e0651722e97/nest-win-x64/Claude-Setup-x64.exe")
sha256sums=('SKIP')

build() {
  cd "$srcdir"
  mkdir -p build
  cd build

  # Copy the downloaded installer into our build dir
  cp "$srcdir/Claude-Setup-x64.exe" .

  echo "Extracting Windows installer..."
  7z x -y Claude-Setup-x64.exe || { echo "Extraction failed"; exit 1; }

  # The installer contains a .nupkg file named using the version (e.g. AnthropicClaude-0.7.9-full.nupkg)
  echo "Extracting nupkg..."
  7z x -y "AnthropicClaude-${pkgver}-full.nupkg" || { echo "nupkg extraction failed"; exit 1; }

  echo "Processing icons..."
  # Extract icons from the exe (wrestool and icotool come from icoutils)
  wrestool -x -t 14 "lib/net45/claude.exe" -o claude.ico || { echo "wrestool failed"; exit 1; }
  icotool -x claude.ico || { echo "icotool failed"; exit 1; }

  echo "Preparing Electron app..."
  mkdir -p electron-app
  cp "lib/net45/resources/app.asar" electron-app/
  cp -r "lib/net45/resources/app.asar.unpacked" electron-app/

  cd electron-app
  # Extract the asar package to allow modifications
  npx asar extract app.asar app.asar.contents || { echo "asar extract failed"; exit 1; }

  echo "Creating stub native module..."
  mkdir -p app.asar.contents/node_modules/claude-native
  cat > app.asar.contents/node_modules/claude-native/index.js << 'EOF'
  // Stub implementation of claude-native using KeyboardKey enum values
  const KeyboardKey = {
    Backspace: 43,
    Tab: 280,
    Enter: 261,
    Shift: 272,
    Control: 61,
    Alt: 40,
    CapsLock: 56,
    Escape: 85,
    Space: 276,
    PageUp: 251,
    PageDown: 250,
    End: 83,
    Home: 154,
    LeftArrow: 175,
    UpArrow: 282,
    RightArrow: 262,
    DownArrow: 81,
    Delete: 79,
    Meta: 187
  };
  Object.freeze(KeyboardKey);
  module.exports = {
    getWindowsVersion: () => "10.0.0",
    setWindowEffect: () => {},
    removeWindowEffect: () => {},
    getIsMaximized: () => false,
    flashFrame: () => {},
    clearFlashFrame: () => {},
    showNotification: () => {},
    setProgressBar: () => {},
    clearProgressBar: () => {},
    setOverlayIcon: () => {},
    clearOverlayIcon: () => {},
    // Disable tray functionality
    createTray: () => {},
    destroyTray: () => {},
    updateTrayMenu: () => {},
    KeyboardKey
  };
EOF


  echo "Copying tray icons..."
  mkdir -p app.asar.contents/resources
  cp ../lib/net45/resources/Tray* app.asar.contents/resources/

  echo "Repacking asar..."
  npx asar pack app.asar.contents app.asar || { echo "asar pack failed"; exit 1; }

  # Return to the build directory root
  cd "$srcdir/build"
}

package() {
  cd "$srcdir/build"

  echo "Installing files..."
  # Create installation directories
  install -d "$pkgdir/usr/lib/claude-desktop"
  install -d "$pkgdir/usr/share/applications"
  install -d "$pkgdir/usr/share/icons/hicolor"
  install -d "$pkgdir/usr/bin"
  install -d "$pkgdir/etc/claude-desktop"

  # Create translation file with all required strings
  cat > "$pkgdir/etc/claude-desktop/en-US.json" << 'EOF'
{
  "EfdnINFnIz": "File",
  "baGq3gy8z1": "New Conversation",
  "ZJZN1+KyJw": "Settings...",
  "7fdcqxofEs": "Exit",
  "/PgA81GVOD": "Edit",
  "fFJxOwJRj2": "Undo",
  "3ML3xT+gEV": "Redo",
  "TH+W2Ad73P": "Cut",
  "3unrKzH4zB": "Copy",
  "KAo3lt5Hv+": "Paste",
  "8YQEOfuaGO": "Select All",
  "O3rtEd7aMd": "Find",
  "LCWUQ/4Fu6": "View",
  "m3GfpKD1WX": "Reload",
  "arbRxbtBkP": "Back",
  "PH29MShDiy": "Forward",
  "+/cwsayrqk": "Actual Size",
  "Z9g5m/V9Nq": "Zoom In",
  "fEeEFfSz4K": "Zoom In (indie cooler version)",
  "XZ36+EBE5/": "Zoom Out",
  "pWXxZASpOB": "Help",
  "vgLHPxjh9O": "Enable Developer Mode",
  "j66cdL4EK5": "Open Documentation",
  "mRXjxhS6p4": "Check for Updates…",
  "3gG1j3kRBX": "Submit Feedback...",
  "DQTgg21B7g": "Show App",
  "dKX0bpR+a2": "Quit"
}
EOF

  # Install the Electron app (app.asar and its unpacked resources)
  cp electron-app/app.asar "$pkgdir/usr/lib/claude-desktop/"
  cp -r electron-app/app.asar.unpacked "$pkgdir/usr/lib/claude-desktop/"
  echo "Installing icons..."
  # Map icon sizes to the extracted filenames
  for size in 16 24 32 48 64 256; do
    icon_file=""
    case "$size" in
      16) icon_file="claude_13_16x16x32.png" ;;
      24) icon_file="claude_11_24x24x32.png" ;;
      32) icon_file="claude_10_32x32x32.png" ;;
      48) icon_file="claude_8_48x48x32.png" ;;
      64) icon_file="claude_7_64x64x32.png" ;;
      256) icon_file="claude_6_256x256x32.png" ;;
    esac
    if [ -f "$srcdir/build/$icon_file" ]; then
      install -Dm644 "$srcdir/build/$icon_file" "$pkgdir/usr/share/icons/hicolor/${size}x${size}/apps/claude-desktop.png"
    else
      echo "Warning: Missing ${size}x${size} icon"
    fi
  done

  echo "Creating desktop entry..."
  cat > "$pkgdir/usr/share/applications/claude-desktop.desktop" << 'EOF'
[Desktop Entry]
Name=Claude
Name=Claude
Exec=claude-desktop %u
Icon=claude-desktop
Type=Application
Terminal=false
Categories=Office;Utility;
MimeType=x-scheme-handler/claude;
StartupWMClass=Claude
EOF

  cat > "$pkgdir/usr/bin/claude-desktop" << 'EOF'
#!/bin/bash

# Set XDG directories properly
export XDG_CONFIG_HOME="${XDG_CONFIG_HOME:-$HOME/.config}"
export XDG_CACHE_HOME="${XDG_CACHE_HOME:-$HOME/.cache}"
export XDG_DATA_HOME="${XDG_DATA_HOME:-$HOME/.local/share}"
export XDG_STATE_HOME="${XDG_STATE_HOME:-$HOME/.local/state}"

# Set Electron specific paths
export ELECTRON_CONFIG_DIR="$XDG_CONFIG_HOME/claude-desktop"
export ELECTRON_CACHE_DIR="$XDG_CACHE_HOME/claude-desktop"
export ELECTRON_DATA_DIR="$XDG_DATA_HOME/claude-desktop"
export ELECTRON_STATE_DIR="$XDG_STATE_HOME/claude-desktop"

# Create directories if they don't exist
mkdir -p "$ELECTRON_CONFIG_DIR"
mkdir -p "$ELECTRON_CACHE_DIR"
mkdir -p "$ELECTRON_DATA_DIR"
mkdir -p "$ELECTRON_STATE_DIR"

# Find the electron installation
ELECTRON_PATH=$(which electron)
if [ -z "$ELECTRON_PATH" ]; then
    echo "Electron not found. Please install electron"
    exit 1
fi

# Detect desktop environment
if [ -n "$XDG_CURRENT_DESKTOP" ]; then
    DE="${XDG_CURRENT_DESKTOP,,}" # Convert to lowercase
    echo "Detected desktop environment: $XDG_CURRENT_DESKTOP"
else
    DE="unknown"
    echo "Could not detect desktop environment"
fi

# Desktop environment specific settings
if [[ "$DE" == *"kde"* ]]; then
    echo "Applying KDE-specific settings"
    export QT_QPA_PLATFORMTHEME=kde
    export QT_STYLE_OVERRIDE=kvantum
elif [[ "$DE" == *"gnome"* ]]; then
    echo "Applying GNOME-specific settings"
    export QT_QPA_PLATFORMTHEME=gnome
    export GTK_USE_PORTAL=1
fi

# Universal settings
export ELECTRON_FORCE_WINDOW_PLATFORM=1
export ELECTRON_ENABLE_HARDWARE_ACCELERATION=1
export ELECTRON_ENABLE_SMOOTH_SCROLLING=1

# Set Electron app specific flags
export ELECTRON_FLAGS=(
    --user-data-dir="$ELECTRON_CONFIG_DIR"
    --cache-path="$ELECTRON_CACHE_DIR"
    --no-sandbox
)

# HiDPI settings
if [ -n "$SCALE_FACTOR" ]; then
    export ELECTRON_FORCE_DEVICE_SCALE_FACTOR=$SCALE_FACTOR
else
    export ELECTRON_FORCE_DEVICE_SCALE_FACTOR=0
    export QT_AUTO_SCREEN_SCALE_FACTOR=1
fi

# Create symlink to en-US.json if it doesn't exist
if [ ! -f "/usr/lib/electron/resources/en-US.json" ]; then
    mkdir -p /usr/lib/electron/resources
    ln -sf /etc/claude-desktop/en-US.json /usr/lib/electron/resources/en-US.json
fi

# Launch the application with proper paths
electron "${ELECTRON_FLAGS[@]}" /usr/lib/claude-desktop/app.asar "$@"
EOF

  chmod +x "$pkgdir/usr/bin/claude-desktop"
}


post_install() {
  # Ensure electron resources directory exists
  mkdir -p /usr/lib/electron/resources

  # Create symlink to en-US.json
  ln -sf /etc/claude-desktop/en-US.json /usr/lib/electron/resources/en-US.json

  echo "==> For the best experience:"
  echo "    - If using KDE:"
  echo "      * Install Kvantum theme engine: 'sudo pacman -S kvantum'"
  echo "      * Install xdg-desktop-portal-kde for better integration"
  echo "    - If using GNOME:"
  echo "      * Install xdg-desktop-portal-gnome for better integration"
  echo "    - If using Wayland:"
  echo "      * Install qt6-wayland for better Wayland support"
  echo "    - Enable 'Allow applications to set cursor size' in your display settings"
}
