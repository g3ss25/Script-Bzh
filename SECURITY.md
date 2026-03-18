-- Charger Kavo UI
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()

-- Fenêtre (theme plus stylé)
local Window = Library.CreateLib("🔥 KaKa Hub | OP Script", "Ocean")

-- Onglets
local PlayerTab = Window:NewTab("Player")
local ESPTab = Window:NewTab("ESP")
local VisualTab = Window:NewTab("Visual")
local ScriptTab = Window:NewTab("Scripts")

-- Services
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- Variables
local flying = false
local noclip = false
local flySpeed = 2
local espEnabled = false
local control = {f=0,b=0,l=0,r=0,u=0,d=0}

-- ================= PLAYER =================

local PlayerSection = PlayerTab:NewSection("🚀 Movement")

PlayerSection:NewSlider("Fly Speed", "Changer la vitesse du fly", 10, 1, function(v)
    flySpeed = v
end)

PlayerSection:NewButton("🕊️ Fly", "Activer le fly", function()
    local player = Players.LocalPlayer
    local char = player.Character
    local hrp = char:WaitForChild("HumanoidRootPart")

    flying = true

    local bg = Instance.new("BodyGyro", hrp)
    local bv = Instance.new("BodyVelocity", hrp)

    bg.P = 9e4
    bg.maxTorque = Vector3.new(9e9,9e9,9e9)
    bv.maxForce = Vector3.new(9e9,9e9,9e9)

    RunService.RenderStepped:Connect(function()
        if flying then
            local cam = workspace.CurrentCamera
            bg.cframe = cam.CFrame

            local vel = (cam.CFrame.LookVector * (control.f - control.b) +
                        cam.CFrame.RightVector * (control.r - control.l) +
                        cam.CFrame.UpVector * (control.u - control.d))

            bv.velocity = vel * (flySpeed * 50)
        end
    end)
end)

PlayerSection:NewButton("🛑 UnFly", "Arrêter le fly", function()
    flying = false

    local char = Players.LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        for _,v in pairs(char.HumanoidRootPart:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then
                v:Destroy()
            end
        end
    end
end)

PlayerSection:NewToggle("👻 Noclip", "Traverse les murs", function(state)
    noclip = state
end)

-- ================= CONTROLS =================

UIS.InputBegan:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W then control.f = 1 end
    if input.KeyCode == Enum.KeyCode.S then control.b = 1 end
    if input.KeyCode == Enum.KeyCode.A then control.l = 1 end
    if input.KeyCode == Enum.KeyCode.D then control.r = 1 end
    if input.KeyCode == Enum.KeyCode.Space then control.u = 1 end
    if input.KeyCode == Enum.KeyCode.LeftShift then control.d = 1 end
end)

UIS.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then control.f = 0 end
    if input.KeyCode == Enum.KeyCode.S then control.b = 0 end
    if input.KeyCode == Enum.KeyCode.A then control.l = 0 end
    if input.KeyCode == Enum.KeyCode.D then control.r = 0 end
    if input.KeyCode == Enum.KeyCode.Space then control.u = 0 end
    if input.KeyCode == Enum.KeyCode.LeftShift then control.d = 0 end
end)

-- ================= NOCLIP =================

