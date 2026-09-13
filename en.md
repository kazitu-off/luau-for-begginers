## 📚 Lesson 1: What are LocalPlayer, Character, and Humanoid?

Before changing the speed, we need to understand what we’re working with. In Roblox, everything is built on objects.

**Hierarchy:**
```
game
└── Players
    └── LocalPlayer (that’s you)
        └── Character (your avatar in the world)
            └── Humanoid (parameters: speed, jump, health)
            └── HumanoidRootPart (physical body)
```

**Code:**
```lua
-- game.Players — this is the service where all players are stored
-- LocalPlayer — this is specifically your player (not someone else’s!)
local LocalPlayer = game.Players.LocalPlayer

-- Character — the model of your character. Can be nil if you haven’t spawned yet
local Character = LocalPlayer.Character

-- Humanoid is the character’s “brain”. Stores WalkSpeed, JumpPower, Health
-- WaitForChild waits for the object to appear (in case the game is loading)
local Humanoid = Character:WaitForChild("Humanoid")
```

**Why use `WaitForChild` instead of just `.Humanoid`?**
Because at the time the script is executed, the object may not have had time to load yet. `WaitForChild` will wait for it to appear and won’t throw an error like `attempt to index nil`.

**Check:**
```lua
print(Humanoid.WalkSpeed) -- will show the current speed (usually 16)
```

---

## 📚 Lesson 2: Changing WalkSpeed — the basics

`WalkSpeed` is a number. The standard in Roblox is **16**. Change the number — the speed changes.

```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

Humanoid.WalkSpeed = 50
```

**What’s happening here:**
1. We get the Humanoid
2. We assign the WalkSpeed value to 50

**Writing options:**
```lua
Humanoid.WalkSpeed = 50          -- replace with 50
Humanoid.WalkSpeed = 16 + 34     -- also 50, but more visually clear
Humanoid.WalkSpeed += 10         -- add 10 to the current value
Humanoid.WalkSpeed = 100         -- faster, but riskier
```

**Roblox limitations:**
- Minimum: `0` (the character doesn’t move)
- Maximum: technically unlimited, but the server may kill/kick the character at higher values
- Safe range: **16–70**
- Dangerous: **100+** (anti‑cheats detect it)

**Why might the server “kick” you?**
The character’s speed is synchronized with the server. If you move faster than the server expects, it sends you back. In games with anti‑cheats, it kicks you.

---

## 📚 Lesson 3: JumpPower and JumpHeight are two different parameters

There’s a nuance here. In Roblox, there are two jump systems, and they’re mutually exclusive.

**Checking which system is used:**
```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
print(Humanoid.UseJumpPower) -- true or false
```

**If `UseJumpPower = true`:**
```lua
Humanoid.JumpPower = 150 -- standard 50
```

**If `UseJumpPower = false`:**
```lua
Humanoid.JumpHeight = 20 -- standard 7.2

**How to switch the system:**
```lua
Humanoid.UseJumpPower = true   -- now JumpPower works
Humanoid.JumpPower = 150
```

**What to set:**
- Standard: `JumpPower = 50`, `JumpHeight = 7.2`
- High jump: `JumpPower = 100–150` or `JumpHeight = 15–25`
- Space: `JumpPower = 300+` (may kick)

---

## 📚 Lesson 4: Respawn Handling (CharacterAdded)

Problem: when you die, the `Character` is destroyed and a new one is created. All your changes are reset.

**Solution — listen to the `CharacterAdded` event:**

```lua
local LocalPlayer = game.Players.LocalPlayer

-- Function that applies settings to the character
local function applySettings(character)
    -- Wait for Humanoid inside the new character
    local humanoid = character:WaitForChild("Humanoid")
    
    -- Change the parameters
    humanoid.WalkSpeed = 50
    humanoid.JumpPower = 120
    
    print("Settings applied to the new character")
end

-- Apply to the current character (if it already exists)
if LocalPlayer.Character then
    applySettings(LocalPlayer.Character)
end

