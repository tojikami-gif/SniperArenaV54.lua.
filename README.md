# SniperArenaV54.lua.
-- [[ TOJIKAMI HUB V53.0 - EMERGENCY FIX ]]
-- Nếu bản này vẫn không hiện menu, hãy kiểm tra lại Executor của bạn.

local Player = game:GetService("Players").LocalPlayer
local Mouse = Player:GetMouse()
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

-- Xóa menu cũ
local oldUI = Player.PlayerGui:FindFirstChild("TJK_Lite")
if oldUI then oldUI:Destroy() end

local Settings = {
    Aimbot = false,
    MagicBullet = false,
    Noclip = false,
    Esp = false,
    Fov = 150
}

-- [[ 1. GIAO DIỆN CƠ BẢN (PLAYER GUI) ]]
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "TJK_Lite"
ScreenGui.ResetOnSpawn = false

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 180, 0, 250)
Main.Position = UDim2.new(0.5, -90, 0.4, 0)
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Main.BorderSizePixel = 2
Main.Active = true
Main.Draggable = true -- Delta hỗ trợ tốt Draggable trong PlayerGui

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "TJK V53 LITE"
Title.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
Title.TextColor3 = Color3.new(1, 1, 1)

local List = Instance.new("UIListLayout", Main)
List.Padding = UDim.new(0, 5)
List.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function CreateButton(name, prop)
    local btn = Instance.new("TextButton", Main)
    btn.Size = UDim2.new(0.9, 0, 0, 35)
    btn.Text = name .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    
    btn.MouseButton1Click:Connect(function()
        Settings[prop] = not Settings[prop]
        btn.Text = name .. (Settings[prop] and ": ON" or ": OFF")
        btn.BackgroundColor3 = Settings[prop] and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(50, 50, 50)
    end)
end

CreateButton("Aimbot", "Aimbot")
CreateButton("Magic Bullet", "MagicBullet")
CreateButton("Smart Noclip", "Noclip")
CreateButton("Clean ESP", "Esp")

-- [[ 2. HỆ THỐNG XỬ LÝ SIÊU NHẸ ]]
local function GetTarget()
    local target = nil
    local dist = Settings.Fov
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= Player and v.Character and v.Character:FindFirstChild("Head") then
            if v.Team ~= Player.Team or Player.Team == nil then
                local pos, on = Camera:WorldToViewportPoint(v.Character.Head.Position)
                if on then
                    local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    if mag < dist then
                        dist = mag
                        target = v.Character
                    end
                end
            end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    local target = GetTarget()
    
    -- Aimbot & Magic Bullet Logic
    if target then
        if Settings.Aimbot then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Head.Position)
        end
        -- Magic Bullet giả lập bằng cách xoay nhân vật về phía mục tiêu khi click
        if Settings.MagicBullet and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            if game:GetService("UserInputService"):IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
                 Player.Character.HumanoidRootPart.CFrame = CFrame.new(Player.Character.HumanoidRootPart.Position, Vector3.new(target.Head.Position.X, Player.Character.HumanoidRootPart.Position.Y, target.Head.Position.Z))
            end
        end
    end

    -- Noclip (Đơn giản nhất)
    if Settings.Noclip and Player.Character then
        for _, v in pairs(Player.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
    
    -- ESP Tên (Dùng BillboardGui - Không dùng Drawing API để tránh lỗi)
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= Player and v.Character and v.Character:FindFirstChild("Head") then
            local head = v.Character.Head
            local bill = head:FindFirstChild("TJK_ESP")
            if Settings.Esp then
                if not bill then
                    bill = Instance.new("BillboardGui", head)
                    bill.Name = "TJK_ESP"
                    bill.Size = UDim2.new(0, 100, 0, 50)
                    bill.AlwaysOnTop = true
                    bill.ExtentsOffset = Vector3.new(0, 3, 0)
                    local txt = Instance.new("TextLabel", bill)
                    txt.Size = UDim2.new(1, 0, 1, 0)
                    txt.BackgroundTransparency = 1
                    txt.Text = v.DisplayName
                    txt.TextColor3 = Color3.new(1, 0, 0)
                    txt.TextStrokeTransparency = 0
                end
                bill.Enabled = true
            else
                if bill then bill.Enabled = false end
            end
        end
    end
end)

