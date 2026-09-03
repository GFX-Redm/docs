# RedM Script Documentation

Welcome to the GFX RedM Scripts documentation. This collection provides high-quality, multi-framework compatible scripts for RedM (Red Dead Redemption 2 multiplayer) servers.

## Supported Frameworks

All GFX scripts work with multiple frameworks through our unified bridge system:

- **VORP Core** - The most popular RedM framework
- **RSG Framework** - QBCore-inspired framework for RedM
- **RedEM:RP** - Alternative RedM roleplay framework

## Getting Started

1. Install `gfx-bridge` resource on your server
2. Ensure your framework (VORP/RSG/RedEM) is running
3. Install any GFX script - it will auto-detect your framework

## Architecture

```
gfx-bridge (required)
    |
    ├── gfx-script-1 (depends on gfx-bridge)
    ├── gfx-script-2 (depends on gfx-bridge)
    └── gfx-script-N (depends on gfx-bridge)
```

Every script communicates with your framework through `gfx-bridge`, ensuring compatibility and easy maintenance.

## Support

- Discord: [GFX Development](https://discord.gg/gfx)
- Documentation: This GitBook
- Tebex Store: [gfx.tebex.io](https://gfx.tebex.io)
