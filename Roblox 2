--// Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Флаги функций и переменные
local AimBotEnabled = false
local ESPEnabled = false
local RayTracingEnabled = false
local noclipEnabled = false
local HPEnabled = false
local SpeedHackEnabled = false
local JumpHackEnabled = false  -- переменная для Jump Hack

local aimbotRange = 75
local aimSpeed = 0.05 -- Быстрый поворот

local SpeedValue = 70      -- значение для Speed Hack
local DefaultSpeed = 16    -- стандартная скорость ходьбы

local JumpPowerValue = 100 -- сила Jump Hack (изменяется во вкладке Advanced)
local DefaultJumpPower = 50 -- стандартная сила прыжка

--// Функция поиска ближайшего игрока с проверкой команд
local function getClosestPlayer()
    local character = LocalPlayer.Character
    if not character then return nil end
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return nil end

    local closestPlayer = nil
    local closestDistance = aimbotRange

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= LocalPlayer and (not LocalPlayer.Team or otherPlayer.Team ~= LocalPlayer.Team) then
            local otherCharacter = otherPlayer.Character
            if otherCharacter then
                local otherRootPart = otherCharacter:FindFirstChild("HumanoidRootPart")
                if otherRootPart then
                    local distance = (humanoidRootPart.Position - otherRootPart.Position).Magnitude
                    if distance < closestDistance then
                        closestDistance = distance
                        closestPlayer = otherRootPart
                    end
                end
            end
        end
    end

    return closestPlayer
end

