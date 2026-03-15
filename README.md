-- [[ TOJIKAMI HUB V54 - RE-FIXED ]]
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Player = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Xoa menu cu
local oldUI = Player.PlayerGui:FindFirstChild("TJK_V54_Lite")
if oldUI then oldUI:Destroy() end

local Settings = {
    Aimbot = false,
    MagicBullet = false,
    Noclip = false,
    Esp = false,
    Fov = 150
}

-- [[ 1. GIAO DIEN ]]
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "TJK_V54_Lite"
ScreenGui.ResetOnSpawn = false

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 180, 0, 260)
Main.Position = UDim2.new(0.5, -90, 0.3, 0)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true

local Corner = Instance.new("UICorner", Main)
Corner.CornerRadius = UDim.new(0, 8)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 35)
Title.Text = "TOJIKAMI V54"
Title.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Instance.new("UICorner", Title)

local Container = Instance.new("Frame", Main)
Container.Position = UDim2.new(0, 0, 0, 40)
Container.Size = UDim2.new(1, 0, 1, -40)
Container.BackgroundTransparency = 1

local List = Instance.new("UIListLayout", Container)
List.Padding = UDim.new(0, 5)
List.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function CreateButton(name, prop)
    local btn = Instance.new("TextButton", Container)
    btn.Size = UDim2.new(0.9, 0, 0, 35)
    btn.Text = name .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.Gotham
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    btn.MouseButton1Click:Connect(function()
        Settings[prop] = not Settings[prop]
        btn.Text = name .. (Settings[prop] and ": ON" or ": OFF")
        btn.BackgroundColor3 = Settings[prop] and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(40, 40, 40)
    end)
end

CreateButton("Aimbot", "Aimbot")
CreateButton("Magic Bullet", "MagicBullet")
CreateButton("Noclip", "Noclip")
CreateButton("ESP Player", "Esp")

-- [[ 2. LOGIC ]]
local function GetTarget()
    local target = nil
    local shortestDist = Settings.Fov
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= Player and v.Character and v.Character:FindFirstChild("Head") then
            if v.Team ~= Player.Team or Player.Team == nil then
                local pos, onScreen = Camera:WorldToViewportPoint(v.Character.Head.Position)
                if onScreen then
                    local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    if mag < shortestDist then
                        shortestDist = mag
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
    if Settings.Aimbot and target and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Head.Position)
    end
    if Settings.MagicBullet and target and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(Player.Character.HumanoidRootPart.Position, Vector3.new(target.Head.Position.X, Player.Character.HumanoidRootPart.Position.Y, target.Head.Position.Z))
        end
    end
    if Settings.Noclip and Player.Character then
        for _, part in pairs(Player.Character:GetChildren()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

task.spawn(function()
    while task.wait(0.5) do
        if Settings.Esp then
            for _, v in pairs(Players:GetPlayers()) do
                if v ~= Player and v.Character and v.Character:FindFirstChild("Head") then
                    local head = v.Character.Head
                    if not head:FindFirstChild("TJK_ESP") then
                        local bill = Instance.new("BillboardGui", head)
                        bill.Name = "TJK_ESP"
                        bill.Size = UDim2.new(0, 100, 0, 50)
                        bill.AlwaysOnTop = true
                        bill.ExtentsOffset = Vector3.new(0, 3, 0)
                        local txt = Instance.new("TextLabel", bill)
                        txt.Size = UDim2.new(1, 0, 1, 0)
                        txt.BackgroundTransparency = 1
                        txt.Text = v.DisplayName
                        txt.TextColor3 = Color3.fromRGB(255, 0, 0)
                        txt.TextStrokeTransparency = 0
                        txt.Font = Enum.Font.GothamBold
                        txt.TextSize = 14
                    end
                end
            end
        end
    end
end)
