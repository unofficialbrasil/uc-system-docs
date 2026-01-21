# UC World Admin Controls

**System:** Unofficial Communities
**Last Updated:** 2026-01-19
**Version:** 1.0.0
**Component:** uc-world (3D Virtual World)

---

## 1. Overview

This document specifies the administrative camera controls and debug tools available to admin users in UC World. These controls enable community managers and platform administrators to monitor, debug, and manage virtual world spaces effectively.

**Target Audience:**
- Platform administrators
- Community moderators
- Development team
- QA testers

---

## 2. Admin Authentication

### 2.1 Identity Resolution

UC World supports two access methods:

| Access Method | URL | Identity Source |
|---------------|-----|-----------------|
| **Dashboard Embed** | `/world/[tag]` | URL params (`?identityId=X&username=Y`) |
| **Direct Access** | `world.unofficialcommunities.com.br` | Session cookie via API |

**Identity Resolution Flow (main.ts):**

1. First check URL parameters (iframe embed from dashboard)
2. If no URL params, fetch from frontend BFF `GET /api/v1/user/me` using session cookie
3. If both fail, allow anonymous access (no admin features)

```typescript
// Identity resolution priority
const urlIdentity = getUserIdentityFromUrl();
if (urlIdentity.identityId) return urlIdentity; // Iframe embed

const apiIdentity = await fetchUserIdentityFromApi(); // Direct access via BFF
if (apiIdentity.identityId) return apiIdentity;

return {}; // Anonymous
```

**Technical Requirements for Direct Access:**
- Session cookie must have `domain=.unofficialcommunities.com.br` (set in login/signup routes)
- Frontend BFF must have CORS headers allowing `world.unofficialcommunities.com.br` (set in next.config.mjs)
- User must be logged in to the main site first

**Note:** UC World calls the frontend BFF (not uc-api directly) because the session is managed by the frontend with a different token format.

### 2.2 How Admin Status is Verified

Admin status is verified **server-side** when a user joins a UC World room:

1. Client sends `identityId` in join options
2. Server calls `GET /v1/identities/{id}/admin-status` on uc-api
3. Response determines `isAdmin` and `role` flags
4. Player schema is updated with admin privileges

```typescript
// Server verification (WorldRoom.ts)
if (options.identityId) {
  const adminStatus = await this.verifyAdminStatus(options.identityId);
  player.isAdmin = adminStatus.isAdmin;
  player.role = adminStatus.role;
}
```

### 2.3 Admin Roles

| Role | Permissions |
|------|-------------|
| `user` | Standard player controls only |
| `moderator` | Admin camera + debug overlay |
| `admin` | Full admin controls + ghost mode |

---

## 3. Camera Controls

### 3.1 Activating Admin Mode

| Key | Action | Description |
|-----|--------|-------------|
| **F2** | Toggle Admin Mode | Switches between normal and admin camera |

When admin mode is activated:
- Camera unlocks from player position
- Extended zoom range becomes available
- Ghost mode is automatically enabled

### 3.2 Orbit Camera Controls (Admin Mode)

| Key | Action | Description |
|-----|--------|-------------|
| **Q** | Rotate Left | Orbit camera counter-clockwise around focus |
| **E** | Rotate Right | Orbit camera clockwise around focus |
| **R** | Pitch Up | Tilt camera upward |
| **F** | Pitch Down | Tilt camera downward |
| **Scroll Up** | Zoom In | Move camera closer (min: 2 units) |
| **Scroll Down** | Zoom Out | Move camera farther (max: 50 units) |
| **Home** | Reset View | Return to default isometric angle |

### 3.3 Free-Fly Camera Mode

| Key | Action | Description |
|-----|--------|-------------|
| **G** | Toggle Free-Fly | Switch between orbit and free camera |

**In Free-Fly Mode:**

| Key | Action | Description |
|-----|--------|-------------|
| **W** | Move Forward | Camera moves in facing direction |
| **S** | Move Backward | Camera moves opposite to facing |
| **A** | Strafe Left | Camera moves left |
| **D** | Strafe Right | Camera moves right |
| **Space** | Fly Up | Camera ascends vertically |
| **Shift** | Fly Down | Camera descends vertically |

### 3.4 Zoom Range Comparison

| Mode | Minimum Zoom | Maximum Zoom |
|------|--------------|--------------|
| Normal User | 5 units | 30 units |
| Admin Mode | 2 units | 50 units |

---

## 4. Ghost Mode (Ethical Transparency)

### 4.1 Purpose

Ghost mode ensures that admin presence is **always visible** to other players. This aligns with the platform's ethical principles of transparency and prevents covert surveillance.

