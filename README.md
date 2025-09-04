-- ===================== Sekai Hub | Full Universal Script =====================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "🔥Sekai",
    LoadingTitle = "Sekai Hub",
    LoadingSubtitle = "by Sekai",
    ShowText = "Sekai",
    Theme = "Default",
    ToggleUIKeybind = "K"
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ================= Notification Helper =================
local function notify(title, text, duration)
    Rayfield:Notify({Title=title, Content=text, Duration=duration or 3})
end

-- ================= Player Features =================
local MainTab = Window:CreateTab("🏡 Main", nil)
MainTab:CreateSection("Player Features")

-- Infinite Jump
local infiniteJump = false
local jumpConnection
local function setupInfiniteJump()
    if jumpConnection then jumpConnection:Disconnect() end
    jumpConnection = UIS.JumpRequest:Connect(function()
        local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if infiniteJump and humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
    end)
end
MainTab:CreateButton({
    Name = "🌀Infinite Jump",
    Callback = function()
        infiniteJump = not infiniteJump
        setupInfiniteJump()
        notify("Infinite Jump", infiniteJump and "ON" or "OFF")
    end
})

-- WalkSpeed
local walkSpeed = 16
local function updateWalkSpeed()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then humanoid.WalkSpeed = walkSpeed end
end
MainTab:CreateSlider({
    Name = "⚡Speed",
    Range = {16,300},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 16,
    Callback = function(value)
        walkSpeed = value
        updateWalkSpeed()
    end
})

-- JumpPower
local jumpPower = 50
local function updateJumpPower()
    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then humanoid.JumpPower = jumpPower end
end
MainTab:CreateButton({
    Name = "🚀JumpPower",
    Callback = function()
        jumpPower = (jumpPower==50) and 130 or 50
        updateJumpPower()
        notify("JumpPower", jumpPower==130 and "130 ON" or "50 OFF")
    end
})

-- Noclip
local noclip = false
local noclipConnection
MainTab:CreateButton({
    Name = "🕳Noclip",
    Callback = function()
        noclip = not noclip
        if noclip then
            noclipConnection = RunService.Stepped:Connect(function()
                local char = player.Character
                if char then
                    for _, part in pairs(char:GetChildren()) do
                        if part:IsA("BasePart") then part.CanCollide=false end
                    end
                end
            end)
        else
            if noclipConnection then noclipConnection:Disconnect() noclipConnection=nil end
        end
        notify("Noclip", noclip and "ON" or "OFF")
    end
})

-- Fly
local flying = false
local flySpeed = 80
local hoverHeight = 2
local flyConn
local function getHRP() return player.Character and player.Character:FindFirstChild("HumanoidRootPart") end
local function getHumanoid() return player.Character and player.Character:FindFirstChildOfClass("Humanoid") end
local function startFly()
    if flying then return end
    flying = true
    flyConn = RunService.Heartbeat:Connect(function()
        local hrp = getHRP()
        local hum = getHumanoid()
        if hrp and hum then
            hum.PlatformStand=true
            local dir = hum.MoveDirection
            hrp.Velocity = Vector3.new(dir.X,0,dir.Z)*flySpeed + Vector3.new(0,hoverHeight,0)
        end
    end)
    notify("Fly","ON")
end
local function stopFly()
    flying=false
    if flyConn then flyConn:Disconnect() flyConn=nil end
    local hrp = getHRP()
    local hum = getHumanoid()
    if hrp then hrp.Velocity=Vector3.zero end
    if hum then hum.PlatformStand=false end
    notify("Fly","OFF")
end
MainTab:CreateButton({Name="🕊Fly", Callback=function() if flying then stopFly() else startFly() end end})

-- ================= ESP =================
local VisualsTab = Window:CreateTab("👁 Visuals", nil)
VisualsTab:CreateSection("ESP Features")

local ESP = {Enabled=false, Boxes={}, Tracers={}, Names={}, Health={}, HealthOutline={}, Connections={}}

local function safeNewDrawing(className)
    local success, result = pcall(Drawing.new, className)
    return success and result or nil
end

local function CreateESP(plr)
    if ESP.Boxes[plr] then return end
    local char = plr.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not hrp or not head then return end

    local box = safeNewDrawing("Square")
    local tracer = safeNewDrawing("Line")
    local name = safeNewDrawing("Text")
    local healthBar = safeNewDrawing("Square")
    local healthOutline = safeNewDrawing("Square")
    if not (box and tracer and name and healthBar and healthOutline) then return end

    -- Box
    box.Visible=false box.Color=Color3.new(1,0,0) box.Thickness=2 box.Filled=false box.Transparency=0.75
    -- Tracer
    tracer.Visible=false tracer.Color=Color3.new(0,1,0) tracer.Thickness=1 tracer.Transparency=0.75
    -- Name
    name.Visible=false name.Color=Color3.new(1,1,1) name.Size=16 name.Center=true name.Outline=true name.Font=2
    -- Health
    healthOutline.Visible=false healthOutline.Filled=true healthOutline.Color=Color3.new(0,0,0)
    healthBar.Visible=false healthBar.Filled=true healthBar.Color=Color3.new(0,1,0)

    ESP.Boxes[plr]=box
    ESP.Tracers[plr]=tracer
    ESP.Names[plr]=name
    ESP.Health[plr]=healthBar
    ESP.HealthOutline[plr]=healthOutline
end

