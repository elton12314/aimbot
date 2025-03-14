local camera = game.Workspace.CurrentCamera
local localplayer = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local FOV = 100 -- FOV inicial
_G.aimbot = false
_G.wallcheck = true
local targetPlayer = nil
local Dragging = false
local MovingUI = false
local DragOffset

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

-- Função para verificar se um jogador está atrás de uma parede
local function isBehindWall(player)
    local char = player.Character
    if not char or not localplayer.Character then return false end

    local rootPart = localplayer.Character:FindFirstChild("HumanoidRootPart")
    local targetHead = char:FindFirstChild("Head")

    if rootPart and targetHead then
        local origin = rootPart.Position
        local destination = targetHead.Position
        local direction = (destination - origin).unit * (destination - origin).magnitude

        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {localplayer.Character, targetHead.Parent}

        local result = workspace:Raycast(origin, direction, raycastParams)
        return result ~= nil
    end
    return false
end

-- Função para aplicar ESP (Highlight) ao jogador
local function applyESP(player)
    if not player.Character then return end
    if player == localplayer then return end
    if player.Character:FindFirstChild("Highlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Parent = player.Character
    highlight.FillColor = Color3.fromRGB(255, 0, 0) -- Cor vermelha quando está atrás da parede
    highlight.FillTransparency = 1  -- Começa invisível
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- Cor da borda
    highlight.OutlineTransparency = 0  -- Totalmente visível
    highlight.Parent = player.Character
end

-- Função para verificar visibilidade entre os jogadores
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
    for _, v in pairs(Players:GetPlayers()) do
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

    -- Atualizar ESP em tempo real
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localplayer and player.Character then
            applyESP(player)
            local highlight = player.Character:FindFirstChild("Highlight")
            local head = player.Character:FindFirstChild("Head")
            local localHead = localplayer.Character and localplayer.Character:FindFirstChild("Head")

            if highlight and localplayer.Character and localplayer:FindFirstChild("Head") and player.Character:FindFirstChild("Head") then
                local origin = localplayer.Character.Head.Position
                local destination = head.Position
                local direction = (destination - origin).unit * (origin - destination).magnitude

                local raycastParams = RaycastParams.new()
                raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                raycastParams.FilterDescendantsInstances = {localplayer.Character, player.Character} 

                local rayResult = workspace:Raycast(origin, direction, raycastParams)

                if rayResult then
                    highlight.FillTransparency = 0.3 -- Aparece brilhante quando está atrás da parede
                else
                    highlight.FillTransparency = 1 -- Fica invisível se não estiver atrás de uma parede
                end
            end
        end
    end
end)

-- Adicionar novos jogadores que entrarem no jogo
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(1)  -- Pequeno delay para garantir que o personagem seja carregado
        applyESP(player)
    end)
end)
