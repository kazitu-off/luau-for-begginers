## 📚 Урок 1: Из чего состоит любой чит-хаб

Любой хаб — это **4 части**:

1. **Библиотека UI** — готовая библиотека, которая рисует окно, кнопки, ползунки. Писать свою с нуля — сотни строк. Обычно используют `Rayfield`, `Fluent`, `Kavo` — это просто код, который скачивается и рисует интерфейс.
2. **Создание окна** — говоришь библиотеке: "сделай окно с таким названием".
3. **Секции и кнопки** — добавляешь вкладки (Speed, ESP, Misc) и элементы (кнопка, слайдер, чекбокс).
4. **Логика** — что делает каждая кнопка. Тут твой код: `Humanoid.WalkSpeed = 70`.

Всё. Хаб = UI + логика. Разберём по шагам.

---

## 📚 Урок 2: Загружаем UI-библиотеку

Библиотека — это чужой скрипт, который создаёт меню. Ты его скачиваешь одной строкой и получаешь объект-функцию для создания окна.

```lua
-- Скачиваем библиотеку Rayfield с GitHub
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
```

**Что тут:**
- `game:HttpGet(url)` — скачивает текст скрипта по ссылке.
- `loadstring(...)` — превращает текст в исполняемую функцию.
- `()` в конце — вызывает эту функцию, она возвращает таблицу с функциями (`Rayfield:CreateWindow` и т.д.).

**Проверка:** после этой строки в переменной `Rayfield` лежит таблица. Можешь написать `print(Rayfield)` — увидишь `table: 0x...`.

---

## 📚 Урок 3: Создаём окно

```lua
local Window = Rayfield:CreateWindow({
    Name = "My First Hub",              -- название окна
    LoadingTitle = "Загрузка...",       -- текст при загрузке
    LoadingSubtitle = "by me",          -- подпись
    ConfigurationSaving = {
        Enabled = false                 -- не сохранять настройки
    },
    KeySystem = false                   -- без ключа
})
```

**Разбор:**
- `CreateWindow` — функция библиотеки, создаёт окно.
- Внутри `{}` — таблица с параметрами. Каждая строка — `Ключ = Значение`.
- `Window` — объект окна, через него будем добавлять вкладки.

**Что произойдёт:** на экране появится пустое окно с названием "My First Hub".

---

## 📚 Урок 4: Добавляем вкладку (Tab)

Вкладки — это разделы меню. Например, "Speed", "ESP", "Misc".

```lua
local SpeedTab = Window:CreateTab("Speed", 4483362458)
```

**Разбор:**
- `Window:CreateTab(название, id_иконки)` — создаёт вкладку.
- `"Speed"` — название, которое увидит пользователь.
- `4483362458` — ID иконки из Roblox (можно найти на сайте rbxassetid). Можно не указывать.

Теперь `SpeedTab` — объект вкладки, в неё будем добавлять кнопки.

---

## 📚 Урок 5: Добавляем секцию и слайдер скорости

Секция — это группа элементов внутри вкладки.

```lua
local SpeedSection = SpeedTab:CreateSection("Настройки скорости")
```

Теперь добавим **слайдер** — ползунок, который меняет скорость.

```lua
local SpeedSlider = SpeedTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 200},       -- минимум и максимум
    Increment = 1,            -- шаг (через сколько двигается)
    Suffix = "studs",        -- подпись после числа
    CurrentValue = 16,       -- начальное значение
    Flag = "SpeedSlider",    -- уникальный ID для сохранения
    Callback = function(value)
        -- ЭТО ЛОГИКА. Выполняется каждый раз, когда двигаешь ползунок.
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = value
        end
    end
})
```

**Разбор `Callback`:**
- `Callback` — функция, которая вызывается при изменении значения.
- `value` — новое значение слайдера (число от 16 до 200).
- `char` — текущий персонаж. Проверяем, что он есть (`if char`).
- `char:FindFirstChild("Humanoid")` — ищем Humanoid. Если нет — ничего не делаем (не крашимся).
- `Humanoid.WalkSpeed = value` — присваиваем скорость.

**Почему через `FindFirstChild`, а не `.Humanoid`:** если персонаж умер и не заспавнился, `.Humanoid` выдаст ошибку. `FindFirstChild` вернёт `nil`, и мы просто пропустим.

---

## 📚 Урок 6: Добавляем кнопку (например, респавн)

```lua
SpeedTab:CreateButton({
    Name = "Respawn",
    Callback = function()
        game.Players.LocalPlayer.Character:BreakJoints()
    end
})
```

**Разбор:**
- `BreakJoints()` — разрывает все соединения в модели персонажа, он разваливается. Это простой способ убить себя.
- `Callback` без аргументов — просто выполняется при клике.

---

## 📚 Урок 7: Добавляем чекбокс (Toggle) для ESP

Чекбокс — это переключатель вкл/выкл. Сделаем ESP через Highlight.

```lua
local ESPEnabled = false  -- храним состояние вне функции

SpeedTab:CreateToggle({
    Name = "ESP (Highlight)",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(state)
        ESPEnabled = state  -- state = true или false
        
        if state then
            -- Включаем ESP для всех игроков
            for _, player in pairs(game.Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer and player.Character then
                    local hl = Instance.new("Highlight")
                    hl.Name = "MyESP"
                    hl.Adornee = player.Character
                    hl.FillColor = Color3.fromRGB(255, 0, 0)
                    hl.FillTransparency = 0.5
                    hl.Parent = player.Character
                end
            end
        else
            -- Выключаем: удаляем все Highlight с именем MyESP
            for _, player in pairs(game.Players:GetPlayers()) do
                if player.Character then
                    local hl = player.Character:FindFirstChild("MyESP")
                    if hl then hl:Destroy() end
                end
            end
        end
    end
})
```

