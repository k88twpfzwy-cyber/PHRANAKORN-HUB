local _version = "1.6.66"
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/download/" .. _version .. "/main.lua"))()
WindUI:AddTheme({
Name = "Moon Space",
Accent = Color3.fromHex("#000000"),
Background = Color3.fromHex("#990000"),
Text = Color3.fromHex("#FFFFFF"),
Placeholder = Color3.fromHex("#FFFF00"),
Button = Color3.fromHex("#FFFF00"),
Icon = Color3.fromHex("#FFFF66"),
Hover = Color3.fromHex("#ffffff"),
WindowBackground = Color3.fromHex("#000000"),
WindowShadow = Color3.fromHex("#FFFFFF"),
})
local Window = WindUI:CreateWindow({
Title = " พระนคร ",
Icon ="mail",
Author = " สายเลือดอาชีวะ",
Folder = "So hub X ",
Size = UDim2.fromOffset(400, 200),
MinSize = Vector2.new(560, 350),
MaxSize = Vector2.new(850, 560),
Transparent = true,
Theme = "Moon Space",
Resizable = true,
SideBarWidth = 200,
User = {
Enabled = true,
Anonymous = false,
Callback = function()
print("clicked")
end,
},
})

Window:Tag({
Title = "v1.0",
Icon = "",
Color = Color3.fromHex("#FFFF00"),
Radius = 13,

})
Window:Section({
    Title = "Tab Section example", -- title. required
    Icon = "calendar", -- icon. optional
    Opened = true, -- whether the section is opened. optional
})
local Tab = Window:Tab({
    Title = "aimbot",
Icon = "target",
})
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local Stats = game:GetService("Stats")
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

local Settings = {
    SilentAim = false,
    FOVSize = 220,
    MaxDistance = 2000,
    AimPart = "Head",
    TeamCheck = true,
    WallCheck = true,
    ShowFOV = true,
    ShowTracer = true,
    CustomBulletSpeed = 2000,
    NoSpread = true,
    NoRecoil = true
}

local CurrentTarget = nil
local hooked = false

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ICEHUB_SilentAim_Overlay"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local FOVCircle = Instance.new("Frame")
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVCircle.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Visible = Settings.ShowFOV
FOVCircle.Parent = ScreenGui
Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)
local FOVStroke = Instance.new("UIStroke", FOVCircle)
FOVStroke.Color = Color3.fromRGB(255, 0, 0)
FOVStroke.Thickness = 1
FOVStroke.Transparency = 0.2

local Tracer = Instance.new("Frame")
Tracer.AnchorPoint = Vector2.new(0.5, 0.5)
Tracer.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
Tracer.BorderSizePixel = 0
Tracer.Visible = false
Tracer.Parent = ScreenGui

local function GetPing()
    local ping = 0.03
    pcall(function()
        if LocalPlayer.GetNetworkPing then
            ping = LocalPlayer:GetNetworkPing()
        else
            ping = Stats.Network.ServerStatsItem["Data Ping"]:GetValue() / 1000
        end
    end)
    return math.clamp(ping, 0, 0.5)
end

local function HasWallBetween(origin, targetPos, character)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    local result = Workspace:Raycast(origin, targetPos - origin, params)
    if result then
        return not result.Instance:IsDescendantOf(character)
    end
    return false
end

local function PredictPosition(origin, targetPart, bulletSpeed)
    if not targetPart or not targetPart.Parent then return nil end
    local pos = targetPart.Position
    local root = targetPart.Parent:FindFirstChild("HumanoidRootPart")
        or targetPart.Parent:FindFirstChild("Torso")
        or targetPart.Parent:FindFirstChild("UpperTorso")
        or targetPart
    local velocity = Vector3.zero
    pcall(function()
        velocity = root.AssemblyLinearVelocity or root.Velocity or Vector3.zero
    end)
    pos = pos + velocity * GetPing()
    local speed = math.max(bulletSpeed or Settings.CustomBulletSpeed, 300)
    local toTarget = pos - origin
    local a = velocity:Dot(velocity) - (speed * speed)
    local b = 2 * toTarget:Dot(velocity)
    local c = toTarget:Dot(toTarget)
    local disc = (b * b) - (4 * a * c)
    local flyTime = toTarget.Magnitude / speed
    if disc >= 0 and math.abs(a) > 0.0001 then
        local t1 = (-b - math.sqrt(disc)) / (2 * a)
        local t2 = (-b + math.sqrt(disc)) / (2 * a)
        if t1 > 0 and t2 > 0 then flyTime = math.min(t1, t2)
        elseif t1 > 0 then flyTime = t1
        elseif t2 > 0 then flyTime = t2 end
    end
    pos = pos + velocity * flyTime
    pos = pos + Vector3.new(0, 0.5 * Workspace.Gravity * (flyTime * flyTime), 0)
    return pos
