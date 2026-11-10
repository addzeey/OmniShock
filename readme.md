# OmniShock

OmniShock connects local R.E.P.O. damage, deaths, and valuable losses to PiShock V3 or OpenShock devices.

Only your local gameplay can trigger your configured device. Other players do not need the mod installed. Each player who wants device integration installs and configures their own copy.

## Features

- Supports PiShock V3 and OpenShock, including compatible HTTPS self-hosted OpenShock servers.
- Separate damage, death, and valuable-loss triggers with configurable intensity and duration.
- Optional proportional intensity based on the fraction of a valuable's value lost.
- Configurable held/released-item responsibility and per-item enemy-hit protection.
- In-game settings from the main, lobby, and Escape menus, with live Apply and immediate Disable.
- Hidden saved credentials, masked replacement fields, and settings for only the selected provider.
- Global intensity cap, cooldown, and single-request protection without automatic retries.

## Requirements

- R.E.P.O.
- [BepInExPack 5.4.2305](https://thunderstore.io/c/repo/p/BepInEx/BepInExPack/).
- [MenuLib 2.6.2](https://thunderstore.io/c/repo/p/REPOModding/MenuLib/).
- A configured PiShock V3 or OpenShock device for physical shocks.

A Thunderstore-compatible mod manager installs the declared dependencies automatically. Avoid enabling multiple mods that send device requests for the same gameplay events.

## Quick Start

1. Install OmniShock with Gale, r2modman, or another R.E.P.O.-compatible mod manager.
2. Launch R.E.P.O. once to create `BepInEx/config/com.github.addzeey.OmniShock.cfg` in the active profile.
3. Open **OmniShock** from the main, lobby, or Escape menu.
4. Leave **Enabled** off. Choose **PiShock** or **OpenShock**, then enter that provider's API key and shocker/device ID.
5. Adjust triggers, intensity limits, durations, and cooldown. Start with low intensity and short durations.
6. Click **Apply** to save. When ready for physical device operation, turn **Enabled** on, Apply, and confirm.

For manual installation, install both dependencies first and copy the package's `plugins/OmniShock` folder into your profile's `BepInEx/plugins` folder. Keep `addzeey.OmniShock.dll`, `OmniShock.Core.dll`, and `Newtonsoft.Json.dll` together. Restart after installing or updating; settings changed through Apply do not need a restart.

## Provider Setup

`General.Provider` defaults to `PiShock`. Selecting a provider shows only its settings. Switching preserves unsaved drafts without saving or sending requests.

### PiShock

**PiShock support uses `https://api.pishock.com`. Hardware must use PiShock V3 for this API.**

Configure these fields in the **PiShock menu section**. Only `ApiUrl` is a normal `.cfg` setting; the key and ID are saved separately.

| Setting | Value |
| --- | --- |
| `ApiUrl` | Keep the default `https://api.pishock.com`. |
| `ApiKey` | Your PiShock API key. |
| `ShockerId` | Your PiShock V3 shocker ID, not a legacy share code. |

- [PiShock V3 documentation](https://docs.pishock.com/pishock/pishock-v3-documentation.html)
- [Shocker ID lookup](https://pishock-lookup.addzeey.com/)

### PiShock Partner Discount

Shop at the [PiShock store](https://store.pishock.com/) and use promo code `ADDZEEY` for 10% off.

### OpenShock

Configure these fields in the **OpenShock menu section**. Only `ApiUrl` is a normal `.cfg` setting; the key and ID are saved separately.

| Setting | Value |
| --- | --- |
| `ApiUrl` | Keep `https://api.openshock.app` unless using another compatible HTTPS server. |
| `ApiKey` | Your OpenShock API token. |
| `DeviceId` | Your OpenShock shocker or share ID. |

Create an API token and copy the appropriate ID from your OpenShock account.

## Gameplay Triggers

Each trigger section has its own `Enabled` setting. These default to enabled, but **General.Enabled defaults to false** and must be enabled before any requests can be sent.

| Event | Section | Intensity | Default duration |
| --- | --- | --- | --- |
| Health damage | `Damage` | Actual health lost × `IntensityPerHealthLost` (default `1`), rounded up and capped. | `500` ms |
| Local death | `Death` | `Intensity` (default `25%`), limited by the global cap. | `1000` ms |
| Valuable damage or destruction | `Valuables` | Value lost × `ValueLossMultiplier` (default `0.1`), rounded up and capped. | `500` ms |

Damage uses actual health lost, not attempted damage. Invincibility, rejected hits, and zero damage do not trigger. A lethal hit uses Death only; disabling Death does not fall back to a damage shock. Remote players' damage and deaths do not trigger your device.

Valuable losses count while you hold an item or within its release window. Destruction counts the remaining lost value. Non-valuables and zero-loss breaks are ignored. Tutorial and mission levels are supported; shops, lobbies, menus, recording, arenas, and loading states do not trigger shocks.

## Valuable Settings

| Setting | Default | Behavior |
| --- | --- | --- |
| `MapValueToIntensity` | `false` | Use lost value / original value × global maximum instead of the loss multiplier. |
| `ValueLossMultiplier` | `0.1` | For example, 200 value lost produces 20% before the global cap. |
| `ResponsibilityWindowSeconds` | `15` | How long a released valuable remains associated with you. |
| `IgnoreHittingEnemies` | `true` | Suppress loss on the same valuable used offensively against an enemy. |
| `ThrownOffensiveGraceSeconds` | `2` | Released items must hit an enemy within this interval to qualify. |
| `HitOffensiveGraceSeconds` | `3` | Suppress further loss on that item after qualifying enemy contact. |

Enemy protection does not suppress player health damage or losses on unrelated valuables. Handing an item to another player does not erase your release window. Delayed network/collision information can miss enemy-hit protection, especially as a non-host.

## Intensity, Duration, And Cooldown

- `General.MaxIntensity` defaults to **25%** and caps every trigger, including Death. Range: `1–100`.
- `General.CooldownSeconds` defaults to **2 seconds**. Range: `0–60`; `0` disables the cooldown. Switching providers does not reset it.
- `DurationMilliseconds` is measured in **milliseconds**: `500` is half a second; `1000` is one second. Config range: `300–15000`.
- Menu sliders use **1%** intensity steps, **500 ms** duration steps, and **0.5** decimal steps. Existing off-step values remain available until edited; opening the menu does not round your config.
- Numeric arrows stop at the minimum/maximum instead of wrapping.

## In-Game Settings And Credentials

Changes remain drafts until **Apply**. Applying enabled settings requires complete credentials and hardware confirmation. Apply automatically pauses shocks, cancels pending work where possible, and saves a Disabled checkpoint before updating the separate credential file. After every save succeeds, it restores your selected Enabled state; if you selected off, it stays off. Failed saves leave shocks Disabled. If credentials saved but resuming failed, the status says so rather than claiming the credentials rolled back. No test shock is sent and cooldown is preserved. **Back/Escape** asks before discarding changes. **Disable** blocks new requests immediately, independent of unsaved edits.

Saved API keys and device IDs never populate input fields. Enter a replacement in the blank field; the visible text stays masked. Credential actions are side by side:

| Action | Behavior |
| --- | --- |
| **Paste** | Stage a replacement from the clipboard. |
| **Keep saved** | Discard a pending replacement or clear. An empty field also keeps the saved value. |
| **Clear saved** | Confirm removal, then Apply to save it. Disable shocks before clearing the active provider's credentials. |

Replacement input clears after saving or closing. Copy/cut are blocked for credential fields. Configured/hidden status means a value is saved, not that authentication or hardware delivery was verified. Opening or saving settings never sends a test shock or authentication request.

## Request Safeguards And Logs

### Credential Storage

API keys and device/share IDs are stored outside the game and mod-manager profile at **`%LOCALAPPDATA%\OmniShock\REPO\credentials.json`** on Windows. Other platforms use .NET's per-user local application-data directory. All profiles for the same OS user share this store; ordinary profile/config exports do not include it. Use the in-game menu, not the `.cfg`

Credentials are **not imported from old config files**. Enter them through the in-game menu if the private store is not configured. Any retired API-key/device-ID entries found in `.cfg` are discarded and removed on startup to prevent export leaks; shocks are disabled when such values are found. Existing private-store credentials remain unchanged. If storage or config cleanup fails, initialization stops; fix permissions and restart before sharing the profile.

The separate file is **plaintext, not encrypted**. Do not share it; other programs running as your OS user may read it. Old profile exports/backups can still contain previous keys—revoke exposed keys, because migration cannot clean copies already shared. Switching OS accounts or PCs requires fresh setup. A corrupt/unreadable private store is preserved rather than silently overwritten.

- Only one provider request can be outstanding. Busy/cooldown events are skipped, not queued for later.
- Requests time out after 10 seconds. Failures are not automatically retried; redirects are not followed.
- Missing credentials prevent requests. Scene/config changes clear pending item observations and cancel outstanding work where possible.
- Logs are written to `BepInEx/LogOutput.log`. `Sent` means HTTP acceptance, not confirmed physical delivery. `Busy` and `Cooldown` are intentional skips; `Failed` or `Cancelled` can be ambiguous if a provider already accepted the command.

## Safety And Privacy

> **OmniShock controls physical hardware. Use it responsibly, follow the manufacturer's safety guidance, start with low intensity and short duration, and stop immediately if you feel uncomfortable.**

- Use only with informed consent and keep hardware stop controls accessible.
- Disabling cannot recall a command already accepted by a provider.
- Do not share the private credentials file or old configs/exports that still contain keys. Menu masking is not encryption, and pasting does not clear your OS clipboard.
- Device/provider limits may be lower than the mod's configured limits.
- OmniShock is provided as-is. You are responsible for device configuration and use.
