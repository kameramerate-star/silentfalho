-- =============================================================================
-- KING HUB - SILENT ENGINE & ESP SYSTEM INTEGRADO
-- =============================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- =============================================================================
-- CONFIGURAÇÕES DO ESP
-- =============================================================================
local espVisible = true
local tracked = {}

local TOGGLE_KEY_ESP = Enum.KeyCode.U
local SHOW_THROUGH_WALLS = true
local BILLBOARD_OFFSET = Vector3.new(0, 2.7, 0)
local FONT = Enum.Font.SourceSansBold
local NAME_TEXT_SIZE = 15
local BAR_WIDTH = 100
local DEBOUNCE_KEY = 0.18
local debounceEsp = false

-- =============================================================================
-- CONFIGURAÇÕES DO SILENT AIM / ENGINE
-- =============================================================================
local TELEGUIADO_ENABLED = true
local MAX_DISTANCE = 10000
local FOV_RADIUS = 237
local FOV_VISIBLE = true
local HIT_PART = "Head"          -- Head | Torso | HumanoidRootPart
local PRIORITY_CROSSHAIR = true  -- true = centro da tela | false = distância 3D
local TEAM_CHECK = true
local VISIBLE_CHECK = true
local HEALTH_CHECK = true
local FOV_ENABLED = true
local FOV_GLOW = false
local FOV_FILLED = false

-- =============================================================================
-- CORES DO TEMA (Estilo Dark & Red)
-- =============================================================================
local RED        = Color3.fromRGB(230, 53, 53)
local DARK_BG    = Color3.fromRGB(13, 13, 13)
local PANEL_BG   = Color3.fromRGB(20, 20, 20)
local TOGGLE_OFF = Color3.fromRGB(50, 50, 50)
local TEXT_WHITE = Color3.fromRGB(225, 225, 225)
local GRAY_TEXT  = Color3.fromRGB(120, 120, 120)
local BORDER_COL = Color3.fromRGB(38, 38, 38)

-- =============================================================================
-- FUNÇÕES DO ESP SYSTEM
-- =============================================================================
local function toggleEspVisibility()
    espVisible = not espVisible
    for _, t in pairs(tracked) do
        if t and t.info and t.info.billboard then
            t.info.billboard.Enabled = espVisible
        end
    end
end

