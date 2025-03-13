local camera = game.Workspace.CurrentCamera
local localplayer = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local FOV = 100 -- FOV inicial
_G.aimbot = false
_G.wallcheck = true
local targetPlayer = nil
local Dragging = false
local MovingUI = false
local DragOffset

-- Configurações do ESP
_G.FriendColor = Color3.fromRGB(0, 0, 255)
_G.EnemyColor = Color3.fromRGB(255, 0, 0)
_G.UseTeamColor = true

-- Criar UI
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local DragButton = Instance.new("TextButton")
local AimbotStatus = Instance.new("TextLabel")
local WallCheckButton = Instance.new("TextButton")
local TextLabel = Instance.new("TextLabel")
local Slider = Instance.new("TextButton")

ScreenGui.Parent = game:GetService("CoreGui")
Frame.Parent = ScreenGui
Frame.Size = UDim2.new(0, 220, 0, 140)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BorderSizePixel = 2
Frame.Active = true

DragButton.Parent = Frame
DragButton.Size = UDim2.new(1, 0, 0, 20)
DragButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
DragButton.Text = "Arraste para mover"
DragButton.TextColor3 = Color3.new(1, 1, 1)

AimbotStatus.Parent = Frame
AimbotStatus.Size = UDim2.new(1, 0, 0, 20)
AimbotStatus.Position = UDim2.new(0, 0, 0, 25)
AimbotStatus.Text = "Aimbot: OFF (Pressione T)"
AimbotStatus.TextColor3 = Color3.new(1, 1, 1)
AimbotStatus.BackgroundTransparency = 1

WallCheckButton.Parent = Frame
WallCheckButton.Size = UDim2.new(0, 200, 0, 30)
WallCheckButton.Position = UDim2.new(0, 10, 0, 50)
WallCheckButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
WallCheckButton.Text = "WallCheck: ON"
WallCheckButton.TextColor3 = Color3.new(1, 1, 1)

TextLabel.Parent = Frame
TextLabel.Size = UDim2.new(1, 0, 0, 30)
TextLabel.Position = UDim2.new(0, 0, 0, 85)
TextLabel.Text = "FOV: " .. FOV
TextLabel.TextColor3 = Color3.new(1, 1, 1)
TextLabel.BackgroundTransparency = 1

Slider.Parent = Frame
Slider.Size = UDim2.new(0, 180, 0, 30)
Slider.Position = UDim2.new(0, 10, 0, 110)
Slider.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
Slider.Text = ""

-- Criar círculo de FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Transparency = 1
FOVCircle.Visible = true

-- Wallcheck (Verifica se há parede entre você e o alvo)
local function isVisible(target)
    if not _G.wallcheck then return true end
    local origin = camera.CFrame.Position
    local destination = target.Character.Head.Position
    local ray = Ray.new(origin, (destination - origin).unit * (destination - origin).magnitude)
    local hit, _ = workspace:FindPartOnRay(ray, localplayer.Character)

    return hit == nil or hit:IsDescendantOf(target.Character)
end

-- Aimbot: Encontra o inimigo mais próximo dentro do FOV e com visão livre
local function closestplayer()
    local dist = math.huge
    local target = nil
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= localplayer and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
            local screenPos, onScreen = camera:WorldToViewportPoint(v.Character.Head.Position)
            local magnitude = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
            if onScreen and magnitude < FOV and magnitude < dist and isVisible(v) then
                dist = magnitude
                target = v
            end
        end
    end
    return target
end

-- Alternar Aimbot com tecla T
UIS.InputBegan:Connect(function(inp)
    if inp.KeyCode == Enum.KeyCode.T then
        _G.aimbot = not _G.aimbot
        AimbotStatus.Text = _G.aimbot and "Aimbot: ON (Pressione T)" or "Aimbot: OFF (Pressione T)"
    end
end)

-- Alternar WallCheck com botão
WallCheckButton.MouseButton1Click:Connect(function()
    _G.wallcheck = not _G.wallcheck
    WallCheckButton.BackgroundColor3 = _G.wallcheck and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    WallCheckButton.Text = _G.wallcheck and "WallCheck: ON" or "WallCheck: OFF"
end)

-- Slider de FOV
Slider.MouseButton1Down:Connect(function()
    Dragging = true
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
        MovingUI = false
    end
end)

DragButton.MouseButton1Down:Connect(function()
    MovingUI = true
    DragOffset = UIS:GetMouseLocation() - Frame.AbsolutePosition
end)

RunService.RenderStepped:Connect(function()
    -- Atualizar posição do círculo FOV
    FOVCircle.Position = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    FOVCircle.Radius = FOV

    -- Atualizar valor do FOV pelo slider
    if Dragging then
        local MousePos = UIS:GetMouseLocation().X
        local SliderPos = math.clamp(MousePos - Frame.AbsolutePosition.X, 0, 180)
        FOV = math.floor((SliderPos / 180) * 300)
        Slider.Size = UDim2.new(0, SliderPos, 0, 30)
        TextLabel.Text = "FOV: " .. FOV
    end

    -- Permitir mover a interface
    if MovingUI then
        Frame.Position = UDim2.new(0, UIS:GetMouseLocation().X - DragOffset.X, 0, UIS:GetMouseLocation().Y - DragOffset.Y)
    end

    -- Atualizar alvo do aimbot
    if _G.aimbot then
        targetPlayer = closestplayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
            camera.CFrame = CFrame.new(camera.CFrame.Position, targetPlayer.Character.Head.Position)
        end
    end
end)

