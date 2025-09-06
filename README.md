-- ===================== Sekai Hub | Full Universal Script (Fly + ESP Fixed) =====================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "🔥Sekai",
    LoadingTitle = "Sekai Hub",
    LoadingSubtitle = "by Sekai",
    ShowText = "Sekai",
    Theme = "Default",
    ToggleUIKeybind = "K"
})

-- ================== Services & Vars ==================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ================= Notification Helper =================
local function notify(title, text, duration)
    Rayfield:Notify({Title=title, Content=text, Duration=duration or 3})
end

-- ================= Player Tab =================
local MainTab = Window:CreateTab("🏡 Main", nil)
MainTab:CreateSection("Player Features")

-- Infinite Jump
local infiniteJump = false
local jumpConnection
local function setupInfiniteJump()
    if jumpConnection then jumpConnection:Disconnect() end
    jumpConnection = UIS.JumpRequest:Connect(function()
        local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if infiniteJump and hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
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
local targetWalkSpeed = walkSpeed
local function getHumanoid() return player.Character and player.Character:FindFirstChildOfClass("Humanoid") end
RunService.Heartbeat:Connect(function(dt)
    local hum = getHumanoid()
    if hum then
        hum.WalkSpeed = hum.WalkSpeed + (targetWalkSpeed - hum.WalkSpeed) * math.clamp(10*dt,0,1)
    end
end)
MainTab:CreateSlider({
    Name = "⚡Speed",
    Range = {16,300},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = walkSpeed,
    Callback = function(v) targetWalkSpeed=v notify("WalkSpeed","Speed set to "..v) end
})

-- Noclip
local noclip=false
local noclipConn
MainTab:CreateButton({
    Name="🕳Noclip",
    Callback=function()
        noclip = not noclip
        if noclip then
            noclipConn=RunService.Stepped:Connect(function()
                local char=player.Character
                if char then
                    for _,p in pairs(char:GetChildren()) do if p:IsA("BasePart") then p.CanCollide=false end end
                end
            end)
        else
            if noclipConn then noclipConn:Disconnect() noclipConn=nil end
        end
        notify("Noclip",noclip and "ON" or "OFF")
    end
})

-- ================= Fly =================
local flying=false
local flySpeed=80
local flyConn
local jumpConn
local jumpPressed=false

MainTab:CreateSlider({
    Name = "🕊 Fly Speed",
    Range = {10,300},
    Increment=1,
    Suffix="Speed",
    CurrentValue=flySpeed,
    Callback=function(v) flySpeed=v end
})

local function getHRP() return player.Character and player.Character:FindFirstChild("HumanoidRootPart") end

local function startFly()
    if flying then return end
    local hrp = getHRP()
    local hum = getHumanoid()
    if not (hrp and hum) then notify("Fly","Character not loaded") return end
    flying=true
    hum.PlatformStand=true
    jumpPressed=false

    jumpConn = UIS.JumpRequest:Connect(function() if flying then jumpPressed=true end end)
    local targetY = hrp.Position.Y

    flyConn = RunService.Heartbeat:Connect(function()
        if not (hrp and hum) then return end
        local moveDir = hum.MoveDirection*flySpeed
        local vertical = (targetY-hrp.Position.Y)*10
        if jumpPressed then vertical=flySpeed targetY=hrp.Position.Y jumpPressed=false
        elseif UIS:IsKeyDown(Enum.KeyCode.LeftShift) then vertical=-flySpeed targetY=hrp.Position.Y end
        hrp.Velocity=Vector3.new(moveDir.X,vertical,moveDir.Z)
        for _,p in pairs(player.Character:GetChildren()) do if p:IsA("BasePart") then p.CanCollide=false end end
    end)
    notify("Fly + Noclip","ON")
end

local function stopFly()
    if not flying then return end
    flying=false
    if flyConn then flyConn:Disconnect() flyConn=nil end
    if jumpConn then jumpConn:Disconnect() jumpConn=nil end
    local hum = getHumanoid()
    if hum then hum.PlatformStand=false end
    local hrp = getHRP()
    if hrp then hrp.Velocity=Vector3.zero hrp.RotVelocity=Vector3.zero end
    local char=player.Character
    if char then for _,p in pairs(char:GetChildren()) do if p:IsA("BasePart") then p.CanCollide=true end end end
    notify("Fly + Noclip","OFF")
end

MainTab:CreateButton({Name="🕊 Fly",Callback=function() if flying then stopFly() else startFly() end end})

-- ================= Visuals Tab =================
local VisualsTab = Window:CreateTab("👁 Visuals", nil)
VisualsTab:CreateSection("ESP Features")

local ESPEnabled=false
local ESPData={}
local SkeletonData={}
local espColorMode="Rainbow"
local customESPColor=Color3.fromRGB(255,0,255)

local function getESPColor(plr)
    if espColorMode=="Rainbow" then
        return Color3.fromHSV((tick()+plr.UserId)%5/5,1,1)
    elseif espColorMode=="Custom" then
        return customESPColor
    else
        return Color3.fromRGB(255,255,255)
    end
end

local function initESP(plr)
    if plr==player or ESPData[plr] then return end
    local char=plr.Character
    if not char then return end
    local head=char:FindFirstChild("Head")
    local hum=char:FindFirstChildOfClass("Humanoid")
    if not (head and hum) then return end
    local billboard=Instance.new("BillboardGui")
    billboard.Name="ESP_Billboard"
    billboard.Adornee=head
    billboard.Size=UDim2.new(0,120,0,60)
    billboard.StudsOffset=Vector3.new(0,2.5,0)
    billboard.AlwaysOnTop=true
    billboard.Parent=head
    local healthBG=Instance.new("Frame")
    healthBG.Size=UDim2.new(0,60,0,6)
    healthBG.Position=UDim2.new(0.5,-30,0,0)
    healthBG.BackgroundColor3=Color3.fromRGB(0,0,0)
    healthBG.BorderSizePixel=1
    healthBG.Parent=billboard
    local healthBar=Instance.new("Frame")
    healthBar.Size=UDim2.new(hum.Health/hum.MaxHealth,0,1,0)
    healthBar.Position=UDim2.new(0,0,0,0)
    healthBar.BackgroundColor3=getESPColor(plr)
    healthBar.BorderSizePixel=0
    healthBar.Parent=healthBG
    local nameLabel=Instance.new("TextLabel")
    nameLabel.Size=UDim2.new(1,0,0,18)
    nameLabel.Position=UDim2.new(0,0,0,6)
    nameLabel.BackgroundTransparency=1
    nameLabel.Text=plr.Name
    nameLabel.TextColor3=getESPColor(plr)
    nameLabel.TextStrokeTransparency=0
    nameLabel.TextScaled=true
    nameLabel.Parent=billboard
    local distanceLabel=Instance.new("TextLabel")
    distanceLabel.Size=UDim2.new(1,0,0,14)
    distanceLabel.Position=UDim2.new(0,0,0,25)
    distanceLabel.BackgroundTransparency=1
    distanceLabel.TextColor3=Color3.fromRGB(200,200,200)
    distanceLabel.TextStrokeTransparency=0
    distanceLabel.TextScaled=true
    distanceLabel.Parent=billboard
    local box = Drawing.new("Square")
    box.Thickness=2
    box.Transparency=1
    box.Filled=false
    box.Color=getESPColor(plr)
    local tracer = Drawing.new("Line")
    tracer.Thickness=1
    tracer.Color=getESPColor(plr)
    ESPData[plr]={char=char,billboard=billboard,healthBar=healthBar,nameLabel=nameLabel,distanceLabel=distanceLabel,box=box,tracer=tracer}
    SkeletonData[char]={}
end

local function removeESP(plr)
    if ESPData[plr] then
        local d=ESPData[plr]
        if d.billboard then d.billboard:Destroy() end
        if d.box then d.box:Remove() end
        if d.tracer then d.tracer:Remove() end
        ESPData[plr]=nil
    end
    local char=plr.Character
    if char and SkeletonData[char] then
        for _,line in pairs(SkeletonData[char]) do line:Remove() end
        SkeletonData[char]=nil
    end
end

local function enableESP()
    ESPEnabled=true
    for _,plr in pairs(Players:GetPlayers()) do
        plr.CharacterAdded:Connect(function() task.wait(0.1) initESP(plr) end)
        if plr.Character then initESP(plr) end
    end
    Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function() task.wait(0.1) initESP(plr) end)
    end)
    Players.PlayerRemoving:Connect(removeESP)
    notify("ESP","ON")