local function createBillboardForCharacter(character, player)
    if player == LocalPlayer then return nil end
    if not character or not character.Parent then return nil end
    
    local head = character:FindFirstChild("Head") or character:FindFirstChild("UpperTorso") or character.PrimaryPart
    if not head then return nil end

    local existing = head:FindFirstChild("ESP_Billboard")
    if existing then existing:Destroy() end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Billboard"
    billboard.Adornee = head
    billboard.AlwaysOnTop = SHOW_THROUGH_WALLS
    billboard.Size = UDim2.new(0, BAR_WIDTH, 0, 52)
    billboard.StudsOffset = BILLBOARD_OFFSET
    billboard.MaxDistance = 10000000
    billboard.Enabled = espVisible
    billboard.Parent = head

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundTransparency = 1
    frame.Parent = billboard

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "ESPName"
    nameLabel.Size = UDim2.new(1, -8, 0, 20)
    nameLabel.Position = UDim2.new(0, 4, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = tostring(player and player.Name or "NPC")
    nameLabel.TextColor3 = Color3.fromRGB(220, 180, 255)
    nameLabel.Font = FONT
    nameLabel.TextSize = NAME_TEXT_SIZE
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.Parent = frame

    local barBack = Instance.new("Frame")
    barBack.Name = "HealthBack"
    barBack.Size = UDim2.new(1, -8, 0, 3)
    barBack.Position = UDim2.new(0, 4, 0, 20)
    barBack.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    barBack.BorderSizePixel = 0
    barBack.Parent = frame
    Instance.new("UICorner", barBack).CornerRadius = UDim.new(0, 8)

    local healthFill = Instance.new("Frame")
    healthFill.Name = "HealthFill"
    healthFill.Size = UDim2.new(1, 0, 1, 0)
    healthFill.BorderSizePixel = 0
    healthFill.BackgroundColor3 = Color3.fromRGB(40, 200, 100)
    healthFill.Parent = barBack
    Instance.new("UICorner", healthFill).CornerRadius = UDim.new(0, 8)

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(18, 18, 18)
    stroke.Thickness = 1.5
    stroke.Parent = barBack

    return {billboard = billboard, nameLabel = nameLabel, healthBack = barBack, healthFill = healthFill}
end

local function updateHudForCharacter(trackedInfo, humanoid)
    if not trackedInfo or not humanoid then return end
    local pct = math.clamp(humanoid.Health / math.max(1, humanoid.MaxHealth), 0, 1)
    trackedInfo.healthFill:TweenSize(UDim2.new(pct, 0, 1, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.12, true)
    trackedInfo.healthFill.BackgroundColor3 = pct > 0.5 and Color3.fromRGB(80, 255, 80) or (pct > 0.3 and Color3.fromRGB(255, 255, 0) or Color3.fromRGB(255, 80, 80))
end

local function trackCharacter(character, player)
    if player == LocalPlayer then return end
    if not character then return end

    if tracked[character] then
        if tracked[character].info and tracked[character].info.billboard then
            tracked[character].info.billboard.Enabled = espVisible
        end
        return
    end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        humanoid = character:WaitForChild("Humanoid", 2)
        if not humanoid then return end
    end

    local info = createBillboardForCharacter(character, player)
    if not info then return end

    updateHudForCharacter(info, humanoid)

    local c1 = humanoid.HealthChanged:Connect(function()
        updateHudForCharacter(info, humanoid)
    end)
    local c2 = humanoid.Died:Connect(function()
        if info.billboard and info.billboard.Parent then
            info.billboard:Destroy()
        end
    end)

    tracked[character] = {info = info, connections = {c1, c2}}
end

local function untrackCharacter(character)
    if not character then return end
    local t = tracked[character]
    if t then
        for _, c in ipairs(t.connections) do
            if c and c.Disconnect then pcall(function() c:Disconnect() end) end
        end
        if t.info and t.info.billboard and t.info.billboard.Parent then
            t.info.billboard:Destroy()
        end
        tracked[character] = nil
    end
end

local function setupESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if player.Character then
                trackCharacter(player.Character, player)
            end
            player.CharacterAdded:Connect(function(c)
                task.defer(function() trackCharacter(c, player) end)
            end)
            player.CharacterRemoving:Connect(function(c)
                untrackCharacter(c)
            end)
        end
    end

    Players.PlayerAdded:Connect(function(player)
        if player ~= LocalPlayer then
            player.CharacterAdded:Connect(function(c)
                task.defer(function() trackCharacter(c, player) end)
            end)
            player.CharacterRemoving:Connect(function(c)
                untrackCharacter(c)
            end)
        end
    end)
end

-- =============================================================================
-- OTIMIZAÇÃO DE MOVIMENTO (SILENT AIM)
-- =============================================================================
local dashActive = false
local dashCooldown = 0

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.X then
        dashActive = true
        dashCooldown = tick()
    end
end)

UserInputService.InputEnded:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.X then
        task.delay(0.5, function()
            dashActive = false
        end)
    end
end)

local function IsCharacterBeingMoved()
    if dashActive then return true end
    if tick() - dashCooldown < 0.6 then return true end
    return false
end

-- =============================================================================
-- BLACKLIST DE SCRIPTS OTIMIZADA
-- =============================================================================
local BLACKLIST = {
    "Camera", "PlayerScripts", "RobloxCharacterSounds",
    "AnimateScript", "Animate", "Health", "HumanoidDescription",
    "ControlScript", "PlayerModule", "CameraModule"
}

local function IsBlacklisted(scriptName)
    if not scriptName then return true end
    for _, v in ipairs(BLACKLIST) do
        if string.find(scriptName, v) then return true end
    end
    return false
end

-- =============================================================================
-- FOV CIRCLE (Renderização Dinâmica via Drawing API)
-- =============================================================================
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1.5
fovCircle.Color = RED
fovCircle.Filled = false
fovCircle.NumSides = 64
fovCircle.Radius = FOV_RADIUS
fovCircle.Visible = false

RunService.RenderStepped:Connect(function()
    if not FOV_ENABLED or not FOV_VISIBLE then
        fovCircle.Visible = false
        return
    end
    local center = Camera.ViewportSize / 2
    fovCircle.Position = center
    fovCircle.Radius = FOV_RADIUS
    fovCircle.Filled = FOV_FILLED
    fovCircle.Visible = TELEGUIADO_ENABLED
end)

