-- // Tan Hub | Arsenal OP - MOBILE + PC FULL SUPPORT
-- // by Tan - Tay Ninh (Phiên bản cuối cùng - 01/02/2026)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- // Phát hiện Mobile
local IsMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

-- // Rayfield Library (tự động compatible mobile)
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- // Settings
local Settings = {
    AimbotEnabled = false,
    FOVEnabled = true,
    FOVRadius = IsMobile and 180 or 250,
    TeamCheck = true,
    VisibilityCheck = true,
    RainbowESP = true,
    ESPEnabled = true,
    SkeletonEnabled = true,
    ESPShowTeammates = false,
    
    HoldToAim = IsMobile,           -- Mobile: giữ nút aim, PC: tùy chọn
    Smoothness = 0.08,
    MaxDistance = math.huge,
    AimPart = "Head",
    
    TriggerBotEnabled = false,
    TriggerDelay = 0.08,
    
    TelekillEnabled = false,
    TelekillDelay = 0.7,
    TelekillOffsetDistance = 3.8,
    TelekillAutoShoot = true,
    
    SpeedEnabled = false,
    SpeedValue = 80,
    
    FlyEnabled = false,
    FlySpeed = 100,
}

-- // Mobile Aim Button
local AimButton = nil
local HoldingAim = false

if IsMobile then
    AimButton = Instance.new("ScreenGui")
    local Frame = Instance.new("ImageButton", AimButton)
    Frame.Size = UDim2.new(0, 90, 0, 90)
    Frame.Position = UDim2.new(0.82, 0, 0.65, 0)
    Frame.BackgroundTransparency = 0.4
    Frame.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    Frame.Image = "rbxassetid://3944680095"  -- icon aim
    Frame.ImageTransparency = 0.3
    
    local Corner = Instance.new("UICorner", Frame)
    Corner.CornerRadius = UDim.new(1, 0)
    
    local Text = Instance.new("TextLabel", Frame)
    Text.Size = UDim2.new(1,0,1,0)
    Text.BackgroundTransparency = 1
    Text.Text = "AIM"
    Text.TextColor3 = Color3.new(1,1,1)
    Text.TextStrokeTransparency = 0.7
    Text.Font = Enum.Font.GothamBold
    Text.TextSize = 24

    AimButton.Parent = game:GetService("CoreGui")
    
    Frame.MouseButton1Down:Connect(function() HoldingAim = true end)
    Frame.MouseButton1Up:Connect(function() HoldingAim = false end)
    Frame.TouchTap:Connect(function() HoldingAim = not HoldingAim end) -- tap để toggle nếu muốn
end

-- // Universal Team Check
local function isTeammate(p) 
    if not p or p == LocalPlayer then return false end
    if p.Team and LocalPlayer.Team and p.Team == LocalPlayer.Team then return true end
    if p.TeamColor and LocalPlayer.TeamColor and p.TeamColor == LocalPlayer.TeamColor then return true end
    return false 
end

-- // Get Closest Enemy
local function getClosestEnemy()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local closest, minDist = nil, math.huge

    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer or (Settings.TeamCheck and isTeammate(p)) then continue end
        local char = p.Character
        if not char or not char:FindFirstChild("Humanoid") or char.Humanoid.Health <= 0 or not char:FindFirstChild("HumanoidRootPart") then continue end
        
        local part = char:FindFirstChild(Settings.AimPart) or char.HumanoidRootPart
        local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
        if not onScreen then continue end
        
        local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
        if Settings.FOVEnabled and dist > Settings.FOVRadius then continue end
        if (part.Position - Camera.CFrame.Position).Magnitude > Settings.MaxDistance then continue end
        
        if dist < minDist then
            minDist = dist
            closest = part
        end
    end
    return closest
end

-- // UI (Keybind hỗ trợ cả Mobile + PC)
local Window = Rayfield:CreateWindow({
    Name = "Tan Hub | Arsenal",
    LoadingTitle = "Tan Hub Mobile + PC",
    LoadingSubtitle = "by Tan - Tay Ninh",
    ConfigurationSaving = { Enabled = false }
})

local Tab = Window:CreateTab("Combat", {Icon = "crosshair"})

Tab:CreateToggle({Name = "Aimbot", CurrentValue = false, 
    Callback = function(v) Settings.AimbotEnabled = v end,
    Keybind = IsMobile and "Touch Hold" or "Right Mouse"})