**Разбор:**
- `state` — `true` если включили, `false` если выключили.
- В цикле `for _, player in pairs(...)` проходим по всем игрокам.
- `player ~= LocalPlayer` — пропускаем себя.
- Создаём `Highlight` и вешаем на персонажа.
- При выключении — ищем `Highlight` с именем `MyESP` и удаляем.

**Проблема:** если игрок зайдёт **после** включения ESP — на нём не будет Highlight. Нужно слушать `PlayerAdded`.

---

## 📚 Урок 8: Обработка новых игроков (чтобы ESP работал всегда)

```lua
-- Функция: вешает ESP на игрока (если ESP включён)
local function applyESP(player)
    if not ESPEnabled then return end
    if player == game.Players.LocalPlayer then return end
    if not player.Character then return end
    
    -- Удаляем старый, если есть
    local old = player.Character:FindFirstChild("MyESP")
    if old then old:Destroy() end
    
    local hl = Instance.new("Highlight")
    hl.Name = "MyESP"
    hl.Adornee = player.Character
    hl.FillColor = Color3.fromRGB(255, 0, 0)
    hl.FillTransparency = 0.5
    hl.Parent = player.Character
end

-- Слушаем новых игроков
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)  -- ждём загрузки персонажа
        applyESP(player)
    end)
end)
```

**Разбор:**
- `PlayerAdded` — игрок зашёл на сервер.
- `CharacterAdded` — у игрока появился персонаж (после спавна или респавна).
- `task.wait(0.5)` — ждём полсекунды, чтобы Humanoid успел создаться.
- `applyESP` — вешает Highlight, если ESP включён.

---

## 📚 Урок 9: Собираем всё вместе

Вот полный код твоего первого хаба. Копируй целиком в инжектор:

```lua
-- 1. Загружаем библиотеку
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- 2. Создаём окно
local Window = Rayfield:CreateWindow({
    Name = "My First Hub",
    LoadingTitle = "Загрузка...",
    LoadingSubtitle = "by me",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

-- 3. Создаём вкладку
local MainTab = Window:CreateTab("Main", nil)

-- 4. Слайдер скорости
MainTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 200},
    Increment = 1,
    Suffix = "studs",
    CurrentValue = 16,
    Flag = "SpeedSlider",
    Callback = function(value)
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = value
        end
    end
})

-- 5. Слайдер прыжка
MainTab:CreateSlider({
    Name = "JumpPower",
    Range = {50, 300},
    Increment = 1,
    Suffix = "power",
    CurrentValue = 50,
    Flag = "JumpSlider",
    Callback = function(value)
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.JumpPower = value
        end
    end
})

-- 6. Кнопка респавна
MainTab:CreateButton({
    Name = "Respawn",
    Callback = function()
        game.Players.LocalPlayer.Character:BreakJoints()
    end
})

-- 7. ESP
local ESPEnabled = false

local function applyESP(player)
    if not ESPEnabled then return end
    if player == game.Players.LocalPlayer then return end
    if not player.Character then return end
    local old = player.Character:FindFirstChild("MyESP")
    if old then old:Destroy() end
    local hl = Instance.new("Highlight")
    hl.Name = "MyESP"
    hl.Adornee = player.Character
    hl.FillColor = Color3.fromRGB(255, 0, 0)
    hl.FillTransparency = 0.5
    hl.Parent = player.Character
end

MainTab:CreateToggle({
    Name = "ESP",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(state)
        ESPEnabled = state
        if state then
            for _, p in pairs(game.Players:GetPlayers()) do
                applyESP(p)
            end
        else
            for _, p in pairs(game.Players:GetPlayers()) do
                if p.Character then
                    local hl = p.Character:FindFirstChild("MyESP")
                    if hl then hl:Destroy() end
                end
            end
        end
    end
})

-- Обработка новых игроков
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        applyESP(player)
    end)
end)

-- 8. Уведомление о загрузке
Rayfield:Notify({
    Title = "My First Hub",
    Content = "Загружено успешно!",
    Duration = 3
})
```

---

## 📚 Урок 10: Как это редактировать и расширять

**Хочешь добавить новую кнопку?** Просто копируй блок `CreateButton` и меняй `Name` и `Callback`.

**Хочешь добавить вкладку?** `local NewTab = Window:CreateTab("Название", nil)`. Потом `NewTab:CreateButton(...)`.

**Хочешь свой цвет Highlight?** Меняй `Color3.fromRGB(255, 0, 0)` на любой другой, например `Color3.fromRGB(0, 255, 0)` — зелёный.

**Хочешь текст над головой (Billboard)?** Вместо Highlight создавай `BillboardGui` с `TextLabel` (это я объяснял в прошлом уроке про ESP).

---

## ⚠️ Что важно

1. **`Callback` — это сердце.** Всё, что ты хочешь сделать при клике/ползунке — пиши в `Callback`.
2. **Проверяй `Character` на `nil`.** Иначе крашится при смерти.
3. **Не создавай объекты каждый кадр.** Только при изменении состояния.
4. **Библиотека Rayfield — не единственная.** Есть Fluent, Kavo, WindUI. Все работают похоже: `CreateWindow → CreateTab → CreateButton/Slider/Toggle`.
