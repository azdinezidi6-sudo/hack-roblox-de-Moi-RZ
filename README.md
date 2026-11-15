-- ============================================================================
-- XENO ULTIMATE GUI - CYBERPUNK NEON ANIMATED EDITION v3.3
-- ============================================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

if CoreGui:FindFirstChild("XenoUltimateGUI") then
    CoreGui.XenoUltimateGUI:Destroy()
end

-- Variables d'état
local flying = false
local flySpeed = 50
local walkSpeedVal = 16
local jumpPowerVal = 50
local bodyPosition, bodyGyro
local following = false
local cameraFollowing = false
local targetPlayer = nil
local espActive = false
local isMinimized = false
local dancing = false
local currentDanceTrack = nil
local noclipActive = false
local spinActive = false
local spinSpeed = 50
local autoLoadActive = false
local connections = {}

local activeScripts = {
    iy = nil,
    coolkid = nil
}

local defaultDances = {
    "rbxassetid://507771019",
    "rbxassetid://507771955",
    "rbxassetid://507772104",
    "rbxassetid://3338044959",
}

--------------------------------------------------------------------------------
-- 1. CONSTRUCTION DU GUI PRINCIPAL (DESIGN ANIMÉ & NÉON)
--------------------------------------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "XenoUltimateGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = CoreGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 500)
mainFrame.Position = UDim2.new(0.05, 0, 0.15, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(6, 8, 12)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local windowGlow = Instance.new("UIStroke", mainFrame)
windowGlow.Color = Color3.fromRGB(0, 242, 254)
windowGlow.Thickness = 2.5
windowGlow.Transparency = 0.1

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

-- Poignée de redimensionnement animée
local resizeBtn = Instance.new("TextButton")
resizeBtn.Size = UDim2.new(0, 22, 0, 22)
resizeBtn.Position = UDim2.new(1, -22, 1, -22)
resizeBtn.BackgroundTransparency = 1
resizeBtn.Text = "◢"
resizeBtn.TextColor3 = Color3.fromRGB(255, 0, 127)
resizeBtn.TextSize = 14
resizeBtn.Parent = mainFrame

local resizing = false
local resizeStartPos, frameStartSize

resizeBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        resizing = true
        resizeStartPos = input.Position
        frameStartSize = mainFrame.AbsoluteSize
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        resizing = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if resizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - resizeStartPos
        local newWidth = math.clamp(frameStartSize.X + delta.X, 260, 600)
        local newHeight = math.clamp(frameStartSize.Y + delta.Y, 300, 900)
        mainFrame.Size = UDim2.new(0, newWidth, 0, newHeight)
    end
end)

local headerBar = Instance.new("Frame")
headerBar.Size = UDim2.new(1, 0, 0, 42)
headerBar.BackgroundColor3 = Color3.fromRGB(12, 16, 26)
headerBar.BorderSizePixel = 0
headerBar.Parent = mainFrame
Instance.new("UICorner", headerBar).CornerRadius = UDim.new(0, 12)

local headerAccent = Instance.new("Frame")
headerAccent.Size = UDim2.new(1, 0, 0, 2)
headerAccent.Position = UDim2.new(0, 0, 1, -2)
headerAccent.BackgroundColor3 = Color3.fromRGB(255, 0, 127)
headerAccent.BorderSizePixel = 0
headerAccent.Parent = headerBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -110, 1, 0)
titleLabel.Position = UDim2.new(0, 12, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ XENO NEON v3.3 (Fait par Réda)"
titleLabel.TextColor3 = Color3.fromRGB(0, 242, 254)
titleLabel.TextSize = 13
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = headerBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 24, 0, 24)
minimizeBtn.Position = UDim2.new(1, -60, 0, 9)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.TextSize = 13
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Parent = headerBar
Instance.new("UICorner", minimizeBtn).CornerRadius = UDim.new(0, 6)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -30, 0, 9)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 20, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 12
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BorderSizePixel = 0
closeBtn.Parent = headerBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

local scrollingFrame = Instance.new("ScrollingFrame")
scrollingFrame.Size = UDim2.new(1, -10, 1, -50)
scrollingFrame.Position = UDim2.new(0, 5, 0, 46)
scrollingFrame.BackgroundTransparency = 1
scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 1120)
scrollingFrame.ScrollBarThickness = 5
scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(255, 0, 127)
scrollingFrame.Parent = mainFrame

