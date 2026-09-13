
## 📚 Урок 1: Что такое LocalPlayer, Character и Humanoid

Прежде чем менять скорость, нужно понять, с чем мы работаем. В Roblox всё построено на объектах.

**Иерархия:**
```
game
└── Players
    └── LocalPlayer (это ты)
        └── Character (твоя тушка в мире)
            └── Humanoid (параметры: скорость, прыжок, здоровье)
            └── HumanoidRootPart (физическое тело)
```

**Код:**
```lua
-- game.Players — это сервис, где лежат все игроки
-- LocalPlayer — конкретно твой игрок (не чужой!)
local LocalPlayer = game.Players.LocalPlayer

-- Character — модель твоего персонажа. Может быть nil, если ты ещё не заспавнился
local Character = LocalPlayer.Character

-- Humanoid — "мозг" персонажа. Хранит WalkSpeed, JumpPower, Health
-- WaitForChild ждёт, пока объект появится (на случай, если игра грузится)
local Humanoid = Character:WaitForChild("Humanoid")
```

**Почему через `WaitForChild`, а не просто `.Humanoid`?**
Потому что в момент выполнения скрипта объект может ещё не успеть загрузиться. `WaitForChild` подождёт его появления и не выдаст ошибку `attempt to index nil`.

**Проверка:**
```lua
print(Humanoid.WalkSpeed) -- покажет текущую скорость (обычно 16)
```

---

## 📚 Урок 2: Меняем WalkSpeed — база

`WalkSpeed` — это число. Стандарт в Roblox — **16**. Меняешь число — меняется скорость.

```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

Humanoid.WalkSpeed = 50
```

**Что тут происходит:**
1. Достаём Humanoid
2. Присваиваем WalkSpeed значение 50

**Варианты записи:**
```lua
Humanoid.WalkSpeed = 50          -- заменить на 50
Humanoid.WalkSpeed = 16 + 34     -- тоже 50, но наглядно
Humanoid.WalkSpeed += 10         -- добавить 10 к текущей
Humanoid.WalkSpeed = 100         -- быстрее, но рискованнее
```

**Ограничения Roblox:**
- Минимум: `0` (персонаж не двигается)
- Максимум: технически не ограничен, но сервер может убить/откинуть при больших значениях
- Безопасный диапазон: **16–70**
- Опасно: **100+** (античиты ловят)

**Почему сервер может "откинуть"?**
Скорость персонажа синхронизируется с сервером. Если ты движешься быстрее, чем сервер ожидает, он возвращает тебя назад. В играх с античитом — кикает.

---

## 📚 Урок 3: JumpPower и JumpHeight — два разных параметра

Тут есть нюанс. В Roblox две системы прыжка, и они взаимоисключающие.

**Проверка, какая система используется:**
```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
print(Humanoid.UseJumpPower) -- true или false
```

**Если `UseJumpPower = true`:**
```lua
Humanoid.JumpPower = 150 -- стандарт 50
```

**Если `UseJumpPower = false`:**
```lua
Humanoid.JumpHeight = 20 -- стандарт 7.2
```

**Как переключить систему:**
```lua
Humanoid.UseJumpPower = true   -- теперь работает JumpPower
Humanoid.JumpPower = 150
```

**Что ставить:**
- Стандарт: `JumpPower = 50`, `JumpHeight = 7.2`
- Высокий прыжок: `JumpPower = 100–150` или `JumpHeight = 15–25`
- Космос: `JumpPower = 300+` (может кикнуть)

---

## 📚 Урок 4: Обработка респавна (CharacterAdded)

Проблема: когда ты умираешь, `Character` уничтожается и создаётся новый. Все твои изменения сбрасываются.

**Решение — слушать событие `CharacterAdded`:**

```lua
local LocalPlayer = game.Players.LocalPlayer

-- Функция, которая применяет настройки к персонажу
local function applySettings(character)
    -- Ждём Humanoid внутри нового персонажа
    local humanoid = character:WaitForChild("Humanoid")
    
    -- Меняем параметры
    humanoid.WalkSpeed = 50
    humanoid.JumpPower = 120
    
    print("Настройки применены к новому персонажу")
end

-- Применяем к текущему персонажу (если он уже есть)
if LocalPlayer.Character then
    applySettings(LocalPlayer.Character)
end

-- Слушаем появление нового персонажа (после смерти)
LocalPlayer.CharacterAdded:Connect(applySettings)
```