local function IsInFOV(targetRoot)
    local screenPos, onScreen = Camera:WorldToViewportPoint(targetRoot.Position)
    if not onScreen then return false end
    local center = Camera.ViewportSize / 2
    return (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude <= FOV_RADIUS
end

-- =============================================================================
-- VISIBLE CHECK (Raycast)
-- =============================================================================
local function IsVisible(targetRoot)
    if not VISIBLE_CHECK then return true end
    local char = LocalPlayer.Character
    if not char then return false end
    local origin = Camera.CFrame.Position
    local direction = targetRoot.Position - origin
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {char, targetRoot.Parent}
    local result = Workspace:Raycast(origin, direction, params)
    return result == nil
end

-- =============================================================================
-- ALGORITMO DE BUSCA DE ALVOS
-- =============================================================================
local function GetClosestEnemy()
    local char = LocalPlayer.Character
    local myRoot = char and char:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end
    local center = Camera.ViewportSize / 2
    local best, shortest = nil, math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        if not player.Character then continue end
        
        if TEAM_CHECK and player.Team and LocalPlayer.Team then
            if player.Team == LocalPlayer.Team then continue end
        end
        
        local root = player.Character:FindFirstChild("HumanoidRootPart")
        if not root then continue end
        
        if HEALTH_CHECK then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0 then continue end
        end
        
        if FOV_ENABLED and not IsInFOV(root) then continue end
        if not IsVisible(root) then continue end
        
        local dist
        if PRIORITY_CROSSHAIR then
            local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
            if not onScreen then continue end
            dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
        end
        
        if not PRIORITY_CROSSHAIR or dist == nil then
            dist = (root.Position - myRoot.Position).Magnitude
            if dist > MAX_DISTANCE then continue end
        end
        
        if dist < shortest then
            shortest = dist
            best = player
        end
    end
    return best
end

local function GetHitPartPosition(character)
    local part = character:FindFirstChild(HIT_PART)
    if not part then
        part = character:FindFirstChild("HumanoidRootPart")
    end
    return part and part.Position or nil
end

-- =============================================================================
-- HOOK CFRAME (Redirecionamento Teleguiado)
-- =============================================================================
local oldCFrame
oldCFrame = hookfunction(CFrame.new, function(...)
    local args = {...}
    if IsCharacterBeingMoved() then
        return oldCFrame(...)
    end
    if TELEGUIADO_ENABLED and #args >= 2 then
        local ok, env = pcall(getfenv, 2)
        if ok and env and env.script then
            local name = tostring(env.script:GetFullName())
            if IsBlacklisted(name) then return oldCFrame(...) end
            if typeof(args[1]) == "Vector3" and typeof(args[2]) == "Vector3" then
                local target = GetClosestEnemy()
                if target and target.Character then
                    local pos = GetHitPartPosition(target.Character)
                    if pos then
                        -- Modifica o destino (ponto para onde o CFrame olha)
                        args[2] = pos
                        return oldCFrame(args[1], args[2])
                    end
                end
            end
        end
    end
    return oldCFrame(...)
end)

-- =============================================================================
-- HUB GUI (INTERFACE VISUAL ALTERADA)
-- =============================================================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SilentEngineHub"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 310, 0, 540)
mainFrame.Position = UDim2.new(0.5, -155, 0.5, -270)
mainFrame.BackgroundColor3 = DARK_BG
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Visible = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = BORDER_COL
mainStroke.Thickness = 1
mainStroke.Parent = mainFrame

-- TITLE BAR
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(16, 16, 16)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 8)

local accentBar = Instance.new("Frame")
accentBar.Size = UDim2.new(1, 0, 0, 2)
accentBar.Position = UDim2.new(0, 0, 1, -2)
accentBar.BackgroundColor3 = RED
accentBar.BorderSizePixel = 0
accentBar.Parent = titleBar

local redDot = Instance.new("Frame")
redDot.Size = UDim2.new(0, 8, 0, 8)
redDot.Position = UDim2.new(0, 14, 0.5, -4)
redDot.BackgroundColor3 = RED
redDot.BorderSizePixel = 0
redDot.Parent = titleBar
Instance.new("UICorner", redDot).CornerRadius = UDim.new(1, 0)

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 1, 0)
titleLabel.Position = UDim2.new(0, 28, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ Silent Engine"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 14
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -38, 0.5, -16)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(100, 100, 100)
closeBtn.TextSize = 13
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)