--// Аимбот (от первого лица)
local function aimAtTarget()
    if AimBotEnabled then
        local character = LocalPlayer.Character
        if not character then return end
        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
        if not humanoidRootPart then return end

        local target = getClosestPlayer()
        if target then
            local targetCFrame = CFrame.new(humanoidRootPart.Position, target.Position)
            local tweenInfo = TweenInfo.new(aimSpeed, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
            local tween = TweenService:Create(humanoidRootPart, tweenInfo, {CFrame = targetCFrame})
            tween:Play()
        end
    end
end

--// ESP (выделение противников через стены)
local function updateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hl = player.Character:FindFirstChildOfClass("Highlight")
            if hl then hl:Destroy() end
        end
    end

    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and (not LocalPlayer.Team or player.Team ~= LocalPlayer.Team) and player.Character then
                local hl = Instance.new("Highlight")
                hl.Parent = player.Character
                hl.FillColor = Color3.new(1, 0, 0)
                hl.FillTransparency = 0
                hl.OutlineTransparency = 1
            end
        end
    end
end

--// Рей‑трейсинг (линии от LocalPlayer до других игроков)
local function updateRayTracing()
    for _, v in pairs(workspace:GetChildren()) do
        if v:IsA("Beam") then
            v:Destroy()
        end
    end

    if RayTracingEnabled then
        local closestPlayer = getClosestPlayer()
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and (not LocalPlayer.Team or player.Team ~= LocalPlayer.Team) and player.Character then
                local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
                if rootPart and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local beam = Instance.new("Beam", workspace)
                    local attachment0 = Instance.new("Attachment", LocalPlayer.Character.HumanoidRootPart)
                    local attachment1 = Instance.new("Attachment", rootPart)
                    beam.Attachment0 = attachment0
                    beam.Attachment1 = attachment1

                    if rootPart == closestPlayer then
                        beam.Color = ColorSequence.new(Color3.new(0, 1, 0))
                    else
                        beam.Color = ColorSequence.new(Color3.new(1, 0, 0))
                    end

                    beam.Width0 = 0.1
                    beam.Width1 = 0.1
                    beam.FaceCamera = true
                    game:GetService("Debris"):AddItem(beam, 0.5)
                end
            end
        end
    end
end

--// Ноклип – отключение столкновений у частей персонажа
RunService.Stepped:Connect(function()
    if noclipEnabled and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

--// Функция отображения HP над головой (в формате "100/100")
local function updateHPBillboards()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                local currentHealth = humanoid.Health
                local maxHealth = humanoid.MaxHealth
                local healthRatio = currentHealth / maxHealth
                if healthRatio < 0 then healthRatio = 0 end
                if healthRatio > 1 then healthRatio = 1 end

                local billboard = head:FindFirstChild("HPBillboard")
                if not billboard then
                    billboard = Instance.new("BillboardGui")
                    billboard.Name = "HPBillboard"
                    billboard.Adornee = head
                    billboard.Size = UDim2.new(0, 50, 0, 20)
                    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                    billboard.AlwaysOnTop = true

                    local textLabel = Instance.new("TextLabel")
                    textLabel.Name = "HPLabel"
                    textLabel.Size = UDim2.new(1, 0, 1, 0)
                    textLabel.BackgroundTransparency = 1
                    textLabel.TextScaled = true
                    textLabel.Font = Enum.Font.SourceSansBold
                    textLabel.Parent = billboard
                    billboard.Parent = head
                end

                local textLabel = billboard:FindFirstChild("HPLabel")
                if textLabel then
                    textLabel.Text = math.floor(currentHealth) .. "/" .. math.floor(maxHealth)
                    textLabel.TextColor3 = Color3.fromHSV(healthRatio * 0.33, 1, 1)
                end
            end
        end
    end
end

-- Если HP выключен – удаляем существующие HPBillboard
local function clearHPBillboards()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local billboard = head:FindFirstChild("HPBillboard")
            if billboard then
                billboard:Destroy()
            end
        end
    end
end

--// Обновление скорости ходьбы (Speed Hack)
RunService.RenderStepped:Connect(function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if SpeedHackEnabled then
            humanoid.WalkSpeed = SpeedValue
        else
            humanoid.WalkSpeed = DefaultSpeed
        end
    end
end)

--// Обновление силы прыжка (Jump Hack)
RunService.RenderStepped:Connect(function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if JumpHackEnabled then
            humanoid.JumpPower = JumpPowerValue
        else
            humanoid.JumpPower = DefaultJumpPower
        end
    end
end)

--// Основной цикл обновления функций
RunService.RenderStepped:Connect(function()
    if AimBotEnabled then
        aimAtTarget()
    end
    if ESPEnabled then
        updateESP()
    end
    if RayTracingEnabled then
        updateRayTracing()
    end
    if HPEnabled then
        updateHPBillboards()
    else
        clearHPBillboards()
    end
end)

---------------------------------------------------------------------------------
--// Создание GUI меню с двумя вкладками (вкладки расположены вертикально слева)
---------------------------------------------------------------------------------
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:FindFirstChildOfClass("PlayerGui"))
ScreenGui.Name = "CheatMenu"
ScreenGui.ResetOnSpawn = false  -- меню не сбрасывается при смерти

-- Главное окно меню (перетаскиваемое)
local MenuFrame = Instance.new("Frame", ScreenGui)
MenuFrame.Name = "MenuFrame"
MenuFrame.Size = UDim2.new(0, 260, 0, 300)  -- начальный размер развернутого меню
MenuFrame.Position = UDim2.new(0.5, -130, 0.5, -150)
MenuFrame.BackgroundColor3 = Color3.new(0, 0, 0)
MenuFrame.BackgroundTransparency = 0.3  -- делаем меню чуть прозрачным
MenuFrame.Active = true
MenuFrame.Draggable = true

-- Скругляем углы меню
local menuCorner = Instance.new("UICorner", MenuFrame)
menuCorner.CornerRadius = UDim.new(0, 10)

-- Объявляем переменную resizer (форвард-декларация)
local resizer

-- Вертикальная панель вкладок (слева)
local TabBar = Instance.new("Frame", MenuFrame)
TabBar.Name = "TabBar"
TabBar.Size = UDim2.new(0, 50, 1, 0)
TabBar.Position = UDim2.new(0, 0, 0, 0)
TabBar.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)

local TabBarLayout = Instance.new("UIListLayout", TabBar)
TabBarLayout.SortOrder = Enum.SortOrder.LayoutOrder
TabBarLayout.Padding = UDim.new(0, 5)

