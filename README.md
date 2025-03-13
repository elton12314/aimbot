local camera = game.Workspace.CurrentCamera
local localplayer = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local FOV = 100 -- FOV inicial
_G.aimbot = false
local targetPlayer = nil
local Dragging = false
local MovingUI = false
local DragOffset

-- Criar UI
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local TextLabel = Instance.new("TextLabel")
local Slider = Instance.new("TextButton")
local DragButton = Instance.new("TextButton")

ScreenGui.Parent = game:GetService("CoreGui")
Frame.Parent = ScreenGui
Frame.Size = UDim2.new(0, 200, 0, 80)
Frame.Position = UDim2.new(0, 50, 0, 50)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BorderSizePixel = 2
Frame.Active = true

DragButton.Parent = Frame
DragButton.Size = UDim2.new(1, 0, 0, 20)
DragButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
DragButton.Text = ""
DragButton.TextColor3 = Color3.new(1, 1, 1)

TextLabel.Parent = Frame
TextLabel.Size = UDim2.new(1, 0, 0, 30)
TextLabel.Position = UDim2.new(0, 0, 0, 20)
TextLabel.Text = "FOV: " .. FOV
TextLabel.TextColor3 = Color3.new(1, 1, 1)
TextLabel.BackgroundTransparency = 1

Slider.Parent = Frame
Slider.Size = UDim2.new(0, 180, 0, 30)
Slider.Position = UDim2.new(0, 10, 0, 50)
Slider.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
Slider.Text = ""

-- Criar círculo de FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Transparency = 1
FOVCircle.Visible = true

function closestplayer()
    local dist = math.huge
    local target = nil
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= localplayer and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
            local screenPos, onScreen = camera:WorldToViewportPoint(v.Character.Head.Position)
            local magnitude = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
            if onScreen and magnitude < FOV and magnitude < dist then
                dist = magnitude
                target = v
            end
        end
    end
    return target
end

UIS.InputBegan:Connect(function(inp)
    if inp.KeyCode == Enum.KeyCode.T then
        _G.aimbot = not _G.aimbot
    end
end)

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

    
    if _G.aimbot then
        targetPlayer = closestplayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
            camera.CFrame = CFrame.new(camera.CFrame.Position, targetPlayer.Character.Head.Position)
        end
    end
end)
