# 🌀 HistoricHelix

Repository for various **Historic Helix**–related items.

---

## ⚙️ Streamerbot Integration

### One-Time Setup Requirements
*(Tested with Streamerbot v1.0.1)*

1. **Copy the Import Code**  
   👉 [Streamerbot Import Code (V1.0.1)](https://github.com/TonkersTinkers/HistoricHelix/blob/main/StreamerbotImportCode_V101.sb)

2. **Open Streamerbot**  
   Click **Import** at the top of the main window.

3. **Paste the Code**  
   Paste the copied text from Step 1 into the text area.

4. **Import Actions**  
   Click **Import** at the bottom of the import window.

5. **Enable HH Actions & Commands**  
   Make sure all *HistoricHelix* actions and commands are turned on.

6. **Initialize Global Variables**  
   Run the action named **`HH _ Init Once`** once to install global variables:  
   - Right-click the trigger **Test** → select **Test Trigger**.  
   *(This only needs to be done once.)*

7. **Verify Global Variable Paths**  
   Ensure the global variables point to the correct folder for the ChatPlays server inside your Historic Helix Steam installation directory.

   **Default Paths:**

**HHChatPlaysApp: C:\Program Files (x86)\Steam\steamapps\common\Historic Helix\HistoricHelix_Data\StreamingAssets\HistoricHelixChatForwarder\HistoricHelixChatProject.exe

**HHChatPlaysDir: C:\Program Files (x86)\Steam\steamapps\common\Historic Helix\HistoricHelix_Data\StreamingAssets\HistoricHelixChatForwarder

---

## 🧩 After Setup

- Enable the imported actions and commands.  
- Customize them if desired, or use defaults.  
- Start the server from the **ChatPlays menu**.

---

## 💬 List of Commands

There are two control modes:

1. Enemy Control (default, most streams use this)
2. Player Control (optional, only if the streamer turns it on)

Anyone in chat can send commands. Commands are parsed case-insensitive.

Q allows you to bypass the 30 second rate limit for the same command.

All commands can also be queued using `!q` / `q` with optional per-step delay. Example:
`!q up 300 left right 1000 down`
This means: `up`, wait 300 ms, `left`, `right`, wait 1000 ms, `down`.

`say` can be sent inline in a queue too.

---

## Enemy Control Mode (default)

In this mode, chat controls an assigned enemy ("rattler"). The game server (`allowEnemyControlInput`) must be on.

When Enemy Control is enabled:
- movement commands go to the enemy
- `/say` displays a speech bubble and tags your name

### Movement

You can move the enemy using either WASD-style tokens or full words. All of these map to the same actions:

| Chat Input | Result                                                       |
|------------|--------------------------------------------------------------|
| `w`, `up`  | Move enemy up                                                |
| `s`, `down`| Move enemy down                                              |
| `a`, `left`| Move enemy left                                              |
| `d`, `right`| Move enemy right                                            |

These get sent to the server as `up`, `down`, `left`, `right`, and are enqueued to that enemy via `ChatPlaysRattlerController.EnqueueDirection(...)`, which injects the next move for that user's assigned rattler.

Only users currently assigned to an active enemy slot are considered "in control". Assignment is handled automatically when you send a valid command, if auto-assign is enabled.

### Talking

| Chat Input                           | Result                                                                                 |
|-------------------------------------|----------------------------------------------------------------------------------------|
| `/say your message here`            | Show `username: your message here` over the controlled enemy                           |
| `say your message here`             | Same as above                                                                          |
| `!say your message here`            | Same as above                                                                          |

How it works:
- The server records the latest message per user.
- The HUD above the rattler shows `username: message`.
- This updates live as `OnSayUpdated` fires.

If you run `/say` and you're currently the active controller for that enemy slot, everyone will see it in-game.

### Queueing multiple moves

| Chat Input example                           | Result                                                                                             |
|---------------------------------------------|----------------------------------------------------------------------------------------------------|
| `!q up left left down`                      | Queues `up`, `left`, `left`, `down` in order                                                      |
| `!q up 300 left right 1000 down`            | Same, but waits 300 ms before `left`, then default delay before `right`, then 1000 ms before `down` |
| `!q say fear me mortals`                    | Sets your `/say` bubble to `fear me mortals`, also updates HUD                                    |

### EXPERIMENTAL
- The rest of the commands and options are experimental only, really to run a chatplays session,
- the above is the only thing needed, however, the following is listed for experimental use only and not to be construed as working properly.

### Joining / eligibility

| Chat Input | Result                                                                                         |
|------------|------------------------------------------------------------------------------------------------|
| `!join`    | Adds you to the raffle/eligibility pools, if the streamer enabled `!join` in settings          |

Notes:
- Streamer can require `!join` before any of your commands count.
- Streamer can also auto-assign you to a free enemy slot on first valid command, even without `!join`.

Technical:
- Per-user queue is capped (default max 20 commands).
- Delay defaults to server config `InitialCommandDelay` (default 1500 ms).
- Each queued item is dispatched in order and logged.

---

## Player Control Mode (optional / advanced)

This is only active if the streamer explicitly turns on Player Control (`allowPlayerControlInput`). Most streams will not use this.

If Player Control is enabled:
- Only one user is considered the "player".
- That user can directly move the player character, navigate menus, zoom, and pan the camera.
- Other users' commands are ignored unless they are chosen as the player.

If you are the chosen player, you can use:

### Movement / navigation

These control the actual player movement and also drive menu navigation:

| Chat Input          | Player Action                                                                 |
|---------------------|-------------------------------------------------------------------------------|
| `w`, `up`           | Move player up, move menu cursor up                                           |
| `s`, `down`         | Move player down, move menu cursor down                                       |
| `a`, `left`         | Move player left, move menu cursor left                                       |
| `d`, `right`        | Move player right, move menu cursor right                                     |

These map to internal commands: `up`, `down`, `left`, `right`.

### Confirm / cancel

| Chat Input         | Player Action                                   |
|--------------------|-------------------------------------------------|
| `g`, `submit`      | Press confirm / accept / interact               |
| `c`, `cancel`      | Press cancel / escape / back                    |

These map to `submit` and `cancel`.

### Camera zoom

| Chat Input         | Player Action                 |
|--------------------|-------------------------------|
| `zi`, `zoomin`     | Zoom camera in once           |
| `zo`, `zoomout`    | Zoom camera out once          |

### Camera pan / look

These temporarily pan the camera in a direction. Pan is held briefly, then automatically released.

| Chat Input         | Player Action                        |
|--------------------|--------------------------------------|
| `pu`, `panup`      | Pan camera up                        |
| `pd`, `pandown`    | Pan camera down                      |
| `pl`, `panleft`    | Pan camera left                      |
| `pr`, `panright`   | Pan camera right                     |

The server simulates a held pan for ~500 ms, then stops pan and recenters.

### Queueing (player mode)

All of the above can also be queued with `!q`, same as enemy mode:
- Movement
- Zoom
- Pan
- submit / cancel
- say (chat bubble)

Queued player commands will only execute if Player Control is enabled and you are the current player.

---

## Summary

- If you're just here to mess with the enemies (the default mode):  
  Use `w a s d`, `up/down/left/right`, `/say hello`, and optionally `!q ...` to script a short path.

- `!join` may be required depending on streamer settings.

- Player Control mode exists, but it's off in most streams. If it's on and you're the chosen player, you also get access to `submit`, `cancel`, `zoomin`, `zoomout`, `panup`, etc.