-- Animation fluide des couleurs néon en boucle (RGB / Cyberpunk Cycle)
table.insert(connections, RunService.RenderStepped:Connect(function()
    local t = tick() * 2
    local hue = (t % 5) / 5
    local neonColor = Color3.fromHSV(hue, 0.9, 1)
    windowGlow.Color = neonColor
    headerAccent.BackgroundColor3 = neonColor
    titleLabel.TextColor3 = neonColor
end))

local function createCyberButton(text, posY, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 36)
    btn.Position = UDim2.new(0, 5, 0, posY)
    btn.BackgroundColor3 = color or Color3.fromRGB(16, 22, 36)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(240, 245, 255)
    btn.TextSize = 11
    btn.Font = Enum.Font.GothamSemibold
    btn.BorderSizePixel = 0
    btn.Parent = scrollingFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = Color3.fromRGB(0, 242, 254)
    stroke.Transparency = 0.4
    stroke.Thickness = 1.2

    -- Animation au survol de la souris (Hover Effect)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 242, 254)}):Play()
        btn.TextColor3 = Color3.fromRGB(6, 8, 12)
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = color or Color3.fromRGB(16, 22, 36)}):Play()
        btn.TextColor3 = Color3.fromRGB(240, 245, 255)
    end)

    return btn
end

local function createSlider(title, posY, minVal, maxVal, defaultVal, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, -10, 0, 48)
    container.Position = UDim2.new(0, 5, 0, posY)
    container.BackgroundColor3 = Color3.fromRGB(12, 16, 26)
    container.BorderSizePixel = 0
    container.Parent = scrollingFrame
    Instance.new("UICorner", container).CornerRadius = UDim.new(0, 6)

    local stroke = Instance.new("UIStroke", container)
    stroke.Color = Color3.fromRGB(255, 0, 127)
    stroke.Transparency = 0.5

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 8, 0, 4)
    label.BackgroundTransparency = 1
    label.Text = title .. " : " .. defaultVal
    label.TextColor3 = Color3.fromRGB(220, 240, 255)
    label.TextSize = 10
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container

    local sliderBg = Instance.new("Frame")
    sliderBg.Size = UDim2.new(1, -16, 0, 8)
    sliderBg.Position = UDim2.new(0, 8, 0, 30)
    sliderBg.BackgroundColor3 = Color3.fromRGB(22, 30, 46)
    sliderBg.BorderSizePixel = 0
    sliderBg.Parent = container
    Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)

    local sliderFill = Instance.new("Frame")
    sliderFill.Size = UDim2.new((defaultVal - minVal)/(maxVal - minVal), 0, 1, 0)
    sliderFill.BackgroundColor3 = Color3.fromRGB(0, 242, 254)
    sliderFill.BorderSizePixel = 0
    sliderFill.Parent = sliderBg
    Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)

    local dragging = false
    sliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local pos = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
            sliderFill.Size = UDim2.new(pos, 0, 1, 0)
            local val = math.floor(minVal + (maxVal - minVal) * pos)
            label.Text = title .. " : " .. val
            callback(val)
        end
    end)
end

-- Boutons et Contrôles
local autoLoadBtn = createCyberButton("🚀 Auto-Load Global (Persistant) : OFF", 5, Color3.fromRGB(40, 20, 60))
local flyBtn = createCyberButton("🛸 Vol (Fly) : OFF", 45, Color3.fromRGB(15, 50, 80))
createSlider("Vitesse Vol", 85, 10, 300, flySpeed, function(v) flySpeed = v end)

createSlider("Boost Vitesse (WalkSpeed)", 138, 16, 200, walkSpeedVal, function(v) 
    walkSpeedVal = v 
    if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
        player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = v
    end
end)

createSlider("Boost Saut (JumpPower)", 191, 50, 300, jumpPowerVal, function(v) 
    jumpPowerVal = v 
    if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        hum.UseJumpPower = true
        hum.JumpPower = v
    end
end)