local MainTabButton = Instance.new("TextButton", TabBar)
MainTabButton.Name = "MainTabButton"
MainTabButton.Size = UDim2.new(1, 0, 0, 40)
MainTabButton.Text = "Main"
MainTabButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
MainTabButton.TextColor3 = Color3.new(1, 1, 1)

local AdvancedTabButton = Instance.new("TextButton", TabBar)
AdvancedTabButton.Name = "AdvancedTabButton"
AdvancedTabButton.Size = UDim2.new(1, 0, 0, 40)
AdvancedTabButton.Text = "Advanced"
AdvancedTabButton.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
AdvancedTabButton.TextColor3 = Color3.new(1, 1, 1)

-- Область для содержимого (справа от вкладок)
local ContentArea = Instance.new("Frame", MenuFrame)
ContentArea.Name = "ContentArea"
ContentArea.Size = UDim2.new(1, -50, 1, 0)
ContentArea.Position = UDim2.new(0, 50, 0, 0)
ContentArea.BackgroundTransparency = 1

-- Скроллируемая область для вкладки Main
local MainContent = Instance.new("ScrollingFrame", ContentArea)
MainContent.Name = "MainContent"
MainContent.Size = UDim2.new(1, 0, 1, 0)
MainContent.Position = UDim2.new(0, 0, 0, 0)
MainContent.CanvasSize = UDim2.new(0, 0, 0, 300)
MainContent.BackgroundTransparency = 1
MainContent.ScrollBarThickness = 6

local MainUIListLayout = Instance.new("UIListLayout", MainContent)
MainUIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
MainUIListLayout.Padding = UDim.new(0, 5)

-- Скроллируемая область для вкладки Advanced
local AdvancedContent = Instance.new("ScrollingFrame", ContentArea)
AdvancedContent.Name = "AdvancedContent"
AdvancedContent.Size = UDim2.new(1, 0, 1, 0)
AdvancedContent.Position = UDim2.new(0, 0, 0, 0)
AdvancedContent.CanvasSize = UDim2.new(0, 0, 0, 220)
AdvancedContent.BackgroundTransparency = 1
AdvancedContent.ScrollBarThickness = 6
AdvancedContent.Visible = false

local AdvancedUIListLayout = Instance.new("UIListLayout", AdvancedContent)
AdvancedUIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
AdvancedUIListLayout.Padding = UDim.new(0, 5)

-- Функция для создания toggle-кнопок (для вкладки Main)
local orderCounter = 1
local function createButton(name, toggleVar, parentFrame)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -10, 0, 30)
    button.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = name .. ": off"
    button.LayoutOrder = orderCounter
    orderCounter = orderCounter + 1
    button.Parent = parentFrame

    button.MouseButton1Click:Connect(function()
        if toggleVar == "AIM" then
            AimBotEnabled = not AimBotEnabled
            button.Text = "Aimbot: " .. (AimBotEnabled and "on" or "off")
        elseif toggleVar == "ESP" then
            ESPEnabled = not ESPEnabled
            button.Text = "ESP: " .. (ESPEnabled and "on" or "off")
            updateESP()
        elseif toggleVar == "RAYTRACE" then
            RayTracingEnabled = not RayTracingEnabled
            button.Text = "Raytrace: " .. (RayTracingEnabled and "on" or "off")
            updateRayTracing()
        elseif toggleVar == "NOCLIP" then
            noclipEnabled = not noclipEnabled
            button.Text = "Noclip: " .. (noclipEnabled and "on" or "off")
        elseif toggleVar == "HP" then
            HPEnabled = not HPEnabled
            button.Text = "HP: " .. (HPEnabled and "on" or "off")
        elseif toggleVar == "SPEED" then
            SpeedHackEnabled = not SpeedHackEnabled
            button.Text = "Speed Hack: " .. (SpeedHackEnabled and "on" or "off")
        elseif toggleVar == "JUMP" then
            JumpHackEnabled = not JumpHackEnabled
            button.Text = "Jump Hack: " .. (JumpHackEnabled and "on" or "off")
        end
    end)
    return button
end