-- Listen for the appearance of a new character (after death)
LocalPlayer.CharacterAdded:Connect(applySettings)
```

**How it works line by line:**
1. `applySettings` — a function that takes a character and changes its Humanoid
2. `if LocalPlayer.Character then` — checks that the character already exists
3. `CharacterAdded:Connect(applySettings)` — Roblox will call `applySettings` itself when a new character appears

**Bonus — death tracking:**
```lua
local humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
humanoid.Died:Connect(function()
    print("I died, waiting for respawn...")
end)
---

## 📚 Lesson 5: Smooth Speed Change (Bypassing Anti-Cheat)

A sudden change (16 → 200) is a red flag for the server. Smooth — looks like natural acceleration.

```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

local targetSpeed = 70    -- where we want to get to
local step = 1            -- how much we increase by each time
local delayTime = 0.1     -- pause between steps (seconds)

-- Loop: while the current speed is less than the target speed
while Humanoid.WalkSpeed < targetSpeed do
    Humanoid.WalkSpeed = Humanoid.WalkSpeed + step
    task.wait(delayTime) -- wait 0.1 seconds
end

print("Done! Speed: " .. Humanoid.WalkSpeed)
```

**Analysis:**
- `while ... do ... end` — a loop that runs as long as the condition is true
- `Humanoid.WalkSpeed + step` — we add 1 at a time
- `task.wait(0.1)` — a pause. `task.wait` is more precise than the outdated `wait`
- `..` — string concatenation (joining text with a number)

**How long to wait for different values:**
| Start | Target | Step | Delay | Total time |
|-------|------|-----|----------|---------------|
| 16 | 70 | 1 | 0.1 | ~5.4 sec |
| 16 | 100 | 2 | 0.05 | ~2.1 sec |
| 16 | 50 | 5 | 0.05 | ~0.34 sec |

**Even safer — via TweenService:**
```lua
local TweenService = game:GetService("TweenService")
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

local tween = TweenService:Create(
    Humanoid,
    TweenInfo.new(2, Enum.EasingStyle.Linear), -- 2 seconds, linear
    {WalkSpeed = 70}
)
tween:Play()
```

`TweenService` smoothly interpolates the value from the current to the target over the specified time. But it doesn’t work with all properties — `WalkSpeed` usually works fine.

---

## 🎛️ Bonus: GUI for speed control

To avoid restarting the script every time, let’s create a simple menu.

```lua
-- Create ScreenGui (a container for the UI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Create a button
local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0, 150, 0, 50)         -- 150x50 pixels
Button.Position = UDim2.new(0, 20, 0, 20)       -- a margin of 20px from the top left corner
Button.Text = "Speed: 16"
Button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.Parent = ScreenGui

-- Logic: click — switch between 16 and 70
local isFast = false
Button.MouseButton1Click:Connect(function()
    local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
    if isFast then
        Humanoid.WalkSpeed = 16
        Button.Text = "Speed: 16"
    else
        Humanoid.WalkSpeed = 70
        Button.Text = "Speed: 70"
    end
    isFast = not isFast
end)
```

**What’s here:**
- `ScreenGui` — the interface layer on top of the game
- `TextButton` — a clickable button
- `UDim2.new(0, 150, 0, 50)` — width 150px, height 50px (0 — multiplier from screen size, 150 — pixels)
- `MouseButton1Click` — left‑click event
- `isFast` — flag, stores the current state

---

## ⚠️ What is important to remember

**1. Error `attempt to index nil with 'Humanoid'`**
This means that `Character` is not loaded yet. Solutions:
```lua
-- Option A: wait
local Character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()

-- Option B: use WaitForChild
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid", 5) -- waits up to 5 seconds
```

**2. Speed is reset**
The game may periodically reset WalkSpeed via a server script. The solution is a loop:
```lua
task.spawn(function()
    while task.wait(0.5) do
        local h = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
        if h then h.WalkSpeed = 70 end
    end
end)
```

**3. `wait()` vs `task.wait()`**
`wait()` — outdated, may cause delays greater than specified. `task.wait()` — more accurate, recommended.
