--= SETTINGS =--
local WHITELIST = {
    [YOUR_USERID_HERE] = true,
}

local brainrotFolder = game.ReplicatedStorage:WaitForChild("Brainrots")

--= UI =--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0, 180, 0, 50)
Button.Position = UDim2.new(0.5, -90, 0.8, 0)
Button.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
Button.Text = "Spawn Brainrot"
Button.Parent = ScreenGui

-- Only show if whitelisted
if not WHITELIST[game.Players.LocalPlayer.UserId] then
    ScreenGui.Enabled = false
end

--= SPAWN FUNCTION =--
Button.MouseButton1Click:Connect(function()
    local chosen = brainrotFolder:GetChildren()[math.random(1, #brainrotFolder:GetChildren())]

    local clone = chosen:Clone()
    clone.Parent = workspace
    clone:SetPrimaryPartCFrame(
        CFrame.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
    )
end)
