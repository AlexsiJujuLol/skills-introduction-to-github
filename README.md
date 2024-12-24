-- Load Library for UI (Add it in case of missing library)
local Library = loadstring(game:HttpGet('https://raw.githubusercontent.com/LuauCloud/Byte/refs/heads/main/Utils/Library.lua'))()

-- Create the main window for UI
local Library_Window = Library.Add_Window('Acceptions')

-- Create a Tab for "Blade Ball"
local BladeBall_Tab = Library_Window.Create_Tab({
    name = 'Blade Ball',
    icon = 'rbxassetid://6023426975' -- Example icon, change as needed
})

-- Create a Section for the "Blade Ball" Tab
local BladeBall_Section = BladeBall_Tab.Create_Section()

-- Auto Parry Variables
local auto_parry_enabled = false
local auto_parry_distance = 10
local spam_speed = 0.5
local Debug = false  -- Set this to true if you want debug output

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local workspace = game:GetService("Workspace")
local Player = Players.LocalPlayer
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)
local Balls = workspace:WaitForChild("Balls", 9e9)

-- Function to print debug information
local function print(...)
    if Debug then
        warn(...)
    end
end

-- Function to check if the ball is valid for parrying
local function VerifyBall(Ball)
    -- Verify if the ball is a valid instance and is in the "Balls" folder with the correct attribute
    return Ball and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true
end

-- Function to check if the player is the current target
local function IsTarget()
    return Player.Character and Player.Character:FindFirstChild("Highlight")
end

-- Function to trigger the parry action
local function Parry()
    local ParryRemote = Remotes:WaitForChild("ParryButtonPress")
    if ParryRemote then
        ParryRemote:Fire()  -- Fire the Parry action
    end
end

-- Auto Parry Logic
local function AutoParry()
    while auto_parry_enabled do
        local success, errorMessage = pcall(function()
            local Balls = workspace:WaitForChild("Balls") -- Refresh the Balls folder reference
            if not Balls then return end

            -- Iterate through all the balls and check if they are valid
            for _, Ball in ipairs(Balls:GetChildren()) do
                if VerifyBall(Ball) then
                    -- Calculate the distance between the ball and the camera's focus
                    local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
                    
                    -- Track ball's position and calculate velocity
                    local PreviousPosition = Ball.Position
                    local LastTick = tick()

                    -- Detect when the ball's position changes
                    Ball:GetPropertyChangedSignal("Position"):Connect(function()
                        local Velocity = (Ball.Position - PreviousPosition).Magnitude / (tick() - LastTick)
                        -- If the distance and velocity ratio meet the threshold, trigger parry
                        if (Distance / Velocity) <= 10 then
                            Parry()  -- Trigger the parry
                        end
                        PreviousPosition = Ball.Position -- Update the previous position
                        LastTick = tick()  -- Update the last tick time
                    end)
                end
            end
        end)
        
        if not success then
            -- If there's an error, kick the player with the message
            Player:Kick("Report to Ikorz: Script not working. Error: " .. errorMessage)
            return
        end
        
        wait(0.05) -- Frequent ball detection without lag
    end
end

-- Create Auto Parry Toggle in UI
local Auto_Parry = BladeBall_Section.Create_DropToggle({
    name = 'Auto Parry',
    section = 'left',
    flag = 'Auto_Parry',
    options = {'Custom', 'Random', 'Backwards'},
    callback = function(state)
        auto_parry_enabled = state
        if auto_parry_enabled then
            print('Auto Parry Enabled')
            task.spawn(AutoParry)  -- Start Auto Parry when enabled
        else
            print('Auto Parry Disabled')
        end
    end,

    callback2 = function(selected)
        print('Selected Auto Parry option:', selected)
    end
})

-- Example: Adjust Parry Distance
BladeBall_Section.Create_Slider({
    name = 'Parry Distance',
    flag = 'BladeBall_Parry_Distance',
    min = 1,
    max = 50,
    default = 10,
    callback = function(value)
        auto_parry_distance = value
        print('Parry Distance set to:', value)
    end
})

-- Example: Adjust Spam Speed
BladeBall_Section.Create_Slider({
    name = 'Spam Speed',
    flag = 'BladeBall_Spam_Speed',
    min = 0.1,
    max = 1,
    default = 0.5,
    callback = function(value)
        spam_speed = value
        print('Spam Speed set to:', value)
    end
})

-- Optional: Debugging message to check UI setup
print("Blade Ball Auto Parry UI Loaded.")

