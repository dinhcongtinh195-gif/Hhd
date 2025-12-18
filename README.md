--[[ 
    THH VIP v5.1 - BLOX FRUITS SPECIAL
    - Fix lỗi không ăn Skill trong Blox Fruit
    - Hỗ trợ khóa mục tiêu cả Player và NPC
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- CẤU HÌNH
local Enabled = false
local AimRadius = 500
local TargetNPCs = true -- Ưu tiên cả NPC để dễ farm

-- GIAO DIỆN
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local MainButton = Instance.new("TextButton", ScreenGui)
MainButton.Size = UDim2.new(0, 150, 0, 50)
MainButton.Position = UDim2.new(0.5, -75, 0.1, 0)
MainButton.Text = "THH VIP: OFF"
MainButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainButton.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", MainButton)

MainButton.MouseButton1Click:Connect(function()
    Enabled = not Enabled
    MainButton.Text = "THH VIP: " .. (Enabled and "ON" or "OFF")
    MainButton.TextColor3 = Enabled and Color3.new(0, 1, 0) or Color3.new(1, 1, 1)
end)

-- HÀM TÌM MỤC TIÊU (FIX CHO BLOX FRUIT)
local function GetClosestTarget()
    local target = nil
    local shortestDist = AimRadius

    -- Quét Players
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            if v.Character.Humanoid.Health > 0 then
                local dist = (LocalPlayer.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
                if dist < shortestDist then
                    target = v.Character.HumanoidRootPart
                    shortestDist = dist
                end
            end
        end
    end

    -- Quét NPC (Dành cho Farm)
    if TargetNPCs and not target then
        for _, v in pairs(workspace.Enemies:GetChildren()) do
            if v:FindFirstChild("HumanoidRootPart") and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
                local dist = (LocalPlayer.Character.HumanoidRootPart.Position - v.HumanoidRootPart.Position).Magnitude
                if dist < shortestDist then
                    target = v.HumanoidRootPart
                    shortestDist = dist
                end
            end
        end
    end
    return target
end

-- [LÕI FIX LỖI SKILL]
local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    
    if Enabled and not checkcaller() then
        -- Blox Fruits dùng các Remote như "Skill", "Attack", "RemoteEvent"
        if method == "FireServer" or method == "InvokeServer" then
            local target = GetClosestTarget()
            if target then
                for i, v in pairs(args) do
                    -- Fix: Đè tất cả tọa độ Vector3/CFrame bằng vị trí địch
                    if typeof(v) == "Vector3" then
                        args[i] = target.Position
                    elseif typeof(v) == "CFrame" then
                        args[i] = target.CFrame
                    end
                end
                return oldNamecall(self, unpack(args))
            end
        end
    end
    return oldNamecall(self, ...)
end)

-- Hook thêm vị trí chuột thực tế để skill định hướng đúng
local oldIndex
oldIndex = hookmetamethod(game, "__index", function(self, key)
    if Enabled and self == Mouse and not checkcaller() then
        local target = GetClosestTarget()
        if target then
            if key == "Hit" then return target.CFrame end
            if key == "Target" then return target end
        end
    end
    return oldIndex(self, key)
end)
