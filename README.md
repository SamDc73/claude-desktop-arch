# Claude Desktop for Linux (Arch Linux)

A native-like Linux port of Claude Desktop, packaged for Arch Linux.

This project was inspired by [claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake), [claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) and [LaurenceGuws's commit](https://github.com/aaddrick/claude-desktop-debian/pull/18).

## Features
- MCP protocol support 
- 🚀 Native desktop integration
- 🎨 Proper theme integration (KDE/GNOME)
- 📁 XDG base directory compliance

## Quick Install

```bash
# Clone the repository
git clone https://github.com/yourusername/claude-desktop-arch
cd claude-desktop-arch

# Build and install
makepkg -si
```

That's it! Launch Claude Desktop from your application menu or run ```claude-desktop``` in terminal.

## Optional Enhancements

### For KDE users:
```bash
sudo pacman -S kvantum xdg-desktop-portal-kde
```

### For GNOME users:
```bash
sudo pacman -S xdg-desktop-portal-gnome gtk3
```

## Troubleshooting

### Clean Install
If you're upgrading from an older version:
```bash
# Remove old package
sudo pacman -R claude-desktop

# Reinstall
makepkg -si
```

### Common Issues

**Q: App doesn't start?**  
A: Make sure electron and asar are installed:
```bash
sudo pacman -S electron
npm i -g asar
```

**Q: UI looks wrong?**  
A: Install the appropriate theme integration packages for your desktop environment (see Optional Enhancements above)

## Updates
Check [Claude.ai](https://claude.ai/desktop) for the latest version. Update the PKGBUILD if needed.

---

Not affiliated with Anthropic.