-- Main logic for Auto Parry when the ball is detected and the player is the target

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return -- Ignore if it's not a valid ball
    end
    
    print('Ball Spawned:', Ball) -- Debugging the ball spawn
    
    local OldPosition = Ball.Position
    local OldTick = tick()

    -- Detect when the ball's position changes
    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then -- Only proceed if the player is the target
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude / (tick() - OldTick) -- Improved velocity calculation
            
            print('Distance:', Distance, 'Velocity:', Velocity, 'Time:', Distance / Velocity)
        
            -- If the ball is moving towards the player and within range, trigger the parry
            if (Distance / Velocity) <= 10 then -- Magic number threshold, works well in practice.
                Parry() -- Parry the ball
            end
        end
        
        -- Update the old position at a fixed rate (to prevent rapid updates)
        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

-- Add a new Tab for "Rivals"
local Rivals_Tab = Library_Window.Create_Tab({
    name = 'Rivals',
    icon = 'rbxassetid://6023426975' -- Example icon, change as needed
})

-- Create a Section for the "Rivals" Tab
local Rivals_Section = Rivals_Tab.Create_Section()

-- Aimbot Variables
local aimbot_enabled = false
local aimbot_fov = 60
local target_enemy = nil

-- ESP Wall Hack Variables
local esp_enabled = false
local esp_wall_parts = {}

-- Function to detect the closest enemy
local function GetClosestEnemy()
    local closest_enemy = nil
    local closest_distance = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Player and player.Character and player.Character:FindFirstChild("Head") then
            local enemy_head = player.Character:FindFirstChild("Head")
            local distance = (enemy_head.Position - workspace.CurrentCamera.CFrame.Position).Magnitude
            if distance < closest_distance then
                closest_distance = distance
                closest_enemy = player
            end
        end
    end
    
    return closest_enemy
end

-- Aimbot Logic
local function Aimbot()
    while aimbot_enabled do
        local closest_enemy = GetClosestEnemy()
        if closest_enemy then
            local enemy_head = closest_enemy.Character:FindFirstChild("Head")
            if enemy_head then
                local screen_position, on_screen = workspace.CurrentCamera:WorldToViewportPoint(enemy_head.Position)
                if on_screen then
                    -- Calculate FOV (Field of View) for aimbot
                    local mouse_position = game:GetService("UserInputService"):GetMouseLocation()
                    local distance = (Vector2.new(screen_position.X, screen_position.Y) - mouse_position).Magnitude
                    if distance < aimbot_fov then
                        -- Smoothly move the camera to the enemy's head
                        workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, enemy_head.Position)
                    end
                end
            end
        end
        wait(0.05)
    end
end

-- ESP Wall Hack Logic
local function ToggleESP()
    for _, part in pairs(esp_wall_parts) do
        if part then
            part:Destroy() -- Remove previous ESP parts
        end
    end

    esp_wall_parts = {}
    
    if esp_enabled then
        for _, object in pairs(workspace:GetDescendants()) do
            if object:IsA("BasePart") and object.Name ~= "Terrain" then
                local esp_part = Instance.new("BillboardGui")
                esp_part.Adornee = object
                esp_part.Size = UDim2.new(0, 200, 0, 50)
                esp_part.StudsOffset = Vector3.new(0, 3, 0)
                esp_part.Parent = object
                
                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextColor3 = Color3.fromRGB(255, 0, 0)
                label.Text = object.Name
                label.Parent = esp_part
                
                table.insert(esp_wall_parts, esp_part)
            end
        end
    end
end

-- Create Aimbot Toggle in UI
Rivals_Section.Create_DropToggle({
    name = 'Aimbot',
    section = 'left',
    flag = 'Aimbot',
    options = {'Enabled', 'Disabled'},
    callback = function(state)
        aimbot_enabled = (state == 'Enabled')
        if aimbot_enabled then
            print('Aimbot Enabled')
            task.spawn(Aimbot)  -- Start Aimbot when enabled
        else
            print('Aimbot Disabled')
        end
    end
})

-- Create Aimbot FOV Slider
Rivals_Section.Create_Slider({
    name = 'Aimbot FOV',
    flag = 'Aimbot_FOV',
    min = 10,
    max = 180,
    default = 60,
    callback = function(value)
        aimbot_fov = value
        print('Aimbot FOV set to:', value)
    end
})

-- Create ESP Toggle in UI
Rivals_Section.Create_Toggle({
    name = 'ESP Wall Hack',
    section = 'left',
    flag = 'ESP_Wall_Hack',
    callback = function(state)
        esp_enabled = state
        ToggleESP()
        print('ESP Wall Hack', esp_enabled and 'Enabled' or 'Disabled')
    end
})

-- Optional: Debugging message to check UI setup
print("Rivals Tab (Aimbot & ESP) UI Loaded.")