### 4.2 Visual Indicators

When an admin enters admin camera mode:

| Indicator | Description |
|-----------|-------------|
| **Semi-Transparent Avatar** | 50% opacity on all materials |
| **Blue Emissive Glow** | Color: `#4a90d9` |
| **[ADMIN] Label** | Visible above player name |

### 4.3 What Other Players See

```
     [ADMIN]
    PlayerName
       👤 (semi-transparent blue glow)
```

### 4.4 Manual Toggle

Admins can manually toggle ghost mode:

| Message Type | Payload |
|--------------|---------|
| `ADMIN_TOGGLE_GHOST_MODE` | `{ enabled: boolean }` |

**Note:** Ghost mode is automatically enabled when entering admin camera mode and cannot be disabled while admin mode is active.

---

## 5. Debug Overlay

### 5.1 Activation

| Key | Action |
|-----|--------|
| **` (backtick)** | Toggle debug overlay |

### 5.2 Debug Panels

#### Tile Coordinates Panel
- Shows (x, y) tile coordinates on mouse hover
- Updates in real-time as cursor moves
- Useful for zone boundary verification

#### Zone Information Panel
- **Zone Name:** Current zone (Hub, Workshop, Lounge, etc.)
- **Privacy Level:** public / semi-private / private
- **Capacity:** Maximum players for zone
- **Current Occupancy:** Live player count

#### Player List Panel
- Lists all players in the room
- Shows: Username, Position (x, y), Current Zone
- Highlights admin players
- Indicates ghost mode status

#### Network Stats Panel
- **Ping:** Round-trip latency (ms)
- **Message Rate:** Messages per second
- **Session ID:** Current Colyseus session
- **Room ID:** Current room identifier

### 5.3 Panel Layout

```
┌─────────────────────────────────────────────┐
│  Tile: (45, 32)                             │  ← Top-left
├─────────────────────────────────────────────┤
│  Zone: Hub                                  │
│  Privacy: public                            │
│  Capacity: 50 / Occupancy: 12               │
├─────────────────────────────────────────────┤
│  Players (12):                              │
│  • PlayerOne (45, 32) [Hub]                 │
│  • AdminUser (40, 46) [Hub] [ADMIN] 👻      │
│  • PlayerTwo (38, 50) [Lounge W]            │
├─────────────────────────────────────────────┤
│  Network:                                   │
│  Ping: 45ms | Rate: 20/s                    │
│  Session: abc123 | Room: world-1            │
└─────────────────────────────────────────────┘
```

---

## 6. Minimap

### 6.1 Overview

The minimap displays the community network visualization showing connected communities in a honeycomb pattern.

### 6.2 Features

| Feature | Description |
|---------|-------------|
| **Current Community** | Highlighted with distinct border |
| **Connected Communities** | Shown as adjacent hexagons |
| **Player Dots** | Aggregate player counts per community |
| **Portal Indicators** | Shows active portal connections |

### 6.3 Interaction

- **Click on adjacent community:** Initiates portal travel request
- **Hover:** Shows community name and player count

---

## 7. Admin Message Types

### 7.1 Client → Server Messages

| Message Type | Payload | Description |
|--------------|---------|-------------|
| `ADMIN_TOGGLE_GHOST_MODE` | `{ enabled: boolean }` | Toggle ghost visibility |

### 7.2 Server → Client Messages

| Message Type | Payload | Description |
|--------------|---------|-------------|
| `ADMIN_STATUS` | `{ isAdmin, role }` | Initial admin status on join |
| `ADMIN_PLAYER_LIST` | `{ players[], timestamp }` | Full player list with positions |

### 7.3 Payload Interfaces

```typescript
interface AdminStatusPayload {
  isAdmin: boolean;
  role: 'user' | 'moderator' | 'admin';
}

interface AdminToggleGhostModePayload {
  enabled: boolean;
}

interface AdminPlayerListPayload {
  players: AdminPlayerInfo[];
  timestamp: number;
}