RunService.Stepped:Connect(function()
    if noclip then
        for _,v in pairs(Players.LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

-- ================= ESP =================

local ESPSection = ESPTab:NewSection("👁️ Player ESP")

ESPSection:NewToggle("ESP Players", "Voir les joueurs à travers les murs", function(state)
    espEnabled = state

    for _,player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer and player.Character then
            if state then
                if not player.Character:FindFirstChild("Highlight") then
                    local hl = Instance.new("Highlight")
                    hl.FillColor = Color3.fromRGB(0,170,255)
                    hl.OutlineColor = Color3.fromRGB(255,255,255)
                    hl.Parent = player.Character
                end
            else
                if player.Character:FindFirstChild("Highlight") then
                    player.Character.Highlight:Destroy()
                end
            end
        end
    end
end)

-- ================= VISUAL =================

local VisualSection = VisualTab:NewSection("🎥 Camera")

VisualSection:NewSlider("FOV", "Augmenter le champ de vision", 120, 70, function(v)
    workspace.CurrentCamera.FieldOfView = v
end)

-- ================= SCRIPTS =================

local ScriptSection = ScriptTab:NewSection("⚡ OP Scripts")

ScriptSection:NewButton("🔥 BZH Insta TP", "Steal automatiquement", function()
    loadstring(game:HttpGet("https://rawscripts.net/raw/Steal-A-Bzh-BZH-INSTA-TP-STEAL-FREE-127827"))()
end)

ScriptSection:NewButton("🐼 Panda Script", "Ouvrir Panda Hub", function()
    loadstring(game:HttpGet("https://vss.pandadevelopment.net/virtual/file/ff6a3157e22c48d2"))()
end)

-- ================= TOGGLE GUI =================

local menuOpen = true

UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.J then
        menuOpen = not menuOpen

        for _,v in pairs(game.CoreGui:GetChildren()) do
            if v:IsA("ScreenGui") and v.Name:lower():find("kavo") then
                v.Enabled = menuOpen
            end
        end
    end
end)
-- ===== TRACERS =====

local tracerEnabled = false
local tracers = {}

ESPSection:NewToggle("Player Tracers", "Tracer des lignes vers les joueurs", function(state)
    tracerEnabled = state

    if not state then
        for _,line in pairs(tracers) do
            if line then
                line:Remove()
            end
        end
        tracers = {}
    end
end)

RunService.RenderStepped:Connect(function()

    if not tracerEnabled then return end

    local cam = workspace.CurrentCamera
    local screen = cam.ViewportSize

    for _,player in pairs(Players:GetPlayers()) do

        if player ~= Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then

            local hrp = player.Character.HumanoidRootPart
            local pos,visible = cam:WorldToViewportPoint(hrp.Position)

            if not tracers[player] then
                local line = Drawing.new("Line")
                line.Thickness = 2
                line.Transparency = 1
                line.Color = Color3.fromRGB(0,170,255)
                tracers[player] = line
            end

            local line = tracers[player]

            if visible then
                line.Visible = true
                line.From = Vector2.new(screen.X/2, screen.Y)
                line.To = Vector2.new(pos.X, pos.Y)
            else
                line.Visible = false
            end

        end
    end

end)

Players.PlayerRemoving:Connect(function(player)
    if tracers[player] then
        tracers[player]:Remove()
        tracers[player] = nil
    end
end)
-- ===== TRACERS RAINBOW =====

local tracerEnabled = false
local tracers = {}
local hue = 0

ESPSection:NewToggle("Rainbow Tracers", "Tracer des lignes rainbow vers les joueurs", function(state)
    tracerEnabled = state

    if not state then
        for _,line in pairs(tracers) do
            if line then
                line:Remove()
            end
        end
        tracers = {}
    end
end)

RunService.RenderStepped:Connect(function()

    if not tracerEnabled then return end

    local cam = workspace.CurrentCamera
    local screen = cam.ViewportSize

    -- rainbow color
    hue = hue + 0.01
    if hue > 1 then
        hue = 0
    end

    local rainbow = Color3.fromHSV(hue,1,1)

    for _,player in pairs(Players:GetPlayers()) do

        if player ~= Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then

            local hrp = player.Character.HumanoidRootPart
            local pos,visible = cam:WorldToViewportPoint(hrp.Position)

            if not tracers[player] then
                local line = Drawing.new("Line")
                line.Thickness = 2
                line.Transparency = 1
                tracers[player] = line
            end

            local line = tracers[player]
            line.Color = rainbow

            if visible then
                line.Visible = true
                line.From = Vector2.new(screen.X/2, screen.Y)
                line.To = Vector2.new(pos.X, pos.Y)
            else
                line.Visible = false
            end

        end
    end

end)

Players.PlayerRemoving:Connect(function(player)
    if tracers[player] then
        tracers[player]:Remove()
        tracers[player] = nil
    end
end)
-- ================= SPEED HACK LOCAL =================

local speedEnabled = false
local speedValue = 50 -- tu peux changer la vitesse max

VisualSection:NewSlider("Speed", "Vitesse du joueur", 200, 16, function(v)
    speedValue = v
end)

VisualSection:NewToggle("Speed Hack", "Activer le speed hack", function(state)
    speedEnabled = state

    -- supprimer le BV si on désactive
    if not state then
        local char = Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp = char.HumanoidRootPart
            if hrp:FindFirstChild("SpeedBV") then
                hrp.SpeedBV:Destroy()
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if speedEnabled then
        local char = Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp = char.HumanoidRootPart
            local bv = hrp:FindFirstChild("SpeedBV")
            if not bv then
                bv = Instance.new("BodyVelocity")
                bv.Name = "SpeedBV"
                bv.MaxForce = Vector3.new(1e5,0,1e5)
                bv.Velocity = Vector3.new(0,0,0)
                bv.Parent = hrp
            end

            local cam = workspace.CurrentCamera
            local vel = (cam.CFrame.LookVector * (control.f - control.b) +
                         cam.CFrame.RightVector * (control.r - control.l))
            bv.Velocity = vel * speedValue
        end
    end
end)
-- ===== BRAINROT ESP =====

local brainrotESP = false

ESPSection:NewToggle("Brainrot ESP", "Voir tous les brainrots", function(state)
    brainrotESP = state

    if not state then
        for _,v in pairs(workspace:GetDescendants()) do
            if v:IsA("Highlight") and v.Name == "BrainrotESP" then
                v:Destroy()
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()

    if not brainrotESP then return end

    for _,obj in pairs(workspace:GetChildren()) do

        if obj:IsA("Model") and obj:FindFirstChild("HumanoidRootPart") and not Players:GetPlayerFromCharacter(obj) then

            if not obj:FindFirstChild("BrainrotESP") then

                local hl = Instance.new("Highlight")
                hl.Name = "BrainrotESP"
                hl.FillColor = Color3.fromRGB(255,0,200)
                hl.OutlineColor = Color3.fromRGB(255,255,255)
                hl.FillTransparency = 0.5
                hl.Parent = obj

                local billboard = Instance.new("BillboardGui")
                billboard.Name = "BrainrotESP"
                billboard.Size = UDim2.new(0,200,0,50)
                billboard.AlwaysOnTop = true
                billboard.Parent = obj.HumanoidRootPart

                local text = Instance.new("TextLabel")
                text.Size = UDim2.new(1,0,1,0)
                text.BackgroundTransparency = 1
                text.TextColor3 = Color3.fromRGB(255,0,200)
                text.TextStrokeTransparency = 0
                text.Font = Enum.Font.GothamBold
                text.TextScaled = true
                text.Text = obj.Name
                text.Parent = billboard

            end
        end
    end

end)
-- ================= AUTO SPEED (BRAINROT SAFE) =================

local speedEnabled = false
local speedValue = 60

VisualSection:NewSlider("Speed", "Vitesse joueur", 120, 16, function(v)
    speedValue = v
end)

VisualSection:NewToggle("Auto Speed", "Speed seulement sans brainrot", function(state)
    speedEnabled = state
end)

RunService.RenderStepped:Connect(function()

    if not speedEnabled then return end

    local char = Players.LocalPlayer.Character
    if not char then return end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    -- détecte si tu portes un objet
    local holdingBrainrot = false

    for _,v in pairs(char:GetChildren()) do
        if v:IsA("Tool") then
            holdingBrainrot = true
        end
    end

    if holdingBrainrot then
        humanoid.WalkSpeed = 16
    else
        humanoid.WalkSpeed = speedValue
    end
end)
ScriptSection:NewButton("💀 SABM Script", "Lancer le script SABM", function()
    loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-SABM-READ-DESC-135961"))()
end)
