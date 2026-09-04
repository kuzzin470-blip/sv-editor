# V0.2
версия 0.2 официально вышла!
требует проверки, ещё не запускалась
# скрипт

```lua
local BG_COLOR = Color3.fromRGB(43, 43, 43)
local TEXT_COLOR = Color3.fromRGB(255, 255, 255)
local ACCENT_COLOR = Color3.fromRGB(0, 200, 255)
local LINE_COLOR = Color3.fromRGB(255, 100, 100)
local BOX_STROKE_THICKNESS = 2
local LINE_THICKNESS = 3

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local ContextActionService = game:GetService("ContextActionService")
local Camera = workspace.CurrentCamera

local localPlayer = Players.LocalPlayer
local PlayerGui = localPlayer:WaitForChild("PlayerGui")

local AdminEvent = ReplicatedStorage:WaitForChild("AdminEvent")
local AdminRequest = ReplicatedStorage:WaitForChild("AdminRequest")

-- Конфигурация системы
local UI_PASSWORD = "admin123"
local MAX_DISTANCE_CHECK = 1000
local RENDER_UPDATE_RATE = 60

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TPSC_Admin_UI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local header = Instance.new("Frame")
header.Name = "Header"
header.Size = UDim2.new(1, 0, 0, 48)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = BG_COLOR
header.BorderSizePixel = 0
header.Parent = screenGui

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 6)
headerCorner.Parent = header

local headerLabel = Instance.new("TextLabel")
headerLabel.Name = "HeaderLabel"
headerLabel.Size = UDim2.new(1, -24, 1, 0)
headerLabel.Position = UDim2.new(0, 12, 0, 0)
headerLabel.BackgroundTransparency = 1
headerLabel.Text = "TPSC SV editor"
headerLabel.TextColor3 = TEXT_COLOR
headerLabel.Font = Enum.Font.SourceSansBold
headerLabel.TextSize = 20
headerLabel.TextXAlignment = Enum.TextXAlignment.Left
headerLabel.Parent = header

local topCurtain = Instance.new("Frame")
topCurtain.Name = "TopCurtain"
topCurtain.Size = UDim2.new(1, 0, 0, 120)
topCurtain.Position = UDim2.new(0, 0, -1, 0)
topCurtain.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
topCurtain.BorderSizePixel = 0
topCurtain.Parent = screenGui

local topCurtainCorner = Instance.new("UICorner", topCurtain)
topCurtainCorner.CornerRadius = UDim.new(0, 0)

local passwordModal
local function createPasswordModal()
    local modal = Instance.new("Frame")
    modal.Name = "PasswordModal"
    modal.Size = UDim2.new(0, 380, 0, 140)
    modal.Position = UDim2.new(0.5, -190, 0.5, -70)
    modal.BackgroundColor3 = BG_COLOR
    modal.BorderSizePixel = 0
    modal.AnchorPoint = Vector2.new(0, 0)
    modal.Parent = screenGui

    local uic = Instance.new("UICorner")
    uic.CornerRadius = UDim.new(0, 8)
    uic.Parent = modal

    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -20, 0, 30)
    title.Position = UDim2.new(0, 10, 0, 10)
    title.BackgroundTransparency = 1
    title.Text = "Введите ключ доступа"
    title.TextColor3 = TEXT_COLOR
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 18
    title.Parent = modal

    local box = Instance.new("TextBox")
    box.Name = "PassBox"
    box.Size = UDim2.new(1, -20, 0, 32)
    box.Position = UDim2.new(0, 10, 0, 50)
    box.PlaceholderText = "Ключ"
    box.ClearTextOnFocus = false
    box.Text = ""
    box.TextColor3 = TEXT_COLOR
    box.BackgroundColor3 = Color3.fromRGB(25,25,25)
    box.BorderSizePixel = 0
    box.Parent = modal
    local boxCorner = Instance.new("UICorner", box)
    boxCorner.CornerRadius = UDim.new(0, 6)

    local ok = Instance.new("TextButton")
    ok.Name = "OK"
    ok.Size = UDim2.new(0, 120, 0, 32)
    ok.Position = UDim2.new(1, -130, 1, -10)
    ok.AnchorPoint = Vector2.new(0, 0)
    ok.Text = "Подтвердить"
    ok.TextColor3 = TEXT_COLOR
    ok.BackgroundColor3 = ACCENT_COLOR
    ok.Parent = modal
    local okCorner = Instance.new("UICorner", ok)
    okCorner.CornerRadius = UDim.new(0, 6)

    local cancel = Instance.new("TextButton")
    cancel.Name = "Cancel"
    cancel.Size = UDim2.new(0, 80, 0, 32)
    cancel.Position = UDim2.new(1, -220, 1, -10)
    cancel.Text = "Отмена"
    cancel.TextColor3 = TEXT_COLOR
    cancel.BackgroundColor3 = Color3.fromRGB(80,80,80)
    cancel.Parent = modal
    local cancelCorner = Instance.new("UICorner", cancel)
    cancelCorner.CornerRadius = UDim.new(0, 6)

    local info = Instance.new("TextLabel")
    info.Name = "Info"
    info.Size = UDim2.new(1, -20, 0, 18)
    info.Position = UDim2.new(0, 10, 0, 92)
    info.BackgroundTransparency = 1
    info.Text = ""
    info.TextColor3 = Color3.fromRGB(255, 120, 120)
    info.Font = Enum.Font.SourceSans
    info.TextSize = 14
    info.TextXAlignment = Enum.TextXAlignment.Left
    info.Parent = modal

    passwordModal = modal

    ok.MouseButton1Click:Connect(function()
        if box.Text == UI_PASSWORD then
            modal:Destroy()
            passwordModal = nil
            playCurtainAndShowMain()
        else
            info.Text = "Неверный ключ"
            local orig = modal.Position
            local tweenInfo = TweenInfo.new(0.06, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, 2, true, 0)
            TweenService:Create(modal, tweenInfo, {Position = orig + UDim2.new(0, 8, 0, 0)}):Play()
        end
    end)

    cancel.MouseButton1Click:Connect(function()
        modal:Destroy()
        passwordModal = nil
    end)
end

local mainBar
local playersScrollingFrame
local selectedUserId = nil
local playerButtonRefs = {}

local function createMainBar()
    local bar = Instance.new("Frame")
    bar.Name = "MainBar"
    bar.Size = UDim2.new(1, -40, 0, 64)
    bar.Position = UDim2.new(0, 20, 0, 56)
    bar.BackgroundColor3 = BG_COLOR
    bar.BorderSizePixel = 0
    bar.AnchorPoint = Vector2.new(0,0)
    bar.Parent = screenGui

    local barCorner = Instance.new("UICorner", bar)
    barCorner.CornerRadius = UDim.new(0, 10)

    local scroll = Instance.new("ScrollingFrame")
    scroll.Name = "PlayersList"
    scroll.Size = UDim2.new(0.7, -12, 1, -12)
    scroll.Position = UDim2.new(0, 8, 0, 6)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 6
    scroll.BorderSizePixel = 0
    scroll.Parent = bar

    local listLayout = Instance.new("UIListLayout", scroll)
    listLayout.FillDirection = Enum.FillDirection.Horizontal
    listLayout.Padding = UDim.new(0, 6)
    listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
    listLayout.VerticalAlignment = Enum.VerticalAlignment.Center

    local uiPadding = Instance.new("UIPadding", scroll)
    uiPadding.PaddingLeft = UDim.new(0, 6)
    uiPadding.PaddingRight = UDim.new(0, 6)
    uiPadding.PaddingTop = UDim.new(0, 6)
    uiPadding.PaddingBottom = UDim.new(0, 6)

    local actions = Instance.new("Frame")
    actions.Name = "Actions"
    actions.Size = UDim2.new(0.3, -12, 1, -12)
    actions.Position = UDim2.new(0.7, 12, 0, 6)
    actions.BackgroundTransparency = 1
    actions.Parent = bar

    local actionsLayout = Instance.new("UIListLayout", actions)
    actionsLayout.FillDirection = Enum.FillDirection.Horizontal
    actionsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
    actionsLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    actionsLayout.Padding = UDim.new(0, 8)

    local function makeButton(text, bg, width)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, width or 110, 0, 40)
        btn.BackgroundColor3 = bg
        btn.Text = text
        btn.TextColor3 = TEXT_COLOR
        btn.Font = Enum.Font.SourceSansBold
        btn.TextSize = 14
        btn.BorderSizePixel = 0
        local c = Instance.new("UICorner", btn)
        c.CornerRadius = UDim.new(0, 6)
        return btn
    end

    local tpBtn = makeButton("Teleport", ACCENT_COLOR, 110)
    tpBtn.Parent = actions

    local espBtn = makeButton("Toggle ESP", Color3.fromRGB(80,80,80), 120)
    espBtn.Parent = actions

    local destroyBtn = makeButton("Destroy GUI", Color3.fromRGB(180,50,50), 120)
    destroyBtn.Parent = actions

    tpBtn.MouseButton1Click:Connect(function()
        if not selectedUserId then return end
        pcall(function() AdminRequest:FireServer("teleport", selectedUserId) end)
    end)

    espBtn.MouseButton1Click:Connect(function()
        if not selectedUserId then return end
        pcall(function() AdminRequest:FireServer("esp_toggle", selectedUserId) end)
    end)

    destroyBtn.MouseButton1Click:Connect(function()
        pcall(function() AdminRequest:FireServer("esp_clear") end)
        cleanupAndDestroy()
    end)

    playersScrollingFrame = scroll
    mainBar = bar
end

local function makePlayerEntry(pl)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 140, 0, 44)
    btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    btn.BorderSizePixel = 0
    btn.TextColor3 = TEXT_COLOR
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.Text = pl.Name .. "  [" .. tostring(pl.UserId) .. "]"
    btn.Parent = playersScrollingFrame
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 8)

    btn.MouseButton1Click:Connect(function()
        selectedUserId = pl.UserId
        for uid, b in pairs(playerButtonRefs) do
            if b and b.Parent then
                b.BackgroundColor3 = Color3.fromRGB(30,30,30)
            end
        end
        btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    end)

    playerButtonRefs[pl.UserId] = btn
end

local function refreshPlayerList()
    for _, child in ipairs(playersScrollingFrame:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    playerButtonRefs = {}

    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= localPlayer then
            makePlayerEntry(pl)
        end
    end
end

Players.PlayerAdded:Connect(function(pl)
    if mainBar then
        wait(0.05)
        if playersScrollingFrame then
            makePlayerEntry(pl)
        end
    end
end)

Players.PlayerRemoving:Connect(function(pl)
    local b = playerButtonRefs[pl.UserId]
    if b and b.Parent then b:Destroy() end
    playerButtonRefs[pl.UserId] = nil
    if selectedUserId == pl.UserId then selectedUserId = nil end
end)

local tracked = {}
local renderBound = false

local function worldPointToScreenVec3(worldPos)
    local vpPoint, onScreen = Camera:WorldToViewportPoint(worldPos)
    return Vector3.new(vpPoint.X, vpPoint.Y, vpPoint.Z), onScreen
end

local function getCharacterBounds(character)
    if not character then return end
    local minV, maxV
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Size.Magnitude > 0 then
            local cfa = part.CFrame
            local sx, sy, sz = part.Size.X/2, part.Size.Y/2, part.Size.Z/2
            local corners = {
                cfa * Vector3.new( sx,  sy,  sz),
                cfa * Vector3.new( sx,  sy, -sz),
                cfa * Vector3.new( sx, -sy,  sz),
                cfa * Vector3.new( sx, -sy, -sz),
                cfa * Vector3.new(-sx,  sy,  sz),
                cfa * Vector3.new(-sx,  sy, -sz),
                cfa * Vector3.new(-sx, -sy,  sz),
                cfa * Vector3.new(-sx, -sy, -sz),
            }
            for _, v in ipairs(corners) do
                if not minV then
                    minV = v
                    maxV = v
                else
                    minV = Vector3.new(math.min(minV.X, v.X), math.min(minV.Y, v.Y), math.min(minV.Z, v.Z))
                    maxV = Vector3.new(math.max(maxV.X, v.X), math.max(maxV.Y, v.Y), math.max(maxV.Z, v.Z))
                end
            end
        end
    end
    if not minV then return end
    local center = (minV + maxV) * 0.5
    return center, minV, maxV
end

local function createESPForPlayer(targetPlayer)
    if not targetPlayer then return end
    local uid = targetPlayer.UserId
    if tracked[uid] then return end

    local container = Instance.new("Frame")
    container.Name = "ESP_" .. tostring(uid)
    container.Size = UDim2.new(0, 0, 0, 0)
    container.Position = UDim2.new(0,0,0,0)
    container.BackgroundTransparency = 1
    container.Parent = screenGui

    local box = Instance.new("Frame")
    box.Name = "Box"
    box.BackgroundTransparency = 1
    box.BorderSizePixel = 0
    box.Parent = container
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = BOX_STROKE_THICKNESS
    stroke.Color = ACCENT_COLOR
    stroke.Parent = box

    local line = Instance.new("Frame")
    line.Name = "Line"
    line.AnchorPoint = Vector2.new(0, 0.5)
    line.BackgroundColor3 = LINE_COLOR
    line.BorderSizePixel = 0
    line.Parent = container

    local label = Instance.new("TextLabel")
    label.Name = "InfoLabel"
    label.BackgroundTransparency = 0.6
    label.BackgroundColor3 = Color3.fromRGB(0,0,0)
    label.TextColor3 = TEXT_COLOR
    label.Font = Enum.Font.SourceSansSemibold
    label.TextSize = 14
    label.Size = UDim2.new(0, 220, 0, 24)
    label.AnchorPoint = Vector2.new(0.5, 0)
    label.Parent = container

    tracked[uid] = {
        container = container,
        box = box,
        stroke = stroke,
        line = line,
        label = label,
        targetPlayer = targetPlayer,
    }
end

local function removeESPForUserId(uid)
    local t = tracked[uid]
    if not t then return end
    if t.container and t.container.Parent then t.container:Destroy() end
    tracked[uid] = nil
end

local function updateOneESP(t)
    local targetPlayer = t.targetPlayer
    if not targetPlayer or not targetPlayer.Parent then
        return false
    end
    local char = targetPlayer.Character
    if not char then
        t.container.Visible = false
        return true
    end

    local head = char:FindFirstChild("Head")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local refPart = head or hrp
    if not refPart then
        t.container.Visible = false
        return true
    end

    local center, minV, maxV = getCharacterBounds(char)
    if not center then
        t.container.Visible = false
        return true
    end

    local corners = {
        Vector3.new(minV.X, minV.Y, minV.Z),
        Vector3.new(minV.X, minV.Y, maxV.Z),
        Vector3.new(minV.X, maxV.Y, minV.Z),
        Vector3.new(minV.X, maxV.Y, maxV.Z),
        Vector3.new(maxV.X, minV.Y, minV.Z),
        Vector3.new(maxV.X, minV.Y, maxV.Z),
        Vector3.new(maxV.X, maxV.Y, minV.Z),
        Vector3.new(maxV.X, maxV.Y, maxV.Z),
    }

    local screenXs = {}
    local screenYs = {}
    local anyOnScreen = false
    for _, corner in ipairs(corners) do
        local screenP, onScreen = worldPointToScreenVec3(corner)
        table.insert(screenXs, screenP.X)
        table.insert(screenYs, screenP.Y)
        if onScreen then anyOnScreen = true end
    end

    if not anyOnScreen then
        t.box.Visible = false
    else
        t.box.Visible = true
        if #screenXs > 0 then
            local minX = math.min(table.unpack(screenXs))
            local maxX = math.max(table.unpack(screenXs))
            local minY = math.min(table.unpack(screenYs))
            local maxY = math.max(table.unpack(screenYs))

            local vp = Camera.ViewportSize
            minX = math.clamp(minX, 0, vp.X)
            maxX = math.clamp(maxX, 0, vp.X)
            minY = math.clamp(minY, 0, vp.Y)
            maxY = math.clamp(maxY, 0, vp.Y)

            local pos = UDim2.new(0, minX, 0, minY)
            local size = UDim2.new(0, math.max(1, maxX - minX), 0, math.max(1, maxY - minY))

            t.box.Position = pos
            t.box.Size = size
        end
    end

    local headScreen, _ = worldPointToScreenVec3(refPart.Position)
    local vpSize = Camera.ViewportSize
    local centerScreen = Vector2.new(vpSize.X/2, vpSize.Y/2)

    local _, _, z = Camera:WorldToViewportPoint(refPart.Position)
    local targetScreenPos2d = Vector2.new(headScreen.X, headScreen.Y)
    if z < 0 then
        targetScreenPos2d = centerScreen + (centerScreen - targetScreenPos2d)
    end

    local dir = targetScreenPos2d - centerScreen
    local length = dir.Magnitude
    local angle = math.deg(math.atan2(dir.Y, dir.X))

    local line = t.line
    line.Position = UDim2.new(0, centerScreen.X, 0, centerScreen.Y)
    line.Size = UDim2.new(0, math.max(1, length), 0, LINE_THICKNESS)
    line.AnchorPoint = Vector2.new(0, 0.5)
    line.Rotation = angle
    line.BackgroundColor3 = LINE_COLOR
    line.Visible = true

    local infoText = ""
    local dist = 0
    if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
        dist = (localPlayer.Character.HumanoidRootPart.Position - refPart.Position).Magnitude
    end
    infoText = string.format("Д: %d м", math.floor(dist))

    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        infoText = infoText .. string.format("  |  HP: %d/%d", math.floor(hum.Health), math.floor(hum.MaxHealth))
    end

    local labelPos = targetScreenPos2d + Vector2.new(0, -30)
    labelPos = Vector2.new(math.clamp(labelPos.X, 0, vpSize.X - 220), math.clamp(labelPos.Y, 0, vpSize.Y - 24))

    t.label.Position = UDim2.new(0, labelPos.X, 0, labelPos.Y)
    t.label.Text = infoText
    t.label.Visible = true

    t.container.Visible = true
    return true
end

local function startRenderLoop()
    if renderBound then return end
    renderBound = true
    RunService:BindToRenderStep("TPSC_AdminESP_Update", Enum.RenderPriority.Camera.Value - 1, function()
        for uid, t in pairs(tracked) do
            local ok = true
            if not t.targetPlayer or not t.targetPlayer.Parent then
                ok = false
            else
                ok = updateOneESP(t)
            end
            if not ok then removeESPForUserId(uid) end
        end
    end)
end

local function stopRenderLoop()
    if not renderBound then return end
    renderBound = false
    pcall(function() RunService:UnbindFromRenderStep("TPSC_AdminESP_Update") end)
end

local adminEventConn
adminEventConn = AdminEvent.OnClientEvent:Connect(function(action, data)
    if action ~= "esp_update" then return end
    if type(data) ~= "table" then return end

    local wanted = {}
    for _, uid in ipairs(data) do
        if type(uid) == "number" then wanted[uid] = true end
    end

    for uid,_ in pairs(tracked) do
        if not wanted[uid] then removeESPForUserId(uid) end
    end

    for uid,_ in pairs(wanted) do
        if not tracked[uid] then
            local p = Players:GetPlayerByUserId(uid)
            if p then createESPForPlayer(p)
            else
                local conn
                conn = Players.PlayerAdded:Connect(function(pl)
                    if pl.UserId == uid then
                        createESPForPlayer(pl)
                        conn:Disconnect()
                    end
                end)
            end
        end
    end

    if next(tracked) then startRenderLoop() else stopRenderLoop() end
end)

function playCurtainAndShowMain()
    local screenSize = workspace.CurrentCamera.ViewportSize
    topCurtain.Position = UDim2.new(0,0, -1, 0)
    local tweenDown = TweenService:Create(topCurtain, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(0,0,0,0)})
    tweenDown:Play()
    tweenDown.Completed:Wait()

    if not mainBar then createMainBar() end
    refreshPlayerList()
    wait(0.25)
    local tweenUp = TweenService:Create(topCurtain, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(0,0, -1, 0)})
    tweenUp:Play()
end

function cleanupAndDestroy()
    stopRenderLoop()
    if adminEventConn then adminEventConn:Disconnect(); adminEventConn = nil end
    if screenGui and screenGui.Parent then screenGui:Destroy() end
end

createPasswordModal()

local function toggleGuiAction(actionName, inputState, inputObject)
    if inputState ~= Enum.UserInputState.Begin then return end
    if screenGui and screenGui.Parent then
        if passwordModal then
            passwordModal:Destroy()
            passwordModal = nil
        end
        cleanupAndDestroy()
    else
        screenGui.Parent = PlayerGui
        createPasswordModal()
    end
end

ContextActionService:BindAction("TPSC_ToggleGui", toggleGuiAction, false, Enum.KeyCode.M, Enum.KeyCode.LeftControl)
```

