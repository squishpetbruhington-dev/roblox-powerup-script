# roblox-powerup-script
Roblox Lua power-up system: spawn items, grant temporary abilities, show GUI countdowns, and boost players during battles.






 --[[
Power-Up Battle System
Fully working Roblox Lua script ~200 lines
Features:
- Random power-ups spawn in the world
- Temporary abilities: Speed, Jump, Shield, Rocket Jump
- GUI countdown for active power-ups
- Console logs for pickups
- Surprise boosts for all players
--]]

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- RemoteEvent for GUI updates
local PowerUpEvent = Instance.new("RemoteEvent")
PowerUpEvent.Name = "PowerUpEvent"
PowerUpEvent.Parent = ReplicatedStorage

-- Table to track active power-ups per player
local ActivePowerUps = {}

-- Define Power-Up types
local PowerUps = {
    {Name = "Speed Boost", Duration = 10},
    {Name = "Jump Boost", Duration = 10},
    {Name = "Shield", Duration = 10},
    {Name = "Rocket Jump", Duration = 5}
}

-- Function to spawn a power-up part
local function spawnPowerUp()
    local part = Instance.new("Part")
    part.Size = Vector3.new(4,1,4)
    part.Anchored = true
    part.CanCollide = false
    part.Material = Enum.Material.Neon
    part.Position = Vector3.new(math.random(-50,50),5,math.random(-50,50))
    
    -- Random color and type
    local power = PowerUps[math.random(1,#PowerUps)]
    part.BrickColor = BrickColor.Random()
    part.Name = power.Name
    
    part.Parent = Workspace
    
    -- Remove after 20s if untouched
    delay(20,function()
        if part.Parent then
            part:Destroy()
        end
    end)
    
    -- Touched event
    part.Touched:Connect(function(hit)
        local player = Players:GetPlayerFromCharacter(hit.Parent)
        if not player then return end
        if ActivePowerUps[player] then return end
        
        print(player.Name.." picked up "..power.Name)
        ActivePowerUps[player] = power.Name
        PowerUpEvent:FireClient(player,power.Name,power.Duration)
        
        local char = player.Character
        if char then
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                if power.Name == "Speed Boost" then
                    humanoid.WalkSpeed = humanoid.WalkSpeed + 16
                elseif power.Name == "Jump Boost" then
                    humanoid.JumpPower = humanoid.JumpPower + 50
                elseif power.Name == "Shield" then
                    local shield = Instance.new("ForceField")
                    shield.Parent = char
                elseif power.Name == "Rocket Jump" then
                    humanoid.JumpPower = humanoid.JumpPower + 100
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end
        
        part:Destroy()
        
        -- Remove effect after duration
        delay(power.Duration,function()
            if char then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    if power.Name == "Speed Boost" then
                        humanoid.WalkSpeed = 16
                    elseif power.Name == "Jump Boost" then
                        humanoid.JumpPower = 50
                    elseif power.Name == "Shield" then
                        local shield = char:FindFirstChildOfClass("ForceField")
                        if shield then shield:Destroy() end
                    elseif power.Name == "Rocket Jump" then
                        humanoid.JumpPower = 50
                    end
                end
            end
            ActivePowerUps[player] = nil
        end)
    end)
end

-- Spawn power-ups periodically
spawn(function()
    while true do
        wait(math.random(5,10))
        spawnPowerUp()
    end
end)

-- Player joined: create GUI
Players.PlayerAdded:Connect(function(player)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PowerUpGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = player:WaitForChild("PlayerGui")
    
    local label = Instance.new("TextLabel")
    label.Name = "PowerUpLabel"
    label.Size = UDim2.new(0,300,0,50)
    label.Position = UDim2.new(0.5,-150,0,50)
    label.Text = "No Power-Up"
    label.BackgroundTransparency = 0.5
    label.BackgroundColor3 = Color3.fromRGB(0,0,0)
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.TextScaled = true
    label.Parent = screenGui
end)

-- Update GUI when player gets power-up
PowerUpEvent.OnServerEvent:Connect(function(player,powerName,duration)
    local gui = player:FindFirstChild("PlayerGui") and player.PlayerGui:FindFirstChild("PowerUpGUI")
    if gui then
        local label = gui:FindFirstChild("PowerUpLabel")
        if label then
            label.Text = powerName.." ("..duration.."s)"
            
            -- Countdown loop
            spawn(function()
                for i=duration,1,-1 do
                    label.Text = powerName.." ("..i.."s)"
                    wait(1)
                end
                label.Text = "No Power-Up"
            end)
        end
    end
end)

-- Player leaving: clear active power-up
Players.PlayerRemoving:Connect(function(player)
    ActivePowerUps[player] = nil
end)

-- Debug: print active power-ups every 10s
spawn(function()
    while true do
        wait(10)
        print("Active Power-Ups:")
        for p,v in pairs(ActivePowerUps) do
            print(p.Name.." -> "..v)
        end
    end
end)

-- Surprise global boosts every 30s
spawn(function()
    while true do
        wait(30)
        for _,player in pairs(Players:GetPlayers()) do
            local char = player.Character
            if char then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = humanoid.WalkSpeed + 8
                    delay(5,function()
                        humanoid.WalkSpeed = 16
                    end)
                end
            end
        end
        print("Everyone got a temporary speed boost!")
    end
end)

-- Optional: more debug info every 15s
spawn(function()
    while true do
        wait(15)
        print("Players in game: "..#Players:GetPlayers())
        for _,player in pairs(Players:GetPlayers()) do
            local power = ActivePowerUps[player] or "None"
            print(player.Name..": "..power)
        end
    end
end)

-- Extra fun: spawn multiple power-ups at once
spawn(function()
    while true do
        wait(25)
        local count = math.random(1,3)
        for i=1,count do
            spawnPowerUp()
        end
        print("Spawned "..count.." bonus power-ups!")
    end
end)

-- End of script