end

local function GetClosestTarget()
    if not Settings.SilentAim then
        CurrentTarget = nil
        return nil
    end
    local closestPart, closestPlayer = nil, nil
    local shortest = Settings.FOVSize + 1
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local myRoot = LocalPlayer.Character and (
        LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        or LocalPlayer.Character:FindFirstChild("UpperTorso")
        or LocalPlayer.Character:FindFirstChild("Torso")
    )
    if not myRoot then
        CurrentTarget = nil
        return nil
    end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            local hum = char:FindFirstChildOfClass("Humanoid")
            local part = char:FindFirstChild(Settings.AimPart) or char:FindFirstChild("Head")
            local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
            if hum and hum.Health > 0 and part and root then
                if Settings.TeamCheck and player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
                    continue
                end
                local dist3D = (myRoot.Position - root.Position).Magnitude
                if dist3D <= Settings.MaxDistance then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen and screenPos.Z > 0 then
                        local dist2D = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                        if dist2D <= Settings.FOVSize then
                            local blocked = Settings.WallCheck and HasWallBetween(Camera.CFrame.Position, part.Position, char)
                            if not blocked then
                                if dist2D < shortest then
                                    shortest = dist2D
                                    closestPart = part
                                    closestPlayer = player
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    CurrentTarget = closestPlayer
    return closestPart
end

local function ApplyHook()
    if hooked then return end

    local ok, handler = pcall(function()
        return require(ReplicatedStorage:WaitForChild("Modules"):WaitForChild("ProjectileHandler"))
    end)
    if not ok or not handler or not handler.SimulateProjectile then
        return
    end

    local old = handler.SimulateProjectile
    handler.SimulateProjectile = function(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15)
        if Settings.SilentAim then
            local targetPart = GetClosestTarget()
            if targetPart then
                pcall(function()
                    if type(a2) == "table" then
                        if Settings.NoSpread then
                            a2.Spread = 0
                            a2.Inaccuracy = 0
                            a2.BulletDeviation = 0
                        end
                        if Settings.NoRecoil then
                            a2.Recoil = 0
                            a2.RecoilUp = 0
                            a2.CameraKick = 0
                        end
                    end
                end)

                local origin
                if a8 and a8.WorldPosition then
                    origin = a8.WorldPosition
                elseif a4 then
                    local gfp = a4:FindFirstChild("GunFirePoint") or a4:FindFirstChild("FirePoint")
                    if gfp then origin = gfp.WorldPosition end
                end
                if not origin and LocalPlayer.Character then
                    local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then origin = hrp.Position end
                end

                if origin then
                    local speed = Settings.CustomBulletSpeed
                    if type(a2) == "table" then
                        local s = a2.BulletSpeed or a2.Speed or a2.MuzzleVelocity
                        if type(s) == "number" and s > 50 then speed = s end
                    end
                    local predicted = PredictPosition(origin, targetPart, speed)
                    if predicted then
                        local dir = predicted - origin
                        if dir.Magnitude > 0.001 then
                            dir = dir.Unit
                            if typeof(a6) == "table" then
                                for i = 1, #a6 do a6[i] = dir end
                            elseif typeof(a6) == "Vector3" then
                                a6 = dir
                            end
                            if typeof(a7) == "table" then
                                for i = 1, #a7 do a7[i] = dir end
                            elseif typeof(a7) == "Vector3" then
                                a7 = dir
                            end
                        end
                    end
                end
            end
        end
        return old(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15)
    end
    hooked = true
end

RunService.RenderStepped:Connect(function()
    FOVCircle.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
    FOVCircle.Visible = Settings.ShowFOV and Settings.SilentAim

    GetClosestTarget()

    if Settings.SilentAim and Settings.ShowTracer then
        if CurrentTarget and CurrentTarget.Character then
            local head = CurrentTarget.Character:FindFirstChild("Head")
            if head then
                local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen and pos.Z > 0 then
                    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    local t2 = Vector2.new(pos.X, pos.Y)
                    local dist = (t2 - center).Magnitude
                    local angle = math.deg(math.atan2(t2.Y - center.Y, t2.X - center.X))
                    
                    Tracer.Size = UDim2.new(0, dist, 0, 1)
                    Tracer.Position = UDim2.new(0, (center.X + t2.X) / 2, 0, (center.Y + t2.Y) / 2)
                    Tracer.Rotation = angle
                    Tracer.Visible = true
                else
                    Tracer.Visible = false
                end
            else
                Tracer.Visible = false
            end
        else
            Tracer.Visible = false
        end
    else
        Tracer.Visible = false
    end
end)

