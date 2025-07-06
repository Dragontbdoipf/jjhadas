-- Infinity Caos Hub completo v1.0

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

local lp = Players.LocalPlayer
local function getChar()
    local c = lp.Character or lp.CharacterAdded:Wait()
    return c:WaitForChild("Humanoid"), c:WaitForChild("HumanoidRootPart"), c
end
local hum, hrp, char = getChar()
lp.CharacterAdded:Connect(getChar)

-- Anti Kick
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local old = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        if getnamecallmethod() == "Kick" then return wait(1e9) end
        return old(self, ...)
    end)
    setreadonly(mt, true)
end)

-- GUI Setup
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local main = Instance.new("Frame", ScreenGui)
main.Size = UDim2.new(0, 320, 0, 480)
main.Position = UDim2.new(0.5, -160, 0.5, -240)
main.BackgroundColor3 = Color3.fromRGB(28,28,28)
main.Active = true
main.Draggable = true

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.new(1,1,1)
title.Text = "Infinity Caos Hub"

local sc = Instance.new("ScrollingFrame", main)
sc.Size = UDim2.new(1,0,1,-40)
sc.Position = UDim2.new(0,0,0,40)
sc.CanvasSize = UDim2.new(0,0,12*55)
sc.ScrollBarThickness = 6

local y = 0
local function newToggle(name, func)
    local btn = Instance.new("TextButton", sc)
    btn.Size = UDim2.new(0.9,0,0,50)
    btn.Position = UDim2.new(0.05,0,0,y)
    btn.Text = name.." [OFF]"
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(45,45,45)
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = name.." ["..(state and "ON" or "OFF").."]"
        func(state)
    end)
    y = y + 55
    return btn
end

local function newEntry(name, callback)
    local input = Instance.new("TextBox", sc)
    input.Size = UDim2.new(0.6,0,0,40)
    input.Position = UDim2.new(0.05,0,0,y)
    input.PlaceholderText = name
    input.TextColor3 = Color3.new(1,1,1)
    input.BackgroundColor3 = Color3.fromRGB(45,45,45)
    local btn = Instance.new("TextButton", sc)
    btn.Size = UDim2.new(0.3,0,0,40)
    btn.Position = UDim2.new(0.67,0,0,y)
    btn.Text = name
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(65,65,65)
    btn.MouseButton1Click:Connect(function()
        callback(input.Text)
    end)
    y = y + 55
end

-- Functions
local flyConn, flyBV, flyBG
local function fly(on)
    if on then
        hum.PlatformStand = true
        flyBV = Instance.new("BodyVelocity",hrp); flyBV.MaxForce = Vector3.new(1e5,1e5,1e5)
        flyBG = Instance.new("BodyGyro",hrp); flyBG.MaxTorque = Vector3.new(1e5,1e5,1e5)
        flyConn = RunService.RenderStepped:Connect(function()
            local dir = Vector3.new()
            local cf = workspace.CurrentCamera.CFrame
            if UIS:IsKeyDown(Enum.KeyCode.W) then dir += cf.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= cf.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= cf.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.D) then dir += cf.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
            flyBV.Velocity = dir * 60
            flyBG.CFrame = CFrame.new(hrp.Position, hrp.Position + cf.LookVector)
        end)
    else
        if flyConn then flyConn:Disconnect() end
        if flyBV then flyBV:Destroy() end
        if flyBG then flyBG:Destroy() end
        hum.PlatformStand = false
    end
end

local noclipConn
local function noclip(on)
    if on then
        noclipConn = RunService.Stepped:Connect(function()
            for _,p in pairs(char:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end
        end)
    else
        if noclipConn then noclipConn:Disconnect() end
    end
end

local speedBtn = newToggle("Fly", fly)
-- Continue definindo outros toggles e funções...

-- Montagem
newToggle("Fly", fly)
newToggle("Noclip", noclip)
newToggle("Speed", function(o) hum.WalkSpeed = o and 100 or 16 end)
newToggle("God Mode", function(o) spawn(function() while o do hum.Health = hum.MaxHealth; RunService.Heartbeat:Wait() end end) end)
newToggle("ESP", function(o)
    -- implement highlight players
end)
newToggle("Invisibility", function(o)
    for _,p in pairs(char:GetDescendants()) do
        if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = o and 1 or 0 end
    end
end)
newToggle("TP Tool", function(o)
    -- implement tool that teleports on click
end)

newEntry("TP to Player", function(n)
    local pl = Players:FindFirstChild(n)
    if pl and pl.Character then
        hrp.CFrame = pl.Character.PrimaryPart.CFrame
    end
end)

newEntry("HeadSit", function(n)
    -- weld hrp to target head
end)

newEntry("View", function(n)
    local pl = Players:FindFirstChild(n)
    if pl and pl.Character then
        workspace.CurrentCamera.CameraSubject = pl.Character.PrimaryPart
    end
end)

print("Infinity Caos Hub carregado!")