--------------------------------------------------------------------

-- Criar ESP
local Holder = Instance.new("Folder", game.CoreGui)
Holder.Name = "ESP"

local Box = Instance.new("BoxHandleAdornment")
Box.Name = "nilBox"
Box.Size = Vector3.new(1, 2, 1)
Box.Color3 = Color3.new(100 / 255, 100 / 255, 100 / 255)
Box.Transparency = 0.7
Box.ZIndex = 0
Box.AlwaysOnTop = false
Box.Visible = false

local NameTag = Instance.new("BillboardGui")
NameTag.Name = "nilNameTag"
NameTag.Enabled = false
NameTag.Size = UDim2.new(0, 200, 0, 50)
NameTag.AlwaysOnTop = true
NameTag.StudsOffset = Vector3.new(0, 1.8, 0)
local Tag = Instance.new("TextLabel", NameTag)
Tag.Name = "Tag"
Tag.BackgroundTransparency = 1
Tag.Position = UDim2.new(0, -50, 0, 0)
Tag.Size = UDim2.new(0, 300, 0, 20)
Tag.TextSize = 15
Tag.TextColor3 = Color3.new(100 / 255, 100 / 255, 100 / 255)
Tag.TextStrokeColor3 = Color3.new(0 / 255, 0 / 255, 0 / 255)
Tag.TextStrokeTransparency = 0.4
Tag.Text = "nil"
Tag.Font = Enum.Font.SourceSansBold
Tag.TextScaled = false

local LoadCharacter = function(v)
    repeat wait() until v.Character ~= nil
    v.Character:WaitForChild("Humanoid")
    local vHolder = Holder:FindFirstChild(v.Name)
    vHolder:ClearAllChildren()
    local b = Box:Clone()
    b.Name = v.Name .. "Box"
    b.Adornee = v.Character
    b.Parent = vHolder
    local t = NameTag:Clone()
    t.Name = v.Name .. "NameTag"
    t.Enabled = true
    t.Parent = vHolder
    t.Adornee = v.Character:WaitForChild("Head", 5)
    if not t.Adornee then
        return UnloadCharacter(v)
    end
    t.Tag.Text = v.Name
    b.Color3 = Color3.new(v.TeamColor.r, v.TeamColor.g, v.TeamColor.b)
    t.Tag.TextColor3 = Color3.new(v.TeamColor.r, v.TeamColor.g, v.TeamColor.b)
    local Update
    local UpdateNameTag = function()
        if not pcall(function()
            v.Character.Humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
            local maxh = math.floor(v.Character.Humanoid.MaxHealth)
            local h = math.floor(v.Character.Humanoid.Health)
        end) then
            Update:Disconnect()
        end
    end
    UpdateNameTag()
    Update = v.Character.Humanoid.Changed:Connect(UpdateNameTag)
end

local UnloadCharacter = function(v)
    local vHolder = Holder:FindFirstChild(v.Name)
    if vHolder and (vHolder:FindFirstChild(v.Name .. "Box") ~= nil or vHolder:FindFirstChild(v.Name .. "NameTag") ~= nil) then
        vHolder:ClearAllChildren()
    end
end

local LoadPlayer = function(v)
    local vHolder = Instance.new("Folder", Holder)
    vHolder.Name = v.Name
    v.CharacterAdded:Connect(function()
        pcall(LoadCharacter, v)
    end)
    v.CharacterRemoving:Connect(function()
        pcall(UnloadCharacter, v)
    end)
    v.Changed:Connect(function(prop)
        if prop == "TeamColor" then
            UnloadCharacter(v)
            wait()
            LoadCharacter(v)
        end
    end)
    LoadCharacter(v)
end

local UnloadPlayer = function(v)
    UnloadCharacter(v)
    local vHolder = Holder:FindFirstChild(v.Name)
    if vHolder then
        vHolder:Destroy()
    end
end

for i,v in pairs(game:GetService("Players"):GetPlayers()) do
    spawn(function() pcall(LoadPlayer, v) end)
end

game:GetService("Players").PlayerAdded:Connect(function(v)
    pcall(LoadPlayer, v)
end)

game:GetService("Players").PlayerRemoving:Connect(function(v)
    pcall(UnloadPlayer, v)
end)

game:GetService("Players").LocalPlayer.NameDisplayDistance = 0

if _G.Reantheajfdfjdgs then
    return
end

_G.Reantheajfdfjdgs = ":suifayhgvsdghfsfkajewfrhk321rk213kjrgkhj432rj34f67df"

local players = game:GetService("Players")
local plr = players.LocalPlayer

function esp(target, color)
    if target.Character then
        if not target.Character:FindFirstChild("GetReal") then
            local highlight = Instance.new("Highlight")
            highlight.RobloxLocked = true
            highlight.Name = "GetReal"
            highlight.Adornee = target.Character
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.FillColor = color
            highlight.Parent = target.Character
        else
            target.Character.GetReal.FillColor = color
        end
    end
end

while task.wait() do
    for i, v in pairs(players:GetPlayers()) do
        if v ~= plr then
            esp(v, _G.UseTeamColor and v.TeamColor.Color or ((plr.TeamColor == v.TeamColor) and _G.FriendColor or _G.EnemyColor))
        end
    end
end
