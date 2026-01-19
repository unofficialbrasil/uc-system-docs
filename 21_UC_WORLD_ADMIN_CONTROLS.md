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

### 2.1 How Admin Status is Verified

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

### 2.2 Admin Roles

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
| **F1** | Toggle Admin Mode | Switches between normal and admin camera |

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
║  F1          Toggle Admin Mode                             ║
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

**Symptom:** F1 does nothing

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

## 12. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-19 | Initial documentation |

---

<!-- Last Updated: 2026-01-19 - Initial creation -->