end
local function disableESP()
    ESPEnabled=false
    for plr,_ in pairs(ESPData) do removeESP(plr) end
    notify("ESP","OFF")
end

VisualsTab:CreateButton({Name="👁 Full ESP",Callback=function() if ESPEnabled then disableESP() else enableESP() end end})

-- Quick Color Buttons
VisualsTab:CreateButton({Name="🔴 Red", Callback=function() espColorMode="Custom" customESPColor=Color3.fromRGB(255,0,0) end})
VisualsTab:CreateButton({Name="⚪ White", Callback=function() espColorMode="Custom" customESPColor=Color3.fromRGB(255,255,255) end})
VisualsTab:CreateButton({Name="⚫ Black", Callback=function() espColorMode="Custom" customESPColor=Color3.fromRGB(0,0,0) end})
VisualsTab:CreateButton({Name="🟢 Green", Callback=function() espColorMode="Custom" customESPColor=Color3.fromRGB(0,255,0) end})
VisualsTab:CreateButton({Name="🌈 Rainbow", Callback=function() espColorMode="Rainbow" end})

-- ESP Update Loop
RunService.RenderStepped:Connect(function()
    if not ESPEnabled then return end
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for plr,data in pairs(ESPData) do
        local char = data.char
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local head = char and char:FindFirstChild("Head")
        if char and hum and head then
            local screenPos,onScreen = camera:WorldToViewportPoint(head.Position)
            -- Health
            local ratio = math.clamp(hum.Health/hum.MaxHealth,0,1)
            data.healthBar.Size = UDim2.new(ratio,0,1,0)
            data.healthBar.BackgroundColor3 = getESPColor(plr)
            -- Name & Distance
            data.nameLabel.Text = plr.Name.." | "..math.floor((head.Position-hrp.Position).Magnitude).."m"
            data.nameLabel.TextColor3 = getESPColor(plr)
            -- Box
            data.box.Position = Vector2.new(screenPos.X-25,screenPos.Y-50)
            data.box.Size = Vector2.new(50,100)
            data.box.Color = getESPColor(plr)
            data.box.Visible = onScreen
            -- Tracer
            data.tracer.From = Vector2.new(camera.ViewportSize.X/2,camera.ViewportSize.Y)
            data.tracer.To = Vector2.new(screenPos.X,screenPos.Y)
            data.tracer.Color = getESPColor(plr)
            data.tracer.Visible = onScreen
        else
            removeESP(plr)
        end
    end
end)

-- ================= Character Respawn Handling =================
local function onCharacterAdded(char)
    setupInfiniteJump()
    updateJumpPower()
    if flying then startFly() end
end
player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

-- ================= Continuous Enforcement =================
spawn(function()
    while true do
        RunService.Heartbeat:Wait()
        local hum=getHumanoid()
        if hum then
            if hum.WalkSpeed~=targetWalkSpeed then hum.WalkSpeed=targetWalkSpeed end
            if hum.JumpPower~=jumpPower then hum.JumpPower=jumpPower end
        end
        if noclip then
            local char=player.Character
            if char then
                for _,p in pairs(char:GetChildren()) do if p:IsA("BasePart") then p.CanCollide=false end end
            end
        end
    end
end)