-- Создаем кнопки для вкладки Main
orderCounter = 1
local aimButton = createButton("Aimbot", "AIM", MainContent)
local espButton = createButton("ESP", "ESP", MainContent)
local rayButton = createButton("Raytrace", "RAYTRACE", MainContent)
local noclipButton = createButton("Noclip", "NOCLIP", MainContent)
local hpButton = createButton("HP", "HP", MainContent)
local speedButton = createButton("Speed Hack", "SPEED", MainContent)
local jumpButton = createButton("Jump Hack", "JUMP", MainContent)

-- На вкладке Advanced – функция изменения aimbot range
local aimbotRangeButton = Instance.new("TextButton")
aimbotRangeButton.Size = UDim2.new(1, -10, 0, 30)
aimbotRangeButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
aimbotRangeButton.TextColor3 = Color3.new(1, 1, 1)
aimbotRangeButton.Text = "aimbot range (" .. aimbotRange .. ")"
aimbotRangeButton.LayoutOrder = 1
aimbotRangeButton.Parent = AdvancedContent

local rangeInput = Instance.new("TextBox")
rangeInput.Size = UDim2.new(1, -10, 0, 30)
rangeInput.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
rangeInput.TextColor3 = Color3.new(1, 1, 1)
rangeInput.PlaceholderText = "Enter new range"
rangeInput.LayoutOrder = 2
rangeInput.Parent = AdvancedContent
rangeInput.Visible = false

aimbotRangeButton.MouseButton1Click:Connect(function()
    rangeInput.Visible = not rangeInput.Visible
end)

rangeInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newRange = tonumber(rangeInput.Text)
        if newRange then
            aimbotRange = newRange
            aimbotRangeButton.Text = "aimbot range (" .. aimbotRange .. ")"
        end
        rangeInput.Text = ""
        rangeInput.Visible = false
    end
end)

-- На вкладке Advanced – кнопка изменения скорости (Speed Change)
local speedChangeButton = Instance.new("TextButton")
speedChangeButton.Size = UDim2.new(1, -10, 0, 30)
speedChangeButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
speedChangeButton.TextColor3 = Color3.new(1, 1, 1)
speedChangeButton.Text = "Speed Change (" .. SpeedValue .. ")"
speedChangeButton.LayoutOrder = 3
speedChangeButton.Parent = AdvancedContent

local speedInput = Instance.new("TextBox")
speedInput.Size = UDim2.new(1, -10, 0, 30)
speedInput.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
speedInput.TextColor3 = Color3.new(1, 1, 1)
speedInput.PlaceholderText = "Enter new speed"
speedInput.LayoutOrder = 4
speedInput.Parent = AdvancedContent
speedInput.Visible = false

speedChangeButton.MouseButton1Click:Connect(function()
    speedInput.Visible = not speedInput.Visible
end)

speedInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newSpeed = tonumber(speedInput.Text)
        if newSpeed then
            SpeedValue = newSpeed
            speedChangeButton.Text = "Speed Change (" .. SpeedValue .. ")"
        end
        speedInput.Text = ""
        speedInput.Visible = false
    end
end)

-- На вкладке Advanced – кнопка изменения силы Jump Hack
local jumpPowerChangeButton = Instance.new("TextButton")
jumpPowerChangeButton.Size = UDim2.new(1, -10, 0, 30)
jumpPowerChangeButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
jumpPowerChangeButton.TextColor3 = Color3.new(1, 1, 1)
jumpPowerChangeButton.Text = "Jump Hack Power (" .. JumpPowerValue .. ")"
jumpPowerChangeButton.LayoutOrder = 5
jumpPowerChangeButton.Parent = AdvancedContent

local jumpPowerInput = Instance.new("TextBox")
jumpPowerInput.Size = UDim2.new(1, -10, 0, 30)
jumpPowerInput.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
jumpPowerInput.TextColor3 = Color3.new(1, 1, 1)
jumpPowerInput.PlaceholderText = "Enter new jump power"
jumpPowerInput.LayoutOrder = 6
jumpPowerInput.Parent = AdvancedContent
jumpPowerInput.Visible = false