local danceBtn = createCyberButton("💃 Danses Aléatoires : OFF", 244, Color3.fromRGB(80, 15, 60))
local espBtn = createCyberButton("🤖 ESP Rôles & Noms : OFF", 284, Color3.fromRGB(60, 15, 80))
local noclipBtn = createCyberButton("👻 Noclip : OFF", 324, Color3.fromRGB(50, 50, 50))
local spinBtn = createCyberButton("🌀 Spin (Tourner) : OFF", 364, Color3.fromRGB(70, 30, 90))
createSlider("Vitesse Spin", 404, 10, 300, spinSpeed, function(v) spinSpeed = v end)

local tpBtn = createCyberButton("⚡ TP à côté de la Cible", 456, Color3.fromRGB(15, 80, 50))
local followBtn = createCyberButton("🏃 Suivre Cible : OFF", 496, Color3.fromRGB(80, 50, 15))
local camBtn = createCyberButton("🎥 Voir Cible (Caméra) : OFF", 536, Color3.fromRGB(50, 20, 80))
local resetCamBtn = createCyberButton("↩️ Retour Caméra sur Moi", 576, Color3.fromRGB(40, 40, 40))
local flingBtn = createCyberButton("🌪️ Fling (FlingV2)", 616, Color3.fromRGB(90, 20, 20))
local coolkidBtn = createCyberButton("🔥 C00lcl4n 2.0 (Coolkid) [Actif]", 656, Color3.fromRGB(120, 20, 20))
local iyBtn = createCyberButton("🛠️ Infinite Yield (IY) [Actif]", 696, Color3.fromRGB(30, 70, 90))
local rejoinBtn = createCyberButton("🔄 Revenir au même Serveur", 736, Color3.fromRGB(40, 70, 40))
local serverHopBtn = createCyberButton("🌐 Changer de Serveur (Server Hop)", 776, Color3.fromRGB(40, 40, 90))

local playerListLabel = Instance.new("TextLabel")
playerListLabel.Size = UDim2.new(1, -10, 0, 25)
playerListLabel.Position = UDim2.new(0, 5, 0, 820)
playerListLabel.BackgroundTransparency = 1
playerListLabel.Text = "👥 SÉLECTIONNER UNE CIBLE (SCROLL ANIMÉ)"
playerListLabel.TextColor3 = Color3.fromRGB(0, 242, 254)
playerListLabel.TextSize = 10
playerListLabel.Font = Enum.Font.GothamBold
playerListLabel.TextXAlignment = Enum.TextXAlignment.Left
playerListLabel.Parent = scrollingFrame

-- Liste des joueurs avec défilement 100% isolé et fluide
local playerListContainer = Instance.new("ScrollingFrame")
playerListContainer.Size = UDim2.new(1, -10, 0, 140)
playerListContainer.Position = UDim2.new(0, 5, 0, 847)
playerListContainer.BackgroundColor3 = Color3.fromRGB(10, 14, 22)
playerListContainer.BorderSizePixel = 0
playerListContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
playerListContainer.ScrollBarThickness = 4
playerListContainer.ScrollingEnabled = true
playerListContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
playerListContainer.Parent = scrollingFrame
Instance.new("UICorner", playerListContainer).CornerRadius = UDim.new(0, 6)

playerListContainer.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        script.Parent = playerListContainer
    end
end)

local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -10, 0, 45)
infoLabel.Position = UDim2.new(0, 5, 0, 996)
infoLabel.BackgroundColor3 = Color3.fromRGB(12, 16, 26)
infoLabel.Text = "Cible Actuelle : Aucune\n[K] = Cacher GUI | [Q] = TP Souris"
infoLabel.TextColor3 = Color3.fromRGB(180, 220, 255)
infoLabel.TextSize = 10
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextWrapped = true
infoLabel.Parent = scrollingFrame
Instance.new("UICorner", infoLabel).CornerRadius = UDim.new(0, 6)

minimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    scrollingFrame.Visible = not isMinimized
    if isMinimized then
        mainFrame.Size = UDim2.new(0, 320, 0, 42)
        minimizeBtn.Text = "□"
    else
        mainFrame.Size = UDim2.new(0, 320, 0, 500)
        minimizeBtn.Text = "_"
    end
end)

closeBtn.MouseButton1Click:Connect(function()
    for _, conn in pairs(connections) do
        if conn then conn:Disconnect() end
    end
    if currentDanceTrack then currentDanceTrack:Stop() end
    noclipActive = false
    spinActive = false
    if screenGui then screenGui:Destroy() end
end)