local AimbotToggle = Tab:Toggle({
    Title = "Enable Silent Aim",
    Default = Settings.SilentAim,
    Callback = function(state)
        Settings.SilentAim = state
        if state then
            ApplyHook()
        end
    end
})

local FOVToggle = Tab:Toggle({
    Title = "Show FOV Circle",
    Default = Settings.ShowFOV,
    Callback = function(state)
        Settings.ShowFOV = state
    end
})

local TracerToggle = Tab:Toggle({
    Title = "Show Tracer",
    Default = Settings.ShowTracer,
    Callback = function(state)
        Settings.ShowTracer = state
    end
})

local FOVSlider = Tab:Slider({
    Title = "FOV Size",
    Value = {
        Min = 50,
        Max = 500,
        Default = Settings.FOVSize
    },
    Callback = function(value)
        Settings.FOVSize = value
    end
})
local Tab = Window:Tab({
    Title = "Esp",
    Icon = "view", -- optional
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local ESPEnabled = false
local ESPObjects = {}

local function CreateESP(player)
    if player == Players.LocalPlayer then return end

    local lines = {}

    for i = 1, 8 do
        local line = Drawing.new("Line")
        line.Thickness = 2
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Transparency = 1
        line.Visible = false
        lines[i] = line
    end

    ESPObjects[player] = lines
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, line in ipairs(ESPObjects[player]) do
            line:Remove()
        end
        ESPObjects[player] = nil
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    CreateESP(player)
end

Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(RemoveESP)

local function HideESP()
    for _, lines in pairs(ESPObjects) do
        for _, line in ipairs(lines) do
            line.Visible = false
        end
    end
end

RunService.RenderStepped:Connect(function()
    if not ESPEnabled then
        HideESP()
        return
    end

    local Camera = workspace.CurrentCamera

    for player, lines in pairs(ESPObjects) do
        local character = player.Character
        local root = character and character:FindFirstChild("HumanoidRootPart")
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")

        if root and humanoid and humanoid.Health > 0 then
            local cf = character:GetBoundingBox()
            local size = character:GetExtentsSize()

            local top = cf.Position + Vector3.new(0, size.Y / 2, 0)
            local bottom = cf.Position - Vector3.new(0, size.Y / 2, 0)

            local topPos, topVisible = Camera:WorldToViewportPoint(top)
            local bottomPos, bottomVisible = Camera:WorldToViewportPoint(bottom)

            if topVisible and bottomVisible and topPos.Z > 0 and bottomPos.Z > 0 then
                local height = math.abs(bottomPos.Y - topPos.Y)
                local width = height * 0.55
                local x = (topPos.X + bottomPos.X) / 2
                local yTop = topPos.Y
                local yBottom = bottomPos.Y

                local cornerX = width / 4
                local cornerY = height / 4

                -- มุมซ้ายบน
                lines[1].From = Vector2.new(x - width / 2, yTop)
                lines[1].To = Vector2.new(x - width / 2 + cornerX, yTop)

                lines[2].From = Vector2.new(x - width / 2, yTop)
                lines[2].To = Vector2.new(x - width / 2, yTop + cornerY)

                -- มุมขวาบน
                lines[3].From = Vector2.new(x + width / 2, yTop)
                lines[3].To = Vector2.new(x + width / 2 - cornerX, yTop)

                lines[4].From = Vector2.new(x + width / 2, yTop)
                lines[4].To = Vector2.new(x + width / 2, yTop + cornerY)

                -- มุมซ้ายล่าง
                lines[5].From = Vector2.new(x - width / 2, yBottom)
                lines[5].To = Vector2.new(x - width / 2 + cornerX, yBottom)

                lines[6].From = Vector2.new(x - width / 2, yBottom)
                lines[6].To = Vector2.new(x - width / 2, yBottom - cornerY)

                -- มุมขวาล่าง
                lines[7].From = Vector2.new(x + width / 2, yBottom)
                lines[7].To = Vector2.new(x + width / 2 - cornerX, yBottom)

                lines[8].From = Vector2.new(x + width / 2, yBottom)
                lines[8].To = Vector2.new(x + width / 2, yBottom - cornerY)

                for _, line in ipairs(lines) do
                    line.Visible = true
                end
            else
                for _, line in ipairs(lines) do
                    line.Visible = false
                end
            end
        else
            for _, line in ipairs(lines) do
                line.Visible = false
            end
        end
    end
end)

local Toggle = Tab:Toggle({
    Title = "ESP",
    Callback = function(state)
        ESPEnabled = state
    end
})
Window:EditOpenButton({
    Title = "พระนคร",
    Icon = "monitor",
    CornerRadius = UDim.new(0,30),
    StrokeThickness = 2,
    Color = ColorSequence.new( -- gradient
        Color3.fromHex("FFFF66"), 
        Color3.fromHex("ffffff")
    ),
    OnlyMobile = false,
    Enabled = true,
    Draggable = true,
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

local ESPEnabled = false
local ESPBoxColor = Color3.fromRGB(255, 0, 0)
local ESPObjects = {}
local ESPConnection

local function RemoveESP(Player)
    local Objects = ESPObjects[Player]
    if not Objects then
        return
    end

    for _, Object in pairs(Objects) do
        Object:Remove()
    end

    ESPObjects[Player] = nil
end

local function CreateESP(Player)
    if Player == LocalPlayer or ESPObjects[Player] then
        return
    end

    local Box = Drawing.new("Square")
    Box.Color = ESPBoxColor
    Box.Thickness = 1.5
    Box.Filled = false
    Box.Visible = false

    local HealthBG = Drawing.new("Square")
    HealthBG.Color = Color3.fromRGB(25, 25, 25)
    HealthBG.Thickness = 0
    HealthBG.Filled = true
    HealthBG.Visible = false

    local HealthBar = Drawing.new("Square")
    HealthBar.Color = Color3.fromRGB(0, 255, 0)
    HealthBar.Thickness = 0
    HealthBar.Filled = true
    HealthBar.Visible = false

    ESPObjects[Player] = {
        Box = Box,
        HealthBG = HealthBG,
        HealthBar = HealthBar
    }
end

local function HideESP(Objects)
    Objects.Box.Visible = false
    Objects.HealthBG.Visible = false
    Objects.HealthBar.Visible = false
end

local function UpdateESP()
    local Camera = workspace.CurrentCamera
    if not Camera then
        return
    end

    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer then
            if not ESPObjects[Player] then
                CreateESP(Player)
            end

            local Objects = ESPObjects[Player]
            local Character = Player.Character
            local Humanoid = Character and Character:FindFirstChildOfClass("Humanoid")
            local Head = Character and Character:FindFirstChild("Head")
            local Root = Character and Character:FindFirstChild("HumanoidRootPart")

            if ESPEnabled and Humanoid and Head and Root and Humanoid.Health > 0 then
                local Top, TopVisible = Camera:WorldToViewportPoint(
                    Head.Position + Vector3.new(0, 0.5, 0)
                )

                local Bottom, BottomVisible = Camera:WorldToViewportPoint(
                    Root.Position - Vector3.new(0, 3, 0)
                )

                if TopVisible and BottomVisible then
                    local Height = math.abs(Bottom.Y - Top.Y)
                    local Width = Height * 0.55
                    local X = Top.X - Width / 2
                    local Y = Top.Y

                    Objects.Box.Position = Vector2.new(X, Y)
                    Objects.Box.Size = Vector2.new(Width, Height)
                    Objects.Box.Color = ESPBoxColor
                    Objects.Box.Visible = true

                    local Percent = math.clamp(
                        Humanoid.Health / math.max(Humanoid.MaxHealth, 1),
                        0,
                        1
                    )

                    Objects.HealthBG.Position = Vector2.new(X - 6, Y)
                    Objects.HealthBG.Size = Vector2.new(4, Height)
                    Objects.HealthBG.Visible = true

                    Objects.HealthBar.Position = Vector2.new(
                        X - 6,
                        Y + Height * (1 - Percent)
                    )

                    Objects.HealthBar.Size = Vector2.new(
                        4,
                        Height * Percent
                    )

                    Objects.HealthBar.Color = Color3.fromRGB(
                        255 * (1 - Percent),
                        255 * Percent,
                        0
                    )

                    Objects.HealthBar.Visible = true
                else
                    HideESP(Objects)
                end
            else
                HideESP(Objects)
            end
        end
    end
end

local Toggle = Tab:Toggle({
    Title = "ESP Box + Health",
    Default = false,
    Callback = function(state)
        ESPEnabled = state

        if state then
            for _, Player in ipairs(Players:GetPlayers()) do
                CreateESP(Player)
            end

            if not ESPConnection then
                ESPConnection = RunService.RenderStepped:Connect(UpdateESP)
            end
        else
            if ESPConnection then
                ESPConnection:Disconnect()
                ESPConnection = nil
            end

            for Player in pairs(ESPObjects) do
                RemoveESP(Player)
            end
        end
    end
})

local Colorpicker = Tab:Colorpicker({
    Title = "ESP Box Color",
    Default = ESPBoxColor,
    Callback = function(color)
        ESPBoxColor = color

        for _, Objects in pairs(ESPObjects) do
            Objects.Box.Color = color
        end
    end
})

Players.PlayerRemoving:Connect(function(Player)
    RemoveESP(Player)
end)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Enabled = false
local SkeletonColor = Color3.fromRGB(255, 0, 0)
local PlayerLines = {}
local RenderConnection

local BonesR15 = {
    {"Head", "UpperTorso"},
    {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"},
    {"LeftUpperArm", "LeftLowerArm"},
    {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"},
    {"RightUpperArm", "RightLowerArm"},
    {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"},
    {"LeftUpperLeg", "LeftLowerLeg"},
    {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"},
    {"RightUpperLeg", "RightLowerLeg"},
    {"RightLowerLeg", "RightFoot"}
}

local BonesR6 = {
    {"Head", "Torso"},
    {"Torso", "Left Arm"},
    {"Torso", "Right Arm"},
    {"Torso", "Left Leg"},
    {"Torso", "Right Leg"}
}

local function GetBones(character)
    if character:FindFirstChild("UpperTorso") then
        return BonesR15
    end
    return BonesR6
end

local function RemoveLines(player)
    local lines = PlayerLines[player]
    if not lines then return end

    for _, line in ipairs(lines) do
        pcall(function()
            line:Remove()
        end)
    end

    PlayerLines[player] = nil
end

local function CreateLines(player)
    if player == LocalPlayer or PlayerLines[player] then
        return
    end

    local lines = {}
    local success = pcall(function()
        for i = 1, 14 do
            local line = Drawing.new("Line")
            line.Color = SkeletonColor
            line.Thickness = 2
            line.Transparency = 1
            line.Visible = false
            lines[i] = line
        end
    end)

    if success then
        PlayerLines[player] = lines
    else
        for _, line in ipairs(lines) do
            pcall(function()
                line:Remove()
            end)
        end
        warn("Skeleton ESP ต้องใช้ Drawing API")
    end
end

local function UpdateESP()
    local camera = workspace.CurrentCamera
    if not camera then return end

    for player, lines in pairs(PlayerLines) do
        local character = player.Character
        local bones = character and GetBones(character)

        for i, line in ipairs(lines) do
            line.Color = SkeletonColor

            local bone = bones and bones[i]
            local partA = bone and character:FindFirstChild(bone[1])
            local partB = bone and character:FindFirstChild(bone[2])

            if Enabled and partA and partB then
                local posA, visibleA =
                    camera:WorldToViewportPoint(partA.Position)
                local posB, visibleB =
                    camera:WorldToViewportPoint(partB.Position)

                line.From = Vector2.new(posA.X, posA.Y)
                line.To = Vector2.new(posB.X, posB.Y)
                line.Visible = visibleA and visibleB
            else
                line.Visible = false
            end
        end
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    CreateLines(player)
end

Players.PlayerAdded:Connect(CreateLines)
Players.PlayerRemoving:Connect(RemoveLines)

local Toggle = Tab:Toggle({
    Title = "Skeleton ESP",
    Callback = function(state)
        Enabled = state

        if state then
            if not RenderConnection then
                RenderConnection =
                    RunService.RenderStepped:Connect(UpdateESP)
            end
        else
            if RenderConnection then
                RenderConnection:Disconnect()
                RenderConnection = nil
            end

            for _, lines in pairs(PlayerLines) do
                for _, line in ipairs(lines) do
                    line.Visible = false
                end
            end
        end
    end
})

local Colorpicker = Tab:Colorpicker({
    Title = "Skeleton ESP Color",
    Callback = function(color)
        SkeletonColor = color

        for _, lines in pairs(PlayerLines) do
            for _, line in ipairs(lines) do
                line.Color = color
            end
        end
    end
})