-- TAB SYSTEM
local TABS = {"Silent Aim", "Prediction", "FOV"}
local tabFrames = {}
local tabBtns = {}
local tabIndicators = {}

local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, 0, 0, 36)
tabBar.Position = UDim2.new(0, 0, 0, 40)
tabBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
tabBar.BorderSizePixel = 0
tabBar.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.Parent = tabBar

local tabSep = Instance.new("Frame")
tabSep.Size = UDim2.new(1, 0, 0, 1)
tabSep.Position = UDim2.new(0, 0, 1, -1)
tabSep.BackgroundColor3 = BORDER_COL
tabSep.BorderSizePixel = 0
tabSep.Parent = tabBar

local contentArea = Instance.new("Frame")
contentArea.Size = UDim2.new(1, 0, 1, -76)
contentArea.Position = UDim2.new(0, 0, 0, 76)
contentArea.BackgroundTransparency = 1
contentArea.Parent = mainFrame

local function SwitchTab(name)
    for n, frame in pairs(tabFrames) do frame.Visible = (n == name) end
    for n, btn in pairs(tabBtns) do 
        btn.TextColor3 = (n == name) and RED or GRAY_TEXT
        btn.Font = (n == name) and Enum.Font.GothamBold or Enum.Font.Gotham
    end
    for n, ind in pairs(tabIndicators) do ind.BackgroundTransparency = (n == name) and 0 or 1 end
end

for i, tabName in ipairs(TABS) do
    local tabW = 310 / #TABS
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, tabW, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = tabName
    btn.TextColor3 = i == 1 and RED or GRAY_TEXT
    btn.TextSize = 12
    btn.Font = i == 1 and Enum.Font.GothamBold or Enum.Font.Gotham
    btn.LayoutOrder = i
    btn.Parent = tabBar
    tabBtns[tabName] = btn

    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0.8, 0, 0, 2)
    indicator.AnchorPoint = Vector2.new(0.5, 1)
    indicator.Position = UDim2.new(0.5, 0, 1, 0)
    indicator.BackgroundColor3 = RED
    indicator.BackgroundTransparency = i == 1 and 0 or 1
    indicator.BorderSizePixel = 0
    indicator.Parent = btn
    tabIndicators[tabName] = indicator

    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 3
    scroll.ScrollBarImageColor3 = RED
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.Visible = i == 1
    scroll.Parent = contentArea
    tabFrames[tabName] = scroll

    local listLayout = Instance.new("UIListLayout")
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Padding = UDim.new(0, 6)
    listLayout.Parent = scroll
    
    listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 20)
    end)

    local pad = Instance.new("UIPadding")
    pad.PaddingLeft = UDim.new(0, 12)
    pad.PaddingRight = UDim.new(0, 12)
    pad.PaddingTop = UDim.new(0, 10)
    pad.PaddingBottom = UDim.new(0, 10)
    pad.Parent = scroll

    btn.MouseButton1Click:Connect(function() SwitchTab(tabName) end)
end

-- =============================================================================
-- GERADORES DE COMPONENTES INTERNOS DA GUI
-- =============================================================================
local function MakeSectionLabel(parent, text, order)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 22)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = RED
    lbl.TextSize = 10
    lbl.Font = Enum.Font.GothamBold
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.LayoutOrder = order
    lbl.Parent = parent
end

local function MakeSubText(parent, text, order)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 30)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = GRAY_TEXT
    lbl.TextSize = 10
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextWrapped = true
    lbl.LayoutOrder = order
    lbl.Parent = parent
end

local function MakeToggle(parent, labelText, defaultValue, order, callback)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 38)
    row.BackgroundColor3 = PANEL_BG
    row.BorderSizePixel = 0
    row.LayoutOrder = order
    row.Parent = parent
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -58, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = labelText
    lbl.TextColor3 = TEXT_WHITE
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = row

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0, 34, 0, 18)
    bg.Position = UDim2.new(1, -44, 0.5, -9)
    bg.BackgroundColor3 = defaultValue and RED or TOGGLE_OFF
    bg.BorderSizePixel = 0
    bg.Parent = row
    Instance.new("UICorner", bg).CornerRadius = UDim.new(1, 0)

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 12, 0, 12)
    dot.Position = defaultValue and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6)
    dot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    dot.BorderSizePixel = 0
    dot.Parent = bg
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local state = defaultValue
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.Parent = row
    
    btn.MouseButton1Click:Connect(function()
        state = not state
        bg.BackgroundColor3 = state and RED or TOGGLE_OFF
        dot.Position = state and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6)
        callback(state)
    end)
