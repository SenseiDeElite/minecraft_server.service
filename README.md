<h1 align="center">minecraft_server.service</h1>

<p align="center">
  <a href="https://github.com/SenseiDeElite/minecraft_server.service/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-D9D9D9?style=for-the-badge&labelColor=1a1a1a&logo=opensourceinitiative&logoColor=white" alt="License: MIT" />
  </a>
</p>

A hardened `systemd` unit file for running a Minecraft server, combining strict sandboxing (filesystem, privilege, syscall, and network lockdown) with JVM performance tuning.

## Features

- **Sandboxing**: `DynamicUser`, `ProtectSystem=strict`, `ProtectHome`, `PrivateDevices`, `ProtectKernelModules`, and related directives to minimize the service's access to the host system.
- **Privilege lockdown**: `NoNewPrivileges`, empty `CapabilityBoundingSet`/`AmbientCapabilities`, `RestrictSUIDSGID`, `RestrictNamespaces`, and `LockPersonality` to reduce the attack surface if the JVM or a plugin is compromised.
- **Syscall filtering**: `SystemCallFilter` restricted to `@system-service` with dangerous syscall groups explicitly excluded.
- **Network lockdown**: `RestrictAddressFamilies=AF_INET` to limit the service to IPv4 sockets only.
- **JVM performance flags**: G1GC tuning and JIT/vectorization flags aimed at reducing GC pause times and improving throughput.

## Requirements

- A Linux distribution with [`systemd`](https://github.com/systemd/systemd).
- [JRE (Java Runtime Environment)](https://openjdk.org/); can be headless.
- A Minecraft server jar ([vanilla](https://www.minecraft.net/en-us/download/server), [Fabric](https://fabricmc.net/use/server/), etc. — not included).

## Installation

1. Place your server jar at `/var/lib/minecraft/jar` (or edit the `ExecStart` path to match).
2. Copy the unit file:
   ```bash
   run0 cp --reflink=auto minecraft_server.service /etc/systemd/system/
   ```
3. Reload systemd and enable the service:
   ```bash
   run0 systemctl daemon-reload
   run0 systemctl enable --now minecraft_server.service
   ```
4. Check status/logs:
   ```bash
   systemctl status minecraft_server.service
   ```

## Notes

- `DynamicUser=yes` means the service runs as an ephemeral, auto-allocated user — no need to create a `minecraft` user manually.
- If you use a mod loader or plugin ecosystem that needs broader filesystem or network access than what's granted here, you may need to relax some of the lockdown directives (in particular `ProtectSystem`, `ReadWritePaths`, or `RestrictAddressFamilies`).
- Server performance monitoring is restricted and may require additional tweaking.
- This is provided as-is; test in a non-production environment before relying on it for a live server.

## Recommended tweaks

- `-Xmx4G` — adjust to your available RAM.
- `-jar jar` — the installed jar file's name; in this example it's literally named `jar`, so update it to match your own server jar's filename.

## Testing

- Verified against a modded Fabric server (vanilla extended, with reputable optimization mods).
- Hardware acceleration should work but may require additional tweaking depending on the native library in use.
- Tested mainly with OpenJDK, since that's what's widely available on Linux. GraalVM may also work.
- Deprecated and GraalVM-only JVM flags were stripped from the unit.

## Credits

The JVM performance flags are adapted from [MeowIce/meowice-flags](https://github.com/MeowIce/meowice-flags).
