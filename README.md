local camera = game.Workspace.CurrentCamera
local localplayer = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local FOV = 50 -- Ajuste o tamanho do FOV conforme necessário 
_G.aimbot = false
local targetPlayer = nil

-- Criar círculo de FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Radius = FOV
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Transparency = 1
FOVCircle.Visible = true

function closestplayer()
    local dist = math.huge
    local target = nil
    for i, v in pairs(game:GetService("Players"):GetPlayers()) do
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
        if _G.aimbot then
            targetPlayer = closestplayer()
        else
            targetPlayer = nil
        end
    end
end)

RunService.RenderStepped:Connect(function()
    -- Atualizar posição do FOVCircle
    FOVCircle.Position = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    
    if _G.aimbot and targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
        camera.CFrame = CFrame.new(camera.CFrame.Position, targetPlayer.Character.Head.Position)
    end
end)