--------------------------------------------------------------------------------
-- 2. AUTO-LOAD GLOBAL & FENÊTRES DE CONTRÔLE ANIMÉES
--------------------------------------------------------------------------------
autoLoadBtn.MouseButton1Click:Connect(function()
    autoLoadActive = not autoLoadActive
    if autoLoadActive then
        autoLoadBtn.Text = "🚀 Auto-Load Global (Persistant) : ON"
        autoLoadBtn.BackgroundColor3 = Color3.fromRGB(0, 230, 120)
        pcall(function()
            if queue_on_teleport then
                queue_on_teleport([[
                    task.wait(2)
                    loadstring(game:HttpGet("YOUR_SCRIPT_RAW_LINK_HERE"))()
                ]])
            end
        end)
    else
        autoLoadBtn.Text = "🚀 Auto-Load Global (Persistant) : OFF"
        autoLoadBtn.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
    end
end)

local function createScriptControlWindow(title, scriptKey, loadstringUrl)
    if activeScripts[scriptKey] then
        pcall(function() activeScripts[scriptKey]:Destroy() end)
        activeScripts[scriptKey] = nil
        return
    end

    local win = Instance.new("Frame")
    win.Size = UDim2.new(0, 220, 0, 70)
    win.Position = UDim2.new(0.8, 0, 0.1 + (scriptKey == "iy" and 0.1 or 0), 0)
    win.BackgroundColor3 = Color3.fromRGB(10, 14, 22)
    win.BorderSizePixel = 0
    win.Active = true
    win.Draggable = true
    win.Parent = screenGui
    Instance.new("UICorner", win).CornerRadius = UDim.new(0, 8)

    local stroke = Instance.new("UIStroke", win)
    stroke.Color = Color3.fromRGB(255, 0, 127)
    stroke.Thickness = 1.5

    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1, 0, 0, 25)
    topBar.BackgroundColor3 = Color3.fromRGB(18, 24, 36)
    topBar.BorderSizePixel = 0
    topBar.Parent = win
    Instance.new("UICorner", topBar).CornerRadius = UDim.new(0, 8)

    local nameLbl = Instance.new("TextLabel")
    nameLbl.Size = UDim2.new(1, -30, 1, 0)
    nameLbl.Position = UDim2.new(0, 8, 0, 0)
    nameLbl.BackgroundTransparency = 1
    nameLbl.Text = title
    nameLbl.TextColor3 = Color3.fromRGB(0, 242, 254)
    nameLbl.TextSize = 10
    nameLbl.Font = Enum.Font.GothamBold
    nameLbl.TextXAlignment = Enum.TextXAlignment.Left
    nameLbl.Parent = topBar

    local xBtn = Instance.new("TextButton")
    xBtn.Size = UDim2.new(0, 20, 0, 20)
    xBtn.Position = UDim2.new(1, -22, 0, 2)
    xBtn.BackgroundColor3 = Color3.fromRGB(200, 20, 50)
    xBtn.Text = "X"
    xBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    xBtn.TextSize = 11
    xBtn.Font = Enum.Font.GothamBold
    xBtn.BorderSizePixel = 0
    xBtn.Parent = topBar
    Instance.new("UICorner", xBtn).CornerRadius = UDim.new(0, 4)

    local statusLbl = Instance.new("TextLabel")
    statusLbl.Size = UDim2.new(1, -10, 0, 35)
    statusLbl.Position = UDim2.new(0, 5, 0, 30)
    statusLbl.BackgroundTransparency = 1
    statusLbl.Text = "Statut : Actif / Exécuté"
    statusLbl.TextColor3 = Color3.fromRGB(0, 230, 120)
    statusLbl.TextSize = 10
    statusLbl.Font = Enum.Font.Gotham
    statusLbl.Parent = win

    activeScripts[scriptKey] = win

    xBtn.MouseButton1Click:Connect(function()
        pcall(function() win:Destroy() end)
        activeScripts[scriptKey] = nil
    end)

    task.spawn(function()
        pcall(function()
            loadstring(game:HttpGet(loadstringUrl))()
        end)
    end)
end