end

local function MakeSlider(parent, labelText, minVal, maxVal, defaultVal, order, callback)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 54)
    row.BackgroundColor3 = PANEL_BG
    row.BorderSizePixel = 0
    row.LayoutOrder = order
    row.Parent = parent
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0.6, 0, 0, 20)
    lbl.Position = UDim2.new(0, 12, 0, 6)
    lbl.BackgroundTransparency = 1
    lbl.Text = labelText
    lbl.TextColor3 = TEXT_WHITE
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = row

    local valLbl = Instance.new("TextLabel")
    valLbl.Size = UDim2.new(0.3, 0, 0, 20)
    valLbl.Position = UDim2.new(0.7, -12, 0, 6)
    valLbl.BackgroundTransparency = 1
    valLbl.Text = tostring(defaultVal)
    valLbl.TextColor3 = GRAY_TEXT
    valLbl.TextSize = 11
    valLbl.Font = Enum.Font.Gotham
    valLbl.TextXAlignment = Enum.TextXAlignment.Right
    valLbl.Parent = row

    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, -24, 0, 4)
    track.Position = UDim2.new(0, 12, 0, 36)
    track.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    track.BorderSizePixel = 0
    track.Parent = track.Parent or row

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = RED
    fill.BorderSizePixel = 0
    fill.Parent = track

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 10, 0, 10)
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.Position = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 0.5, 0)
    knob.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
    knob.Parent = track
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

    local dragging = false
    local hitArea = Instance.new("TextButton")
    hitArea.Size = UDim2.new(1, 0, 0, 24)
    hitArea.Position = UDim2.new(0, 0, 0, 26)
    hitArea.BackgroundTransparency = 1
    hitArea.Text = ""
    hitArea.Parent = row

    hitArea.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local rel = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
            local value = math.floor(minVal + (maxVal - minVal) * rel)
            fill.Size = UDim2.new(rel, 0, 1, 0)
            knob.Position = UDim2.new(rel, 0, 0.5, 0)
            valLbl.Text = tostring(value)
            callback(value)
        end
    end)
end