jumpPowerChangeButton.MouseButton1Click:Connect(function()
    jumpPowerInput.Visible = not jumpPowerInput.Visible
end)

jumpPowerInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newJumpPower = tonumber(jumpPowerInput.Text)
        if newJumpPower then
            JumpPowerValue = newJumpPower
            jumpPowerChangeButton.Text = "Jump Hack Power (" .. JumpPowerValue .. ")"
        end
        jumpPowerInput.Text = ""
        jumpPowerInput.Visible = false
    end
end)

-- Переключение вкладок (при нажатии кнопок в TabBar)
MainTabButton.MouseButton1Click:Connect(function()
    MainContent.Visible = true
    AdvancedContent.Visible = false
    MainTabButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    AdvancedTabButton.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
end)

AdvancedTabButton.MouseButton1Click:Connect(function()
    MainContent.Visible = false
    AdvancedContent.Visible = true
    MainTabButton.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    AdvancedTabButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
end)

---------------------------------------------------------------------------------
-- Кнопка сворачивания/разворачивания меню (верхний правый угол)
---------------------------------------------------------------------------------
local defaultExpandedSize = UDim2.new(0, 260, 0, 300)  -- исходный размер развернутого меню
local collapsedSize = UDim2.new(0, 50, 0, 20)           -- размер свёрнутого меню (равен размеру кнопки)

local collapseButton = Instance.new("TextButton", MenuFrame)
collapseButton.Size = UDim2.new(0, 50, 0, 20)
-- В развернутом состоянии кнопка располагается в правом верхнем углу с небольшим смещением вверх
collapseButton.AnchorPoint = Vector2.new(1, 0)
collapseButton.Position = UDim2.new(1, -5, 0, -10)
collapseButton.Text = "Collapse"
collapseButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
collapseButton.TextColor3 = Color3.new(1, 1, 1)
collapseButton.Active = true

local menuExpanded = true
local expandedSize = defaultExpandedSize  -- текущий размер развернутого меню

collapseButton.MouseButton1Click:Connect(function()
    if menuExpanded then
        expandedSize = MenuFrame.Size
        MenuFrame:TweenSize(collapsedSize, "Out", "Quad", 0.3, true)
        collapseButton.Text = "Expand"
        TabBar.Visible = false
        ContentArea.Visible = false
        if resizer then resizer.Visible = false end
        collapseButton.AnchorPoint = Vector2.new(0, 0)
        collapseButton.Position = UDim2.new(0, 0, 0, 0)
        menuExpanded = false
    else
        MenuFrame:TweenSize(expandedSize, "Out", "Quad", 0.3, true)
        collapseButton.Text = "Collapse"
        TabBar.Visible = true
        ContentArea.Visible = true
        if resizer then resizer.Visible = true end
        collapseButton.AnchorPoint = Vector2.new(1, 0)
        collapseButton.Position = UDim2.new(1, -5, 0, -10)
        menuExpanded = true
    end
end)

---------------------------------------------------------------------------------
-- Ресайзер: изменение размера меню через перетаскивание нижнего правого угла
---------------------------------------------------------------------------------
resizer = Instance.new("Frame", MenuFrame)
resizer.Name = "Resizer"
if UserInputService.TouchEnabled then
    resizer.Size = UDim2.new(0, 40, 0, 40)
else
    resizer.Size = UDim2.new(0, 20, 0, 20)
end
resizer.AnchorPoint = Vector2.new(1, 1)
resizer.Position = UDim2.new(1, 0, 1, 0)
resizer.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
resizer.BorderSizePixel = 0
resizer.Active = true
resizer.Selectable = true

local resizing = false
local startSize
local startPos

resizer.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        resizing = true
        startSize = MenuFrame.AbsoluteSize
        startPos = input.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                resizing = false
                if menuExpanded then
                    expandedSize = MenuFrame.Size
                end
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input, gameProcessed)
    if resizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - startPos
        local newWidth = math.max(200, startSize.X + delta.X)
        local newHeight = math.max(150, startSize.Y + delta.Y)
        MenuFrame.Size = UDim2.new(0, newWidth, 0, newHeight)
    end
end)