--------------------------------------------------------------------------------
-- 3. LOGIQUE DES FONCTIONNALITÉS (VOL, ESP, NOCLIP, SPIN, ETC.)
--------------------------------------------------------------------------------
danceBtn.MouseButton1Click:Connect(function()
    dancing = not dancing
    local char = player.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    
    if dancing then
        danceBtn.Text = "💃 Danses Aléatoires : ON"
        danceBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 127)
        if humanoid then
            local randomAnim = defaultDances[math.random(1, #defaultDances)]
            local anim = Instance.new("Animation")
            anim.AnimationId = randomAnim
            currentDanceTrack = humanoid:LoadAnimation(anim)
            currentDanceTrack.Priority = Enum.AnimationPriority.Action
            currentDanceTrack.Looped = true
            currentDanceTrack:Play()
        end
    else
        danceBtn.Text = "💃 Danses Aléatoires : OFF"
        danceBtn.BackgroundColor3 = Color3.fromRGB(80, 15, 60)
        if currentDanceTrack then
            currentDanceTrack:Stop()
            currentDanceTrack = nil
        end
    end
end)

local function refreshPlayerList()
    for _, child in pairs(playerListContainer:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end

    local yOffset = 0
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -6, 0, 36)
            btn.Position = UDim2.new(0, 3, 0, yOffset)
            btn.BackgroundColor3 = Color3.fromRGB(16, 22, 36)
            btn.Text = "        " .. p.DisplayName .. " (@" .. p.Name .. ")"
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextSize = 10
            btn.Font = Enum.Font.GothamSemibold
            btn.TextXAlignment = Enum.TextXAlignment.Left
            btn.BorderSizePixel = 0
            btn.Parent = playerListContainer
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)

            local icon = Instance.new("ImageLabel")
            icon.Size = UDim2.new(0, 30, 0, 30)
            icon.Position = UDim2.new(0, 3, 0, 3)
            icon.BackgroundTransparency = 1
            icon.Parent = btn
            Instance.new("UICorner", icon).CornerRadius = UDim.new(1, 0)

            task.spawn(function()
                local content, isReady = Players:GetUserThumbnailAsync(p.UserId, Enum.ThumbnailType.AvatarBust, Enum.ThumbnailSize.Size150x150)
                if isReady and icon then
                    icon.Image = content
                end
            end)

            btn.MouseButton1Click:Connect(function()
                targetPlayer = p
                infoLabel.Text = "Cible : " .. p.DisplayName .. " (@" .. p.Name .. ")"
                if cameraFollowing and targetPlayer and targetPlayer.Character then
                    local hum = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
                    local hrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 then
                        camera.CameraSubject = hum
                    elseif hrp then
                        camera.CameraSubject = hrp
                    end
                end
            end)

            yOffset = yOffset + 40
        end
    end
end

table.insert(connections, Players.PlayerAdded:Connect(refreshPlayerList))
table.insert(connections, Players.PlayerRemoving:Connect(refreshPlayerList))
refreshPlayerList()

local espHighlights = {}
local espBillboards = {}

local function analyzeRole(p)
    local backpack = p:FindFirstChild("Backpack")
    local char = p.Character
    local hasGun = (backpack and (backpack:FindFirstChild("Gun") or backpack:FindFirstChild("Revolver"))) or (char and (char:FindFirstChild("Gun") or char:FindFirstChild("Revolver")))
    local hasKnife = (backpack and (backpack:FindFirstChild("Knife") or backpack:FindFirstChild("Dagger"))) or (char and (char:FindFirstChild("Knife") or char:FindFirstChild("Dagger")))
    
    if hasKnife then
        return "Tueur", Color3.fromRGB(255, 30, 30)
    elseif hasGun then
        return "Shérif", Color3.fromRGB(255, 220, 0)
    else
        return "Innocent", Color3.fromRGB(0, 230, 100)
    end
end

local function removeEspForPlayer(p)
    if p.Character then
        local hl = p.Character:FindFirstChild("XenoBodyESP")
        if hl then hl:Destroy() end
        local bb = p.Character:FindFirstChild("XenoNameESP")
        if bb then bb:Destroy() end
    end
end

