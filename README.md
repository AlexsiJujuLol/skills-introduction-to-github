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
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true then
        return true
    end
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
        local Balls = workspace:WaitForChild("Balls") -- Refresh the Balls folder reference
        if not Balls then return end

        for _, Ball in ipairs(Balls:GetChildren()) do
            if VerifyBall(Ball) then
                local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
                local Velocity = (Ball.Position - Ball.Position).Magnitude -- Calculate velocity based on ball movement
                
                if (Distance / Velocity) <= 10 then -- Magic threshold for parry decision
                    Parry()  -- Trigger the parry
                end
            end
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
            local Velocity = (OldPosition - Ball.Position).Magnitude -- Using distance change as velocity (since .Velocity didn't work)
            
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
