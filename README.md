-- Server-side spawner script (Roblox Lua)
-- Place in ServerScriptService
-- Requirements:
--   - ServerStorage.Latamrot (a Model)
--   - Workspace.SpawnPoints (Folder) containing Parts that mark spawn positions
-- Optional:
--   - ReplicatedStorage.SpawnLatamrot (RemoteEvent) to trigger a spawn from client (for dev tools)

local ServerStorage = game:GetService("ServerStorage")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local SPAWN_MODEL_NAME = "Latamrot"
local spawnFolder = workspace:FindFirstChild("SpawnPoints")
local spawnInterval = 20           -- seconds between auto spawns
local maxSimultaneous = 6         -- max latamrots present at once
local autoSpawnEnabled = true

-- RemoteEvent (optional) to allow clients with permission to request a spawn
local remoteEvent = ReplicatedStorage:FindFirstChild("SpawnLatamrot")
if not remoteEvent then
    remoteEvent = Instance.new("RemoteEvent")
    remoteEvent.Name = "SpawnLatamrot"
    remoteEvent.Parent = ReplicatedStorage
end

local function getSpawnPositions()
    local positions = {}
    if spawnFolder then
        for _, part in ipairs(spawnFolder:GetChildren()) do
            if part:IsA("BasePart") then
                table.insert(positions, part)
            end
        end
    end
    return positions
end

local function countExisting()
    local count = 0
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name == SPAWN_MODEL_NAME then
            count = count + 1
        end
    end
    return count
end

local function pickSpawnPart(parts)
    if #parts == 0 then return nil end
    return parts[math.random(1, #parts)]
end

local function spawnLatamrotAt(part)
    local template = ServerStorage:FindFirstChild(SPAWN_MODEL_NAME)
    if not template then
        warn("Spawner: template '"..SPAWN_MODEL_NAME.."' not found in ServerStorage.")
        return nil
    end
    local clone = template:Clone()
    clone.Name = SPAWN_MODEL_NAME
    clone.Parent = workspace

    -- Positioning: move model so its PrimaryPart sits at the spawn part's CFrame
    if clone.PrimaryPart then
        clone:SetPrimaryPartCFrame(part.CFrame + Vector3.new(0, (clone.PrimaryPart.Size.Y/2), 0))
    else
        -- If no PrimaryPart, attempt to position by bounding box
        local cf = part.CFrame
        clone:MoveTo(cf.Position + Vector3.new(0, 2, 0))
    end

    -- Optional: tag for cleanup or game logic
    local tag = Instance.new("ObjectValue")
    tag.Name = "SpawnerTag"
    tag.Value = script
    tag.Parent = clone

    return clone
end

local function tryAutoSpawn()
    if not autoSpawnEnabled then return end
    local parts = getSpawnPositions()
    if #parts == 0 then
        warn("Spawner: no spawn points found (Workspace.SpawnPoints).")
        return
    end
    if countExisting() >= maxSimultaneous then
        return
    end
    local part = pickSpawnPart(parts)
    if part then
        spawnLatamrotAt(part)
    end
end

-- Auto-spawn loop
spawn(function()
    while true do
        tryAutoSpawn()
        wait(spawnInterval)
    end
end)

-- RemoteEvent handler (server-authoritative)
remoteEvent.OnServerEvent:Connect(function(player, action)
    -- Basic permission: only allow server owner / place owner or devs (adjust as needed)
    -- For safety, we only allow players in the game's CreatorId list or with a whitelist.
    local allowed = false
    -- Example: allow only place owner(s)
    local placeOwner = game.CreatorId
    if player.UserId == placeOwner then
        allowed = true
    end

    if not allowed then
        warn("Spawner: "..player.Name.." attempted to trigger spawn but is not allowed.")
        return
    end

    if action == "spawnNow" then
        local parts = getSpawnPositions()
        local part = pickSpawnPart(parts)
        if part then
            spawnLatamrotAt(part)
        end
    end
end)

print("Spawner loaded. Auto-spawn:", tostring(autoSpawnEnabled), "Interval:", spawnInterval)