local function applyEspToCharacter(char, p)
    if not espActive or p == player then return end
    removeEspForPlayer(p)
    
    local roleName, roleColor = analyzeRole(p)

    if char and not char:FindFirstChild("XenoBodyESP") then
        local hl = Instance.new("Highlight")
        hl.Name = "XenoBodyESP"
        hl.Adornee = char
        hl.FillColor = roleColor
        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
        hl.FillTransparency = 0.4
        hl.Parent = char
        table.insert(espHighlights, hl)
    end

    if char and char:FindFirstChild("Head") and not char:FindFirstChild("XenoNameESP") then
        local bb = Instance.new("BillboardGui")
        bb.Name = "XenoNameESP"
        bb.Size = UDim2.new(0, 200, 0, 50)
        bb.StudsOffset = Vector3.new(0, 2.8, 0)
        bb.AlwaysOnTop = true
        bb.Adornee = char.Head
        bb.Parent = char

        local txt = Instance.new("TextLabel")
        txt.Name = "EspText"
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.TextSize = 11
        txt.Font = Enum.Font.GothamBold
        txt.TextColor3 = roleColor
        txt.Text = p.DisplayName .. " (@" .. p.Name .. ")\n[" .. roleName .. "]"
        txt.Parent = bb

        table.insert(espBillboards, bb)
    end
end

espBtn.MouseButton1Click:Connect(function()
    espActive = not espActive
    if espActive then
        espBtn.Text = "🤖 ESP Rôles & Noms : ON"
        espBtn.BackgroundColor3 = Color3.fromRGB(0, 242, 254)
        for _, p in pairs(Players:GetPlayers()) do
            if p.Character then applyEspToCharacter(p.Character, p) end
        end
    else
        espBtn.Text = "🤖 ESP Rôles & Noms : OFF"
        espBtn.BackgroundColor3 = Color3.fromRGB(60, 15, 80)
        for _, p in pairs(Players:GetPlayers()) do
            removeEspForPlayer(p)
        end
        espHighlights = {}
        espBillboards = {}
    end
end)

table.insert(connections, RunService.RenderStepped:Connect(function()
    if espActive then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local roleName, roleColor = analyzeRole(p)
                local hl = p.Character:FindFirstChild("XenoBodyESP")
                if hl then hl.FillColor = roleColor end

                local bb = p.Character:FindFirstChild("XenoNameESP")
                if bb and bb:FindFirstChild("EspText") then
                    bb.EspText.TextColor3 = roleColor
                    bb.EspText.Text = p.DisplayName .. " (@" .. p.Name .. ")\n[" .. roleName .. "]"
                elseif not bb then
                    applyEspToCharacter(p.Character, p)
                end
            end
        end
    end
end))

noclipBtn.MouseButton1Click:Connect(function()
    noclipActive = not noclipActive
    if noclipActive then
        noclipBtn.Text = "👻 Noclip : ON"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(0, 230, 120)
    else
        noclipBtn.Text = "👻 Noclip : OFF"
        noclipBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end
end)

table.insert(connections, RunService.Stepped:Connect(function()
    if noclipActive and player.Character then
        for _, part in pairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end))

spinBtn.MouseButton1Click:Connect(function()
    spinActive = not spinActive
    if spinActive then
        spinBtn.Text = "🌀 Spin (Tourner) : ON"
        spinBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 255)
    else
        spinBtn.Text = "🌀 Spin (Tourner) : OFF"
        spinBtn.BackgroundColor3 = Color3.fromRGB(70, 30, 90)
    end
end)

table.insert(connections, RunService.RenderStepped:Connect(function()
    if spinActive and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = player.Character.HumanoidRootPart
        hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(spinSpeed), 0)
    end
end))

flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    local char = player.Character
    if not char then return end
    local hrp, humanoid = char:FindFirstChild("HumanoidRootPart"), char:FindFirstChildOfClass("Humanoid")

    if flying then
        flyBtn.Text = "🛸 Vol (Fly) : ON"
        flyBtn.BackgroundColor3 = Color3.fromRGB(0, 242, 254)
        if humanoid then humanoid.PlatformStand = true end
        if hrp then
            bodyPosition = Instance.new("BodyPosition", hrp)
            bodyPosition.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bodyPosition.Position = hrp.Position
            bodyGyro = Instance.new("BodyGyro", hrp)
            bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bodyGyro.CFrame = hrp.CFrame
        end
    else
        flyBtn.Text = "🛸 Vol (Fly) : OFF"
        flyBtn.BackgroundColor3 = Color3.fromRGB(15, 50, 80)
        if humanoid then humanoid.PlatformStand = false end
        if bodyPosition then bodyPosition:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
    end
end)

