-- Load necessary services
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")

-- Load the Cloud UI Library
local Library = loadstring(game:HttpGet('https://raw.githubusercontent.com/LuauCloud/Byte/refs/heads/main/Utils/Library.lua'))()
local Library_Window = Library.Add_Window('Auto Parry Settings')

-- Variables for Auto Parry
local auto_parry_enabled = false
local auto_parry_distance = 10
local show_parry_circle = false
local parry_circle = nil
local last_parry_time = 0
local parry_cooldown = 0.1  -- Cooldown between parries to avoid too many actions

-- Function to update the parry circle's position and size
local function UpdateParryCircle()
    if not show_parry_circle then
        if parry_circle then
            parry_circle:Destroy()
            parry_circle = nil
        end
        return
    end

    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end

    if not parry_circle then
        parry_circle = Instance.new("Part")
        parry_circle.Shape = Enum.PartType.Cylinder
        parry_circle.Anchored = true
        parry_circle.CanCollide = false
        parry_circle.Transparency = 0.5
        parry_circle.Material = Enum.Material.Neon
        parry_circle.Color = Color3.new(1, 0, 0)
        parry_circle.Parent = workspace
    end

    parry_circle.Size = Vector3.new(0.2, auto_parry_distance * 2, auto_parry_distance * 2)
    parry_circle.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position) * CFrame.Angles(math.pi / 2, 0, 0)
end

-- Function to continuously update the parry circle's position
local function UpdateCirclePosition()
    while auto_parry_enabled and show_parry_circle do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            parry_circle.Position = player.Character.HumanoidRootPart.Position
        end
        task.wait(0.05)
    end
end

-- Function to automatically parry balls with enhanced performance
local function AutoParryBall()
    while auto_parry_enabled do
        if not player.Character then return end

        local character = player.Character
        local rootPart = character:FindFirstChild("HumanoidRootPart")

        if rootPart and tick() - last_parry_time > parry_cooldown then
            last_parry_time = tick()  -- Update last parry time

            local nearby_ball = nil
            local min_distance = auto_parry_distance

            -- Find the nearest ball within range
            for _, ball in pairs(workspace:GetChildren()) do
                if ball.Name == "Ball" and ball:IsA("Part") then
                    local distance = (ball.Position - rootPart.Position).Magnitude
                    if distance <= min_distance then
                        nearby_ball = ball
                        min_distance = distance
                    end
                end
            end

            -- If a ball is within range, apply the parry logic
            if nearby_ball then
                local bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
                bodyVelocity.Velocity = (nearby_ball.Position - rootPart.Position).unit * 100
                bodyVelocity.Parent = nearby_ball
                Debris:AddItem(bodyVelocity, 0.5)  -- Clean up after parry
                print("Parried a ball!")
            end
        end
        task.wait(0.05)  -- Lower frequency of checks for performance
    end
end

-- Function to toggle auto parry and activate the external auto-block script
local function ToggleAutoParry(state)
    auto_parry_enabled = state
    if state then
        -- Start updating circle position and parrying balls
        task.spawn(UpdateCirclePosition)
        task.spawn(AutoParryBall)
        
        -- Activate Auto-Block with Red Circle (external script)
        loadstring(game:HttpGet("https://raw.githubusercontent.com/1f0yt/community/main/Circle"))()
        print("Auto-Parry and Auto-Block Activated!")
    else
        if parry_circle then
            parry_circle:Destroy()  -- Remove parry circle when disabled
            parry_circle = nil
        end
        print("Auto-Parry and Auto-Block Disabled!")
    end
end

-- Cloud UI Tab Creation
local BladeBall_Tab = Library_Window.Create_Tab({name = 'Blade Ball', icon = 'rbxassetid://6023426975'})

-- Section for Auto Parry Settings
local BladeBall_Section = BladeBall_Tab.Create_Section({name = 'Auto Parry Settings'})

-- Toggle for Auto Parry
BladeBall_Section.Create_Toggle({
    name = 'Enable Auto Parry',
    flag = 'Auto_Parry',
    callback = function(state)
        ToggleAutoParry(state)
    end
})

-- Slider to adjust parry distance
BladeBall_Section.Create_Slider({
    name = 'Parry Distance',
    flag = 'Parry_Distance',
    min = 5,
    max = 50,
    default = auto_parry_distance,
    callback = function(value)
        auto_parry_distance = value
        UpdateParryCircle()  -- Update the circle whenever the distance changes
    end
})

-- Toggle to show or hide the parry circle
BladeBall_Section.Create_Toggle({
    name = 'Show Parry Circle',
    flag = 'Show_Parry_Circle',
    callback = function(state)
        show_parry_circle = state
        UpdateParryCircle()  -- Update the circle's visibility
    end
})

-- Finalize the UI setup
Library_Window:Show()

print("Auto Parry UI Loaded! Press the button to toggle.")