local function MakeDropdown(parent, labelText, options, defaultOpt, order, callback)
    local isOpen = false
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, 0, 0, 54)
    container.BackgroundColor3 = PANEL_BG
    container.BorderSizePixel = 0
    container.LayoutOrder = order
    container.Parent = parent
    Instance.new("UICorner", container).CornerRadius = UDim.new(0, 6)

    local catLbl = Instance.new("TextLabel")
    catLbl.Size = UDim2.new(1, -20, 0, 16)
    catLbl.Position = UDim2.new(0, 12, 0, 6)
    catLbl.BackgroundTransparency = 1
    catLbl.Text = labelText
    catLbl.TextColor3 = GRAY_TEXT
    catLbl.TextSize = 10
    catLbl.Parent = container

    local dropBtn = Instance.new("TextButton")
    dropBtn.Size = UDim2.new(1, -24, 0, 24)
    dropBtn.Position = UDim2.new(0, 12, 0, 22)
    dropBtn.BackgroundColor3 = Color3.fromRGB(26, 26, 26)
    dropBtn.Text = "  " .. defaultOpt
    dropBtn.TextColor3 = TEXT_WHITE
    dropBtn.TextSize = 12
    dropBtn.Font = Enum.Font.Gotham
    dropBtn.TextXAlignment = Enum.TextXAlignment.Left
    dropBtn.Parent = container
    Instance.new("UICorner", dropBtn).CornerRadius = UDim.new(0, 4)

    local list = Instance.new("Frame")
    list.Size = UDim2.new(1, -24, 0, #options * 24)
    list.Position = UDim2.new(0, 12, 1, 2)
    list.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    list.Visible = false
    list.ZIndex = 100 -- ZIndex alto garante que sobreponha outros itens ao abrir
    list.Parent = container
    Instance.new("UIListLayout", list)

    for _, opt in ipairs(options) do
        local optBtn = Instance.new("TextButton")
        optBtn.Size = UDim2.new(1, 0, 0, 24)
        optBtn.BackgroundTransparency = 1
        optBtn.Text = "  " .. opt
        optBtn.TextColor3 = TEXT_WHITE
        optBtn.TextSize = 11
        optBtn.Font = Enum.Font.Gotham
        optBtn.TextXAlignment = Enum.TextXAlignment.Left
        optBtn.Parent = list

        optBtn.MouseButton1Click:Connect(function()
            dropBtn.Text = "  " .. opt
            isOpen = false
            list.Visible = false
            callback(opt)
        end)
    end

    dropBtn.MouseButton1Click:Connect(function()
        isOpen = not isOpen
        list.Visible = isOpen
    end)
end

-- =============================================================================
-- VINCULAÇÃO E CONSTRUÇÃO DAS ABAS DA INTERFACE
-- =============================================================================
local saTab = tabFrames["Silent Aim"]
MakeSectionLabel(saTab, "SILENT AIM OPTIONS", 1)
MakeToggle(saTab, "Enabled", TELEGUIADO_ENABLED, 2, function(v) TELEGUIADO_ENABLED = v end)
MakeToggle(saTab, "Team Check", TEAM_CHECK, 3, function(v) TEAM_CHECK = v end)
MakeToggle(saTab, "Visible Check", VISIBLE_CHECK, 4, function(v) VISIBLE_CHECK = v end)
MakeToggle(saTab, "Health Check", HEALTH_CHECK, 5, function(v) HEALTH_CHECK = v end)
MakeSlider(saTab, "Distance", 100, 10000, MAX_DISTANCE, 6, function(v) MAX_DISTANCE = v end)
MakeDropdown(saTab, "Hit Part", {"Head", "Torso", "HumanoidRootPart"}, HIT_PART, 7, function(v) HIT_PART = v end)

MakeSectionLabel(saTab, "PRIORITY SETTINGS", 8)
MakeToggle(saTab, "Closest to Crosshair", PRIORITY_CROSSHAIR, 9, function(v) PRIORITY_CROSSHAIR = v end)
MakeSubText(saTab, "Prioritizes enemies closest to your screen center crosshair.", 10)

-- PREDICTION TAB
local predTab = tabFrames["Prediction"]
MakeSectionLabel(predTab, "PREDICTION CALIBRATION", 1)
MakeSubText(predTab, "Prediction tracking algorithms will interface automatically with bullet velocity objects here.", 2)

-- FOV TAB
local fovTab = tabFrames["FOV"]
MakeSectionLabel(fovTab, "FOV VISUALIZER", 1)
MakeToggle(fovTab, "Enabled", FOV_ENABLED, 2, function(v) FOV_ENABLED = v end)
MakeToggle(fovTab, "Glow", FOV_GLOW, 3, function(v) fovCircle.Thickness = v and 2.5 or 1.5 end)
MakeToggle(fovTab, "Filled", FOV_FILLED, 4, function(v) fovCircle.Filled = v end)
MakeSlider(fovTab, "Size / Radius", 50, 500, FOV_RADIUS, 5, function(v) FOV_RADIUS = v end)

-- =============================================================================
-- CONTROLE DE ENTRADAS (INPUTS GLOBAIS)
-- =============================================================================
local OPEN_KEY = Enum.KeyCode.RightShift

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Tecla para abrir/fechar menu principal
    if input.KeyCode == OPEN_KEY then
        mainFrame.Visible = not mainFrame.Visible
    end
    
    -- Tecla para ligar/desligar ESP individualmente
    if input.KeyCode == TOGGLE_KEY_ESP then
        if debounceEsp then return end
        debounceEsp = true
        toggleEspVisibility()
        task.delay(DEBOUNCE_KEY, function() debounceEsp = false end)
    end
end)

-- Initialize Systems
setupESP()

print("--------------------------------------------------")
print(" ✅ KING HUB INTEGRADO E CARREGADO COM SUCESSO!")
print(" SHIFT DIREITO → Abrir/Fechar menu principal")
print(" TECLA 'U'     → Ativar/Desativar Visibilidade do ESP")
print("--------------------------------------------------")
