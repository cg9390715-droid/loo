local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local configFile = "FlopTP_Positions.json"

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UltimatePositionManager"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 370)
frame.Position = UDim2.new(0.5, -160, 0.5, -185) -- Centered
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = false -- Manual dragging
frame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 16)
frameCorner.Parent = frame

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 60)
titleBar.BackgroundColor3 = Color3.fromRGB(128, 0, 128)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 16)
titleBarCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -100, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Flop tp"
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.TextSize = 26
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Close Button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 40, 0, 40)
closeBtn.Position = UDim2.new(1, -50, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 28
closeBtn.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 10)
closeCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- PERFECT DRAGGING (Works on PC + Mobile, even if drag off GUI)
local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

local function update(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

titleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Saved Positions
local savedPositions = {}

-- Safe HRP Getter
local function getHRP()
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        return char.HumanoidRootPart
    end
    return nil
end

-- Button Factory
local function createButton(text, emoji, color, yPos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 55)
    btn.Position = UDim2.new(0.05, 0, 0, yPos)
    btn.BackgroundColor3 = color
    btn.Text = emoji .. " " .. text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    btn.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = btn

    return btn
end

-- Buttons
local saveBtn = createButton("Save Current Position", "💾", Color3.fromRGB(0, 170, 255), 75)
local tpBtn = createButton("Teleport to All (Slower)", "⚡", Color3.fromRGB(0, 210, 100), 145)
local deleteBtn = createButton("Delete All Positions", "🗑️", Color3.fromRGB(230, 50, 50), 215)
local saveConfigBtn = createButton("Save Config Forever", "💾🔒", Color3.fromRGB(255, 170, 0), 285)

-- Counter Label
local countLabel = Instance.new("TextLabel")
countLabel.Size = UDim2.new(0.9, 0, 0, 30)
countLabel.Position = UDim2.new(0.05, 0, 1, -45)
countLabel.BackgroundTransparency = 1
countLabel.Text = "Saved: 0 positions"
countLabel.TextColor3 = Color3.fromRGB(180, 220, 255)
countLabel.Font = Enum.Font.GothamSemibold
countLabel.TextSize = 19
countLabel.Parent = frame

-- AUTO-LOAD CONFIG (No errors if no file/exploit)
pcall(function()
    local jsonStr = readfile(configFile)
    local positions = HttpService:JSONDecode(jsonStr)
    if positions and type(positions) == "table" then
        savedPositions = positions
        countLabel.Text = "Saved: " .. #savedPositions .. " positions"
        print("✅ Loaded " .. #savedPositions .. " positions from config!")
    end
end)

-- Button Functions
saveBtn.MouseButton1Click:Connect(function()
    local hrp = getHRP()
    if not hrp then
        saveBtn.Text = "❌ No character found!"
        task.delay(1.2, function() 
            if saveBtn and saveBtn.Parent then
                saveBtn.Text = "💾 Save Current Position"
            end
        end)
        return
    end

    table.insert(savedPositions, {hrp.Position.X, hrp.Position.Y, hrp.Position.Z})
    countLabel.Text = "Saved: " .. #savedPositions .. " positions"
    saveBtn.Text = "✅ Saved! (" .. #savedPositions .. ")"

    task.delay(1, function()
        if saveBtn and saveBtn.Parent then
            saveBtn.Text = "💾 Save Current Position"
        end
    end)
end)

tpBtn.MouseButton1Click:Connect(function()
    if #savedPositions == 0 then
        tpBtn.Text = "⚠️ No positions saved!"
        task.delay(1.2, function()
            if tpBtn and tpBtn.Parent then
                tpBtn.Text = "⚡ Teleport to All (Slower)"
            end
        end)
        return
    end

    local hrp = getHRP()
    if not hrp then
        tpBtn.Text = "❌ No character!"
        task.delay(1.2, function()
            if tpBtn and tpBtn.Parent then
                tpBtn.Text = "⚡ Teleport to All (Slower)"
            end
        end)
        return
    end

    tpBtn.Text = "Starting... (0/" .. #savedPositions .. ")"
    tpBtn.Active = false

    for i, posTable in ipairs(savedPositions) do
        tpBtn.Text = string.format("Teleporting %d/%d", i, #savedPositions)
        
        local currentHRP = getHRP()
        if currentHRP then
            -- Teleport slightly above to avoid clipping
            currentHRP.CFrame = CFrame.new(Vector3.new(posTable[1], posTable[2], posTable[3]) + Vector3.new(0, 3, 0))
        else
            tpBtn.Text = "❌ Character lost!"
            break
        end
        
        task.wait(0.12)  -- A little slower (~8 positions/sec, smooth patrol feel)
    end

    tpBtn.Text = "✅ Teleported to All!"
    task.delay(1.5, function()
        if tpBtn and tpBtn.Parent then
            tpBtn.Text = "⚡ Teleport to All (Slower)"
            tpBtn.Active = true
        end
    end)
end)

deleteBtn.MouseButton1Click:Connect(function()
    if #savedPositions == 0 then
        deleteBtn.Text = "Nothing to delete!"
        task.delay(1, function()
            if deleteBtn and deleteBtn.Parent then
                deleteBtn.Text = "🗑️ Delete All Positions"
            end
        end)
        return
    end

    savedPositions = {}
    countLabel.Text = "Saved: 0 positions"
    deleteBtn.Text = "🗑️ All Deleted!"

    task.delay(1.2, function()
        if deleteBtn and deleteBtn.Parent then
            deleteBtn.Text = "🗑️ Delete All Positions"
        end
    end)
end)

saveConfigBtn.MouseButton1Click:Connect(function()
    local oldText = saveConfigBtn.Text
    local success = pcall(function()
        local data = HttpService:JSONEncode(savedPositions)
        writefile(configFile, data)
    end)
    
    if success then
        saveConfigBtn.Text = "✅ Config Saved Forever! (" .. #savedPositions .. ")"
    else
        saveConfigBtn.Text = "❌ Failed (No writefile?)"
    end
    
    task.delay(1.5, function()
        if saveConfigBtn and saveConfigBtn.Parent then
            saveConfigBtn.Text = oldText
        end
    end)
end)

print("✅ ULTIMATE Position Manager + FOREVER CONFIG (Slower Teleport) Loaded Successfully!")