local function RemoveESP(plr)
    if ESP.Boxes[plr] then ESP.Boxes[plr]:Remove() ESP.Boxes[plr]=nil end
    if ESP.Tracers[plr] then ESP.Tracers[plr]:Remove() ESP.Tracers[plr]=nil end
    if ESP.Names[plr] then ESP.Names[plr]:Remove() ESP.Names[plr]=nil end
    if ESP.Health[plr] then ESP.Health[plr]:Remove() ESP.Health[plr]=nil end
    if ESP.HealthOutline[plr] then ESP.HealthOutline[plr]:Remove() ESP.HealthOutline[plr]=nil end
end

local function UpdateESP()
    if not ESP.Enabled then return end
    for plr, box in pairs(ESP.Boxes) do
        local char = plr.Character
        if char and plr~=player then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local head = char:FindFirstChild("Head")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hrp and head and hum and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local rootPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
                local headPos, headOnScreen = camera:WorldToViewportPoint(head.Position)
                if onScreen and headOnScreen and rootPos.Z>0 then
                    local height = math.clamp(2000/rootPos.Z,40,350)
                    local width = height/2
                    local x, y = rootPos.X, rootPos.Y

                    -- Box
                    box.Visible=true
                    box.Size=Vector2.new(width, height)
                    box.Position=Vector2.new(x-width/2, y-height/2)

                    -- Tracer
                    local screenSize = camera.ViewportSize
                    ESP.Tracers[plr].Visible=true
                    ESP.Tracers[plr].From=Vector2.new(screenSize.X/2, screenSize.Y)
                    ESP.Tracers[plr].To=Vector2.new(x,y+height/2)

                    -- Name
                    local dist=(hrp.Position-player.Character.HumanoidRootPart.Position).Magnitude
                    ESP.Names[plr].Visible=true
                    ESP.Names[plr].Position=Vector2.new(x, y-height/2-16)
                    ESP.Names[plr].Text=plr.Name.." ["..math.floor(dist).."m]"

                    -- Health Bar
                    local maxHP = hum.MaxHealth>0 and hum.MaxHealth or 100
                    local hp = math.clamp(hum.Health,0,maxHP)
                    local ratio = hp/maxHP
                    local barHeight = height
                    local barWidth = 5
                    local barX = (x-width/2) - (barWidth+4)
                    local barY = (y-height/2)

                    ESP.HealthOutline[plr].Visible=true
                    ESP.HealthOutline[plr].Size=Vector2.new(barWidth+2, barHeight+2)
                    ESP.HealthOutline[plr].Position=Vector2.new(barX-1, barY-1)

                    ESP.Health[plr].Visible=true
                    ESP.Health[plr].Size=Vector2.new(barWidth, barHeight*ratio)
                    ESP.Health[plr].Position=Vector2.new(barX, barY+(barHeight*(1-ratio)))
                    ESP.Health[plr].Color=Color3.fromRGB(255*(1-ratio),255*ratio,0)
                else
                    box.Visible=false ESP.Tracers[plr].Visible=false ESP.Names[plr].Visible=false
                    ESP.Health[plr].Visible=false ESP.HealthOutline[plr].Visible=false
                end
            else
                box.Visible=false ESP.Tracers[plr].Visible=false ESP.Names[plr].Visible=false
                ESP.Health[plr].Visible=false ESP.HealthOutline[plr].Visible=false
            end
        else
            box.Visible=false ESP.Tracers[plr].Visible=false ESP.Names[plr].Visible=false
            ESP.Health[plr].Visible=false ESP.HealthOutline[plr].Visible=false
        end
    end
end

local function EnableESP()
    ESP.Enabled=true
    for _, plr in pairs(Players:GetPlayers()) do
        if plr~=player then
            plr.CharacterAdded:Connect(function() task.wait(0.1) CreateESP(plr) end)
            if plr.Character then CreateESP(plr) end
        end
    end
    ESP.Connections.PlayerAdded = Players.PlayerAdded:Connect(function(plr)
        if plr~=player then
            plr.CharacterAdded:Connect(function() task.wait(0.1) CreateESP(plr) end)
            if plr.Character then CreateESP(plr) end
        end
    end)
    ESP.Connections.PlayerRemoving = Players.PlayerRemoving:Connect(RemoveESP)
    ESP.Connections.RenderStepped = RunService.RenderStepped:Connect(UpdateESP)
    notify("ESP","ON")
end

local function DisableESP()
    ESP.Enabled=false
    for plr,_ in pairs(ESP.Boxes) do RemoveESP(plr) end
    if ESP.Connections.PlayerAdded then ESP.Connections.PlayerAdded:Disconnect() ESP.Connections.PlayerAdded=nil end
    if ESP.Connections.PlayerRemoving then ESP.Connections.PlayerRemoving:Disconnect() ESP.Connections.PlayerRemoving=nil end
    if ESP.Connections.RenderStepped then ESP.Connections.RenderStepped:Disconnect() ESP.Connections.RenderStepped=nil end
    notify("ESP","OFF")
end

VisualsTab:CreateButton({
    Name="👁ESP",
    Callback=function() if ESP.Enabled then DisableESP() else EnableESP() end end
})

-- ================= Respawn & Continuous Loop =================
local function onCharacterAdded(char)
    updateWalkSpeed()
    updateJumpPower()
    setupInfiniteJump()
    if flying then startFly() end
end
player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

spawn(function()
    while true do
        RunService.Heartbeat:Wait()
        local humanoid=player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            if humanoid.WalkSpeed~=walkSpeed then humanoid.WalkSpeed=walkSpeed end
            if humanoid.JumpPower~=jumpPower then humanoid.JumpPower=jumpPower end
        end
        if noclip then
            local char=player.Character
            if char then
                for _,part in pairs(char:GetChildren()) do
                    if part:IsA("BasePart") then part.CanCollide=false end
                end
            end
        end
    end
end)
