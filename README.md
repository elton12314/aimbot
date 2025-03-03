local camera = game.Workspace.CurrentCamera
local localplayer = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
_G.aimbot = false
local targetPlayer = nil

function closestplayer()
    local dist = math.huge
    local target = nil
    for i, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= localplayer and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
            local magnitude = (v.Character.Head.Position - localplayer.Character.Head.Position).magnitude
            if magnitude < dist then
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

game:GetService("RunService").RenderStepped:Connect(function()
    if _G.aimbot and targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
        camera.CFrame = CFrame.new(camera.CFrame.Position, targetPlayer.Character.Head.Position)
    end
end)
