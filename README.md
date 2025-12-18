--[[ 
    THH VIP v5 - ULTIMATE SILENT AIM (OPTIMIZED)
    - Hỗ trợ: Delta, Arceus X, Vega X, Fluxus Mobile
    - Tính năng: Silent Aim (Tự định hướng chiêu), Anti-Wall, Team Check.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- CẤU HÌNH HỆ THỐNG
local Config = {
    Enabled = false,
    AimRadius = 500,
    TeamCheck = true,
    WallCheck = false -- Đổi thành true nếu muốn chỉ aim người đứng trống
}

-- TẠO GIAO DIỆN (Đã fix lỗi hiển thị)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "THHVIP_Final"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainButton = Instance.new("TextButton")
MainButton.Parent = ScreenGui
MainButton.Size = UDim2.new(0, 160, 0, 45)
MainButton.Position = UDim2.new(0.5, -80, 0.1, 0)
MainButton.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainButton.Text = "THH VIP: OFF"
MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MainButton.Font = Enum.Font.GothamBold
MainButton.TextSize = 14
MainButton.ClipsDescendants = true

-- Bo góc và viền
local UICorner = Instance.new("UICorner", MainButton)
UICorner.CornerRadius = UDim.new(0, 10)
local UIStroke = Instance.new("UIStroke", MainButton)
UIStroke.Thickness = 2
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- Kéo thả UI (Dành cho Mobile)
local dragging, dragInput, dragStart, startPos
MainButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MainButton.Position
    end
end)
MainButton.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
MainButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
end)

-- Hiệu ứng Rainbow
task.spawn(function()
    while task.wait() do
        local hue = tick() % 5 / 5
        UIStroke.Color = Color3.fromHSV(hue, 0.7, 1)
        if Config.Enabled then MainButton.TextColor3 = Color3.fromHSV(hue, 0.7, 1) end
    end
end)

-- Xử lý Bật/Tắt
MainButton.MouseButton1Click:Connect(function()
    Config.Enabled = not Config.Enabled
    MainButton.Text = "THH VIP: " .. (Config.Enabled and "ON" or "OFF")
    MainButton.TextColor3 = Config.Enabled and Color3.new(0,1,0) or Color3.new(1,1,1)
end)

-- HÀM TÌM MỤC TIÊU TỐI ƯU
local function IsVisible(part)
    if not Config.WallCheck then return true end
    local char = LocalPlayer.Character
    if not char then return false end
    local ray = Ray.new(char.HumanoidRootPart.Position, (part.Position - char.HumanoidRootPart.Position).Unit * 500)
    local hit = workspace:FindPartOnRayWithIgnoreList(ray, {char})
    return hit and hit:IsDescendantOf(part.Parent)
end

local function GetClosestTarget()
    local target = nil
    local shortestDist = Config.AimRadius
    
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            -- Kiểm tra máu và Team
            local hum = v.Character:FindFirstChildOfClass("Humanoid")
            local isSameTeam = Config.TeamCheck and v.Team == LocalPlayer.Team
            
            if hum and hum.Health > 0 and not isSameTeam then
                local root = v.Character.HumanoidRootPart
                local dist = (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude
                
                if dist < shortestDist and IsVisible(root) then
                    target = root
                    shortestDist = dist
                end
            end
        end
    end
    return target
end

-- [HOOKING HỆ THỐNG - SILENT AIM CORE]
-- Hook Index (Dành cho các script sử dụng Mouse.Hit / Mouse.Target)
local oldIndex
oldIndex = hookmetamethod(game, "__index", function(self, key)
    if Config.Enabled and self == Mouse and not checkcaller() then
        local t = GetClosestTarget()
        if t then
            if key == "Hit" then return t.CFrame end
            if key == "Target" then return t end
        end
    end
    return oldIndex(self, key)
end)

-- Hook Namecall (Dành cho các game bắn súng/skill dùng RemoteEvent)
local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    
    if Config.Enabled and not checkcaller() and (method == "FireServer" or method == "InvokeServer") then
        local t = GetClosestTarget()
        if t then
            for i, arg in pairs(args) do
                if typeof(arg) == "Vector3" then
                    args[i] = t.Position
                elseif typeof(arg) == "CFrame" then
                    args[i] = t.CFrame
                end
            end
        end
    end
    return oldNamecall(self, unpack(args))
end)

print("THH VIP v5 Loaded Successfully!")