interface AdminPlayerInfo {
  id: string;
  username: string;
  x: number;
  y: number;
  zone: string | null;
  isAdmin: boolean;
  isGhostMode: boolean;
}
```

---

## 8. Key Source Files

| File | Purpose |
|------|---------|
| `client/src/core/AdminCameraController.ts` | Camera orbit and free-fly logic |
| `client/src/systems/DebugOverlay.ts` | Debug panels and tile tooltip |
| `client/src/ui/Minimap.ts` | Community network visualization |
| `client/src/entities/Player.ts` | Ghost mode rendering |
| `server/src/rooms/WorldRoom.ts` | Admin verification and message handlers |
| `shared/src/messages.ts` | Admin message type definitions |

---

## 9. Keyboard Reference Card

### Quick Reference (Print-Friendly)

```
╔═══════════════════════════════════════════════════════════╗
║              UC WORLD ADMIN CONTROLS                       ║
╠═══════════════════════════════════════════════════════════╣
║  F2          Toggle Admin Mode                             ║
║  ` (backtick) Toggle Debug Overlay                         ║
╠═══════════════════════════════════════════════════════════╣
║  ORBIT CAMERA (Admin Mode):                                ║
║  Q / E       Rotate camera left / right                    ║
║  R / F       Pitch camera up / down                        ║
║  Scroll      Zoom in / out (range: 2-50)                   ║
║  Home        Reset to default view                         ║
╠═══════════════════════════════════════════════════════════╣
║  FREE-FLY CAMERA (Press G to toggle):                      ║
║  W/A/S/D     Move camera                                   ║
║  Space       Fly up                                        ║
║  Shift       Fly down                                      ║
╠═══════════════════════════════════════════════════════════╣
║  Note: Ghost mode auto-enables in admin mode               ║
║  Your avatar becomes semi-transparent with [ADMIN] label   ║
╚═══════════════════════════════════════════════════════════╝
```

---

## 10. Ethical Considerations

### 10.1 Transparency Principles

| Principle | Implementation |
|-----------|----------------|
| **No Hidden Surveillance** | Ghost mode always visible to players |
| **Admin Identification** | [ADMIN] label cannot be hidden |
| **Audit Trail** | Admin actions logged server-side |

### 10.2 Go / No-Go Rules

| GO | NO-GO |
|----|-------|
| Semi-transparent ghost mode | Invisible admin presence |
| Visible [ADMIN] label | Hidden admin identity |
| Aggregate player monitoring | Individual user tracking without consent |
| Debug tools for troubleshooting | Exposing private user data |

---

## 11. Troubleshooting

### 11.1 Admin Mode Not Activating

**Symptom:** F2 does nothing

**Causes:**
1. Identity not verified as admin in database
2. API endpoint `/v1/identities/{id}/admin-status` returning `isAdmin: false`
3. Not authenticated (no `identityId` in join options)

**Solution:** Verify admin status in uc-api database

### 11.2 Ghost Mode Not Rendering

**Symptom:** Avatar appears solid despite admin mode

**Causes:**
1. Material shader not supporting transparency
2. Client-side rendering bug

**Solution:** Check browser console for Three.js errors

### 11.3 Debug Overlay Not Showing

**Symptom:** Backtick key does nothing

**Causes:**
1. Keyboard layout mapping issue
2. Input manager not initialized

**Solution:** Try alternative key or check InputManager initialization

---

## 12. Known Issues

Currently no known issues.

### 12.1 Background Strip at Max Zoom (RESOLVED)

**Status:** ✅ Fixed (2026-01-20)

**Original Symptom:** When using scroll wheel to zoom out to maximum (frustumSize=55), a dark strip appeared at the bottom of the screen that covered the hexagon border lines but not HTML elements (zone labels).

**Root Cause:** The orthographic camera frustum at max zoom extended beyond world geometry, exposing the WebGL clear color.

**Solution:** Screen-space shader quad that renders directly in clip space coordinates, bypassing all camera transformations. This ensures the background always fills the entire viewport regardless of camera position or frustum settings.

**Implementation:** See `uc-world/CLAUDE.md` "Recently Fixed" section and DEC-0012 in Decision Log.

---

## 13. Technical Notes

### 13.1 Why F2 Instead of F1?

The admin mode toggle uses **F2** because F1 is reserved by browsers:
- **Chrome/Edge**: F1 opens browser help
- **Firefox**: F1 opens OS or browser help
- **Safari**: F1 triggers system functions

F2 is not reserved by any major browser, making it safe for application use.

---

## 14. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.4.0 | 2026-01-20 | Background strip issue RESOLVED with screen-space shader quad |
| 1.3.0 | 2026-01-20 | Added Known Issues section documenting background strip at max zoom |
| 1.2.0 | 2026-01-19 | Fixed direct access: uses frontend BFF with CORS, cookie domain sharing |
| 1.1.0 | 2026-01-19 | Added direct access support via session cookie API fetch |
| 1.0.1 | 2026-01-19 | Added debug logging for identity tracking |
| 1.0.0 | 2026-01-19 | Initial documentation (F2 key for admin mode) |

---

<!-- Last Updated: 2026-01-20 - Background strip issue fixed with screen-space shader quad -->
