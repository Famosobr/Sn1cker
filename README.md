-- // SERVIÇOS
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local playerGui = LocalPlayer:WaitForChild("PlayerGui")

-- REMOVE UI ANTIGA (ANTI DUPLICAÇÃO)
for _, v in pairs(playerGui:GetChildren()) do
    if v.Name == "KaeneWS_v5" then
        v:Destroy()
    end
end

-- // CORES
local COLORS = {
    Bg = Color3.fromRGB(15, 15, 22),
    Sidebar = Color3.fromRGB(20, 20, 28),
    Card = Color3.fromRGB(30, 30, 42),
    Text = Color3.fromRGB(255, 255, 255),
    Green = Color3.fromRGB(46, 204, 113)
}

-- // ESTADOS
local Flags = { ESP = false, Noclip = false, Spin = false, Fly = false }
local Connections = {}

-- // UI PRINCIPAL
local Screen = Instance.new("ScreenGui")
Screen.Name = "KaeneWS_v5"
Screen.ResetOnSpawn = false
Screen.DisplayOrder = 999
Screen.Parent = playerGui

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 450, 0, 350)
Main.Position = UDim2.new(0.5, 0, 0.5, 0)
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.BackgroundColor3 = COLORS.Bg
Main.Active = true
Main.Draggable = true
Main.Parent = Screen
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)

-- // BOTÃO ABRIR/FECHAR
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 120, 0, 35)
ToggleBtn.Position = UDim2.new(0, 10, 0, 10)
ToggleBtn.BackgroundColor3 = COLORS.Card
ToggleBtn.TextColor3 = COLORS.Text
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 14
ToggleBtn.Text = "Fechar"
ToggleBtn.Parent = Screen
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 8)

-- estado UI
local uiOpen = true

local function UpdateUI()
    uiOpen = not uiOpen
    Main.Visible = uiOpen
    ToggleBtn.Text = uiOpen and "Fechar" or "Abrir"
end

ToggleBtn.MouseButton1Click:Connect(UpdateUI)

-- Ctrl Direito
UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.RightControl then
        UpdateUI()
    end
end)

-- // SIDEBAR
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 120, 1, 0)
Sidebar.BackgroundColor3 = COLORS.Sidebar
Sidebar.Parent = Main
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 12)

local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(1, 0, 1, -20)
TabContainer.Position = UDim2.new(0, 0, 0, 10)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = Sidebar

local TabLayout = Instance.new("UIListLayout", TabContainer)
TabLayout.Padding = UDim.new(0, 5)

-- // CONTENT
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -130, 1, -20)
Content.Position = UDim2.new(0, 130, 0, 10)
Content.BackgroundTransparency = 1
Content.Parent = Main

local Pages = {}

local function CreatePage(name)
    local Page = Instance.new("ScrollingFrame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.ScrollBarThickness = 0
    Page.Visible = false
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Page.Parent = Content

    local L = Instance.new("UIListLayout", Page)
    L.Padding = UDim.new(0, 8)

    Pages[name] = Page
    return Page
end

local HomePage = CreatePage("Home")
local PlayerPage = CreatePage("Player")

local function ShowPage(name)
    for i, v in pairs(Pages) do
        v.Visible = (i == name)
    end
end

local function NewTab(name)
    local T = Instance.new("TextButton")
    T.Size = UDim2.new(0.9, 0, 0, 35)
    T.BackgroundColor3 = COLORS.Card
    T.Text = name
    T.TextColor3 = COLORS.Text
    T.Font = Enum.Font.GothamBold
    T.TextSize = 13
    T.Parent = TabContainer
    Instance.new("UICorner", T)

    T.MouseButton1Click:Connect(function()
        ShowPage(name)
        if name == "Player" then
            for _, v in pairs(PlayerPage:GetChildren()) do
                if v:IsA("TextButton") then v:Destroy() end
            end

            for _, p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer then
                    local btn = Instance.new("TextButton")
                    btn.Size = UDim2.new(1, -10, 0, 40)
                    btn.BackgroundColor3 = COLORS.Card
                    btn.Text = "Ir para: " .. p.Name
                    btn.TextColor3 = COLORS.Text
                    btn.Font = Enum.Font.Gotham
                    btn.Parent = PlayerPage
                    Instance.new("UICorner", btn)

                    btn.MouseButton1Click:Connect(function()
                        if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                            LocalPlayer.Character.HumanoidRootPart.CFrame =
                                p.Character.HumanoidRootPart.CFrame * CFrame.new(0,0,3)
                        end
                    end)
                end
            end
        end
    end)
end

-- // TOGGLES
local function NewToggle(text, callback)
    local B = Instance.new("TextButton")
    B.Size = UDim2.new(1, -10, 0, 45)
    B.BackgroundColor3 = COLORS.Card
    B.Text = text .. " [OFF]"
    B.TextColor3 = COLORS.Text
    B.Font = Enum.Font.GothamMedium
    B.Parent = HomePage
    Instance.new("UICorner", B)

    local active = false
    B.MouseButton1Click:Connect(function()
        active = not active
        B.Text = active and text.." [ON]" or text.." [OFF]"
        TweenService:Create(B, TweenInfo.new(0.2), {
            BackgroundColor3 = active and COLORS.Green or COLORS.Card
        }):Play()
        callback(active)
    end)
end

-- INIT
NewTab("Home")
NewTab("Player")
ShowPage("Home")

-- ESP
NewToggle("ESP", function(s)
    Flags.ESP = s
    while Flags.ESP do
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and not p.Character:FindFirstChild("ESPHighlight") then
                local h = Instance.new("Highlight", p.Character)
                h.Name = "ESPHighlight"
                h.FillColor = Color3.new(1,1,1)
            end
        end
        task.wait(1)
    end
end)

-- NOCLIP
NewToggle("Noclip", function(s)
    Flags.Noclip = s
end)

RunService.Stepped:Connect(function()
    if Flags.Noclip and LocalPlayer.Character then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

-- SPIN
NewToggle("Spin", function(s)
    Flags.Spin = s
    if s then
        Connections.Spin = RunService.Heartbeat:Connect(function()
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.CFrame *= CFrame.Angles(0, math.rad(20), 0)
            end
        end)
    elseif Connections.Spin then
        Connections.Spin:Disconnect()
    end
end)

-- FLY
NewToggle("Fly", function(s)
    Flags.Fly = s
    local char = LocalPlayer.Character
    if not char then return end

    if s and char:FindFirstChild("HumanoidRootPart") then
        local bv = Instance.new("BodyVelocity", char.HumanoidRootPart)
        bv.Name = "FlyV"
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)

        Connections.Fly = RunService.RenderStepped:Connect(function()
            bv.Velocity = char.Humanoid.MoveDirection * 50
        end)
    else
        if Connections.Fly then Connections.Fly:Disconnect() end
        local b = char:FindFirstChild("HumanoidRootPart") and char.HumanoidRootPart:FindFirstChild("FlyV")
        if b then b:Destroy() end
    end
end)

-- REJOIN
local Rj = Instance.new("TextButton")
Rj.Size = UDim2.new(1, -10, 0, 45)
Rj.BackgroundColor3 = Color3.fromRGB(45,45,60)
Rj.Text = "REJOIN SERVER"
Rj.TextColor3 = COLORS.Text
Rj.Font = Enum.Font.GothamBold
Rj.Parent = HomePage
Instance.new("UICorner", Rj)

Rj.MouseButton1Click:Connect(function()
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)