table.insert(connections, RunService.RenderStepped:Connect(function()
    if flying and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local moveDir = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.Z) or UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Q) or UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0, 1, 0) end

        if bodyPosition and bodyGyro then
            bodyPosition.Position = bodyPosition.Position + (moveDir * (flySpeed / 20))
            bodyGyro.CFrame = camera.CFrame
        end
    end
end))

followBtn.MouseButton1Click:Connect(function()
    following = not following
    if following then
        followBtn.Text = "🏃 Suivre Cible : ON"
        followBtn.BackgroundColor3 = Color3.fromRGB(255, 153, 0)
    else
        followBtn.Text = "🏃 Suivre Cible : OFF"
        followBtn.BackgroundColor3 = Color3.fromRGB(80, 50, 15)
    end
end)

table.insert(connections, RunService.RenderStepped:Connect(function()
    if following and targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetRoot = targetPlayer.Character.HumanoidRootPart
        local myRoot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if myRoot then
            myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 3)
        end
    end
end))

camBtn.MouseButton1Click:Connect(function()
    cameraFollowing = not cameraFollowing
    if cameraFollowing then
        camBtn.Text = "🎥 Voir Cible (Caméra) : ON"
        camBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 255)
        if targetPlayer and targetPlayer.Character then
            local hum = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
            local hrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 then camera.CameraSubject = hum
            elseif hrp then camera.CameraSubject = hrp end
        end
    else
        camBtn.Text = "🎥 Voir Cible (Caméra) : OFF"
        camBtn.BackgroundColor3 = Color3.fromRGB(50, 20, 80)
        if player.Character then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if hum then camera.CameraSubject = hum end
        end
    end
end)

resetCamBtn.MouseButton1Click:Connect(function()
    cameraFollowing = false
    camBtn.Text = "🎥 Voir Cible (Caméra) : OFF"
    camBtn.BackgroundColor3 = Color3.fromRGB(50, 20, 80)
    if player.Character then
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        if hum then camera.CameraSubject = hum end
    end
end)

table.insert(connections, RunService.RenderStepped:Connect(function()
    if cameraFollowing and targetPlayer then
        local char = targetPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 then
                if camera.CameraSubject ~= hum then camera.CameraSubject = hum end
            elseif hrp then
                if camera.CameraSubject ~= hrp then camera.CameraSubject = hrp end
            end
        end
    end
end))

player.CharacterAdded:Connect(function(newChar)
    task.wait(0.5)
    if not cameraFollowing and newChar then
        local hum = newChar:FindFirstChildOfClass("Humanoid")
        if hum then camera.CameraSubject = hum end
    end
end)

tpBtn.MouseButton1Click:Connect(function()
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local myRoot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if myRoot then
            myRoot.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(2, 0, 2)
        end
    end
end)

flingBtn.MouseButton1Click:Connect(function()
    task.spawn(function()
        pcall(function()
            loadstring(game:HttpGet('https://raw.githubusercontent.com/unrexl/Scripts/refs/heads/main/FlingV2'))()
        end)
    end)
end)

coolkidBtn.MouseButton1Click:Connect(function()
    createScriptControlWindow("C00lcl4n 2.0 (Coolkid)", "coolkid", "https://gist.githubusercontent.com/wateringcancan-eng/209697b404ddee53bac61f2dca1237fc/raw/690217aa6cf7e4d5717b4caa285f2236b1dcde26/C00lcl4n_2_0.lua")
end)

iyBtn.MouseButton1Click:Connect(function()
    createScriptControlWindow("Infinite Yield (IY)", "iy", "https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source")
end)

rejoinBtn.MouseButton1Click:Connect(function()
    pcall(function()
        TeleportService:Teleport(game.PlaceId, player)
    end)
end)

serverHopBtn.MouseButton1Click:Connect(function()
    pcall(function()
        local servers = {}
        local req = HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"))
        for _, s in ipairs(req.data) do
            if type(s) == "table" and s.maxPlayers > s.playing and s.id ~= game.JobId then
                table.insert(servers, s.id)
            end
        end
        if #servers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], player)
        else
            infoLabel.Text = "Aucun autre serveur trouvé."
        end
    end)
end)

table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Q then
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") and mouse.Hit then
            character.HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0, 3, 0))
        end
    end
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        mainFrame.Visible = not mainFrame.Visible
    end
end))
