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

-- Debug Mode
local Debug = true

-- Function for debug logging
local function debugPrint(...)
    if Debug then
        print("[DEBUG]:", ...)
    end
end

-- Auto Parry Variables
local auto_parry_enabled = false
local auto_parry_distance = 10
local spam_speed = 0.5

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Player = Players.LocalPlayer
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9)
local Balls = Workspace:WaitForChild("Balls", 9e9)

-- Function to verify valid balls
local function VerifyBall(Ball)
    return Ball and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true
end

-- Function to check if the player is the target
local function IsTarget()
    return Player.Character and Player.Character:FindFirstChild("Highlight")
end

-- Function to trigger the parry action
local function Parry()
    local ParryRemote = Remotes:FindFirstChild("ParryButtonPress")
    if ParryRemote then
        ParryRemote:Fire()
    end
end

-- Auto Parry Logic
local function AutoParry()
    while auto_parry_enabled do
        local success, errorMessage = pcall(function()
            local Balls = Workspace:FindFirstChild("Balls")
            if not Balls then return end
            for _, Ball in ipairs(Balls:GetChildren()) do
                if VerifyBall(Ball) then
                    local Distance = (Ball.Position - Workspace.CurrentCamera.Focus.Position).Magnitude
                    local Velocity = (Ball.Velocity).Magnitude
                    if (Distance / Velocity) <= auto_parry_distance then
                        debugPrint("Parrying ball at distance:", Distance, "and velocity:", Velocity)
                        Parry()
                    end
                end
            end
        end)
        if not success then
            Player:Kick("Script error: " .. errorMessage)
            return
        end
        task.wait(0.05)
    end
end

-- UI Controls for Blade Ball
BladeBall_Section.Create_Toggle({
    name = 'Auto Parry',
    flag = 'Auto_Parry',
    callback = function(state)
        auto_parry_enabled = state
        if state then
            debugPrint("Auto Parry Enabled")
            task.spawn(AutoParry)
        else
            debugPrint("Auto Parry Disabled")
        end
    end
})

BladeBall_Section.Create_Slider({
    name = 'Parry Distance',
    flag = 'BladeBall_Parry_Distance',
    min = 1,
    max = 50,
    default = auto_parry_distance,
    callback = function(value)
        auto_parry_distance = value
        debugPrint("Parry Distance set to:", value)
    end
})

BladeBall_Section.Create_Slider({
    name = 'Spam Speed',
    flag = 'BladeBall_Spam_Speed',
    min = 0.1,
    max = 1,
    default = spam_speed,
    callback = function(value)
        spam_speed = value
        debugPrint("Spam Speed set to:", value)
    end
})

debugPrint("Blade Ball UI Loaded.")

-- Create Rivals Tab
local Rivals_Tab = Library_Window.Create_Tab({
    name = 'Rivals',
    icon = 'rbxassetid://6023426975'
})

local Rivals_Section = Rivals_Tab.Create_Section()

-- Aimbot Variables
local aimbot_enabled = false
local aimbot_fov = 60

-- Aimbot Logic
local function Aimbot()
    while aimbot_enabled do
        local closest_enemy = nil
        local closest_distance = math.huge

        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Player and player.Character and player.Character:FindFirstChild("Head") then
                local enemy_head = player.Character.Head
                local distance = (enemy_head.Position - Workspace.CurrentCamera.CFrame.Position).Magnitude
                if distance < closest_distance then
                    closest_distance = distance
                    closest_enemy = player
                end
            end
        end

        if closest_enemy then
            local enemy_head = closest_enemy.Character.Head
            Workspace.CurrentCamera.CFrame = CFrame.new(Workspace.CurrentCamera.CFrame.Position, enemy_head.Position)
        end

        task.wait(0.05)
    end
end

-- UI Controls for Rivals
Rivals_Section.Create_Toggle({
    name = 'Aimbot',
    flag = 'Aimbot',
    callback = function(state)
        aimbot_enabled = state
        if state then
            debugPrint("Aimbot Enabled")
            task.spawn(Aimbot)
        else
            debugPrint("Aimbot Disabled")
        end
    end
})

Rivals_Section.Create_Slider({
    name = 'Aimbot FOV',
    flag = 'Aimbot_FOV',
    min = 10,
    max = 180,
    default = aimbot_fov,
    callback = function(value)
        aimbot_fov = value
        debugPrint("Aimbot FOV set to:", value)
    end
})

debugPrint("Rivals Tab Loaded.")