**Как это работает построчно:**
1. `applySettings` — функция, которая берёт персонаж и меняет его Humanoid
2. `if LocalPlayer.Character then` — проверка, что персонаж уже существует
3. `CharacterAdded:Connect(applySettings)` — Roblox сам вызовет `applySettings`, когда появится новый персонаж

**Бонус — отслеживание смерти:**
```lua
local humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")
humanoid.Died:Connect(function()
    print("Я умер, жду респавна...")
end)
```

---

## 📚 Урок 5: Плавное изменение скорости (обход античита)

Резкое изменение (16 → 200) — красный флаг для сервера. Плавное — выглядит как естественное ускорение.

```lua
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

local targetSpeed = 70    -- куда хотим прийти
local step = 1            -- на сколько увеличиваем за раз
local delayTime = 0.1     -- пауза между шагами (секунды)

-- Цикл: пока текущая скорость меньше целевой
while Humanoid.WalkSpeed < targetSpeed do
    Humanoid.WalkSpeed = Humanoid.WalkSpeed + step
    task.wait(delayTime) -- ждём 0.1 сек
end

print("Готово! Скорость: " .. Humanoid.WalkSpeed)
```

**Разбор:**
- `while ... do ... end` — цикл, выполняется пока условие true
- `Humanoid.WalkSpeed + step` — прибавляем по 1
- `task.wait(0.1)` — пауза. `task.wait` точнее, чем устаревший `wait`
- `..` — конкатенация строк (склейка текста с числом)

**Сколько ждать при разных значениях:**
| Старт | Цель | Шаг | Задержка | Итого времени |
|-------|------|-----|----------|---------------|
| 16 | 70 | 1 | 0.1 | ~5.4 сек |
| 16 | 100 | 2 | 0.05 | ~2.1 сек |
| 16 | 50 | 5 | 0.05 | ~0.34 сек |

**Ещё безопаснее — через TweenService:**
```lua
local TweenService = game:GetService("TweenService")
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid")

local tween = TweenService:Create(
    Humanoid,
    TweenInfo.new(2, Enum.EasingStyle.Linear), -- 2 секунды, линейно
    {WalkSpeed = 70}
)
tween:Play()
```

`TweenService` плавно интерполирует значение от текущего к целевому за указанное время. Но работает не со всеми свойствами — `WalkSpeed` обычно ок.

---

## 🎛️ Бонус: GUI для управления скоростью

Чтобы не перезапускать скрипт каждый раз, сделаем простое меню.

```lua
-- Создаём ScreenGui (контейнер для UI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Создаём кнопку
local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0, 150, 0, 50)         -- 150x50 пикселей
Button.Position = UDim2.new(0, 20, 0, 20)       -- отступ 20px от левого верхнего угла
Button.Text = "Speed: 16"
Button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.Parent = ScreenGui

-- Логика: клик — переключение между 16 и 70
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

**Что тут:**
- `ScreenGui` — слой интерфейса поверх игры
- `TextButton` — кликабельная кнопка
- `UDim2.new(0, 150, 0, 50)` — ширина 150px, высота 50px (0 — множитель от размера экрана, 150 — пиксели)
- `MouseButton1Click` — событие клика левой кнопкой
- `isFast` — флаг, хранит текущее состояние

---

## ⚠️ Что важно помнить

**1. Ошибка `attempt to index nil with 'Humanoid'`**
Значит `Character` ещё не загружен. Решения:
```lua
-- Вариант А: ждать
local Character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()

-- Вариант Б: использовать WaitForChild
local Humanoid = game.Players.LocalPlayer.Character:WaitForChild("Humanoid", 5) -- ждёт до 5 сек
```

**2. Скорость сбрасывается**
Игра может периодически сбрасывать WalkSpeed через серверный скрипт. Решение — цикл:
```lua
task.spawn(function()
    while task.wait(0.5) do
        local h = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
        if h then h.WalkSpeed = 70 end
    end
end)
```

**3. `wait()` vs `task.wait()`**
`wait()` — устаревший, может давать задержки больше указанных. `task.wait()` — точнее, рекомендуется.