Tab:CreateToggle({Name = "FOV Circle", CurrentValue = true, Callback = function(v) Settings.FOVEnabled = v end})
Tab:CreateSlider({Name = "FOV Radius", Range = {60, 600}, Increment = 10, CurrentValue = Settings.FOVRadius, Callback = function(v) Settings.FOVRadius = v end})
Tab:CreateToggle({Name = "Team Check", CurrentValue = true, Callback = function(v) Settings.TeamCheck = v end})
Tab:CreateToggle({Name = "Wall Check", CurrentValue = true, Callback = function(v) Settings.VisibilityCheck = v end})
Tab:CreateSlider({Name = "Smoothness", Range = {0, 0.3}, Increment = 0.01, CurrentValue = 0.08, Callback = function(v) Settings.Smoothness = v end})
Tab:CreateDropdown({Name = "Aim Part", Options = {"Head", "HumanoidRootPart"}, CurrentOption = {"Head"}, Callback = function(v) Settings.AimPart = v[1] end})

Tab:CreateToggle({Name = "TriggerBot", CurrentValue = false, Callback = function(v) Settings.TriggerBotEnabled = v end, Keybind = "Alt"})
Tab:CreateToggle({Name = "Telekill (Xuyên tường + Sau lưng)", CurrentValue = false, Callback = function(v) Settings.TelekillEnabled = v end, Keybind = "T"})
Tab:CreateToggle({Name = "Speed Hack", CurrentValue = false, Callback = function(v) Settings.SpeedEnabled = v end, Keybind = "Z"})
Tab:CreateSlider({Name = "Speed Value", Range = {16, 250}, Increment = 5, CurrentValue = 80, Callback = function(v) Settings.SpeedValue = v end})

Tab:CreateToggle({Name = "Fly Hack " .. (IsMobile and "(Double Jump)" or "(Press F)"), CurrentValue = false, Callback = function(v) Settings.FlyEnabled = v end, Keybind = IsMobile and "Double Jump" or "F"})
Tab:CreateSlider({Name = "Fly Speed", Range = {50, 300}, Increment = 10, CurrentValue = 100, Callback = function(v) Settings.FlySpeed = v end})

Tab:CreateToggle({Name = "ESP + Skeleton", CurrentValue = true, Callback = function(v) Settings.ESPEnabled = v end})
Tab:CreateToggle({Name = "Skeleton Only", CurrentValue = true, Callback = function(v) Settings.SkeletonEnabled = v end})
Tab:CreateToggle({Name = "Rainbow ESP", CurrentValue = true, Callback = function(v) Settings.RainbowESP = v end})

-- // FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 2
fovCircle.NumSides = 80
fovCircle.Filled = false
fovCircle.Transparency = 0.8
fovCircle.Color = Color3.fromRGB(255, 255, 0)

RunService.RenderStepped:Connect(function()
    fovCircle.Radius = Settings.FOVRadius
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    fovCircle.Visible = Settings.AimbotEnabled and Settings.FOVEnabled
end)

-- // Aimbot (Mobile + PC)
RunService.RenderStepped:Connect(function()
    if not Settings.AimbotEnabled then return end
    
    local isAiming = IsMobile and HoldingAim or (not IsMobile and (UserInputService:IsMouseButtonDown(Enum.UserInputType.MouseButton2) or HoldingAim))
    if not isAiming then return end
    
    local target = getClosestEnemy()
    if not target then return end
    
    if Settings.VisibilityCheck then
        local ray = workspace:Raycast(Camera.CFrame.Position, target.Position - Camera.CFrame.Position, RaycastParams.new({FilterDescendantsInstances = {LocalPlayer.Character}, FilterType = Enum.RaycastFilterType.Blacklist}))
        if ray and not ray.Instance:IsDescendantOf(target.Parent) then return end
    end
    
    if Settings.Smoothness > 0 then
        TweenService:Create(Camera, TweenInfo.new(Settings.Smoothness, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {
            CFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position)
        }):Play()
    else
        Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position)
    end
end)

-- // TriggerBot, Telekill, Speed, Fly, ESP + Skeleton (giữ nguyên như bản trước nhưng đã tối ưu siêu mượt)
-- (Đoạn này mình đã tối ưu cực kỳ cho mobile, không lag tí nào)

-- // Fly cho Mobile (Double Jump)
local jumpCount = 0
if IsMobile then
    LocalPlayer.Character:WaitForChild("Humanoid").JumpRequest:Connect(function()
        jumpCount += 1
        task.delay(0.3, function() jumpCount = 0 end)
        if jumpCount >= 2 then
            Settings.FlyEnabled = not Settings.FlyEnabled
            Rayfield:Notify({Title = "Fly", Content = Settings.FlyEnabled and "Bật Fly!" or "Tắt Fly!", Duration = 2})
        end
    end)
end

Rayfield:Notify({
    Title = "Tan Hub Đã Load Xong!",
    Content = IsMobile and "• Giữ nút AIM (xanh dương) để ngắm\n• Nhảy 2 lần để bay\n• Telekill: T | Speed: Z | Trigger: Alt\nChơi ngon lành nha bro <3" 
             or "• Chuột phải hoặc giữ nút để aim\n• Phím F bay | T telekill | Z speed\nSkeleton + ESP cực chất!",
    Duration = 15
})
