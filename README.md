-- Load Library for UI (Add it in case of missing library)
local Library = loadstring(game:HttpGet('https://raw.githubusercontent.com/LuauCloud/Byte/refs/heads/main/Utils/Library.lua'))()

-- Create the main window for UI
local Library_Window = Library.Add_Window('Acceptions')

-- **Services**
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Player = Players.LocalPlayer

-- **Debugging and Variables**
local Debug = false -- Set to `true` to enable debug output
local function debugPrint(...)
    if Debug then
        warn("[DEBUG]:", ...)
    end
end

local auto_parry_enabled = false
local auto_parry_distance = 10
local spam_speed = 0.5

local aimbot_enabled = false
local aimbot_fov = 60
local esp_enabled = false
local esp_wall_parts = {}

local Remotes = ReplicatedStorage:FindFirstChild("Remotes")
if not Remotes then
    warn("Remotes folder not found. Script will not run.")
    return
end

local Balls = Workspace:FindFirstChild("Balls")
if not Balls then
    warn("Balls container not found. Script will not run.")
    return
end

-- **Utility Functions**
local function VerifyBall(Ball)
    if not Ball then
        debugPrint("Ball is nil")
        return false
    end
    if not Ball:IsA("BasePart") then
        debugPrint("Invalid Ball type:", Ball.ClassName)
        return false
    end
    if not Ball:GetAttribute("realBall") then
        debugPrint("Ball is missing 'realBall' attribute")
        return false
    end
    return true
end

local function IsTarget()
    if not Player.Character then
        debugPrint("Player character not loaded")
        return false
    end
    if not Player.Character:FindFirstChild("Highlight") then
        debugPrint("Player is not a target")
        return false
    end
    return true
end

local function Parry()
    local ParryRemote = Remotes:FindFirstChild("ParryButtonPress")
    if ParryRemote then
        ParryRemote:Fire()
    else
        debugPrint("ParryButtonPress remote not found")
    end
end

-- **Auto Parry Logic**
local function AutoParry()
    while auto_parry_enabled do
        local success, errorMessage = pcall(function()
            for _, Ball in ipairs(Balls:GetChildren()) do
                if VerifyBall(Ball) then
                    local OldPosition = Ball.Position
                    local OldTick = tick()
                    Ball:GetPropertyChangedSignal("Position"):Connect(function()
                        if IsTarget() then
                            local Distance = (Ball.Position - Workspace.CurrentCamera.Focus.Position).Magnitude
                            local Velocity = (OldPosition - Ball.Position).Magnitude / (tick() - OldTick)
                            if (Distance / Velocity) <= auto_parry_distance then
                                Parry()
                            end
                            OldPosition = Ball.Position
                            OldTick = tick()
                        end
                    end)
                end
            end
        end)
        if not success then
            debugPrint("Error in AutoParry:", errorMessage)
            Player:Kick("Script error: " .. errorMessage)
            return
        end
        task.wait(0.05)
    end
    debugPrint("Auto Parry stopped")
end

-- **Aimbot Logic**
local function GetClosestEnemy()
    local closest_enemy, closest_distance = nil, math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Player and player.Character and player.Character:FindFirstChild("Head") then
            local distance = (player.Character.Head.Position - Workspace.CurrentCamera.CFrame.Position).Magnitude
            if distance < closest_distance then
                closest_distance = distance
                closest_enemy = player
            end
        end
    end
    return closest_enemy
end

local function Aimbot()
    while aimbot_enabled do
        local closest_enemy = GetClosestEnemy()
        if closest_enemy and closest_enemy.Character then
            local enemy_head = closest_enemy.Character:FindFirstChild("Head")
            if enemy_head then
                local screen_pos, on_screen = Workspace.CurrentCamera:WorldToViewportPoint(enemy_head.Position)
                if on_screen then
                    local mouse_pos = game:GetService("UserInputService"):GetMouseLocation()
                    local distance = (Vector2.new(screen_pos.X, screen_pos.Y) - mouse_pos).Magnitude
                    if distance < aimbot_fov then
                        Workspace.CurrentCamera.CFrame = CFrame.new(Workspace.CurrentCamera.CFrame.Position, enemy_head.Position)
                    end
                end
            end
        end
        task.wait(0.05)
    end
    debugPrint("Aimbot stopped")
end

-- **ESP Logic**
local function ToggleESP()
    for _, part in pairs(esp_wall_parts) do
        if part then part:Destroy() end
    end
    esp_wall_parts = {}
    if esp_enabled then
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Name ~= "Terrain" then
                local esp = Instance.new("BillboardGui", obj)
                esp.Size = UDim2.new(0, 200, 0, 50)
                esp.StudsOffset = Vector3.new(0, 3, 0)
                local label = Instance.new("TextLabel", esp)
                label.Size = UDim2.new(1, 0, 1, 0)
                label.Text = obj.Name
                label.TextColor3 = Color3.new(1, 0, 0)
                label.BackgroundTransparency = 1
                table.insert(esp_wall_parts, esp)
            end
        end
    end
end

-- **UI Setup**
local BladeBall_Tab = Library_Window.Create_Tab({name = 'Blade Ball', icon = 'rbxassetid://6023426975'})
local BladeBall_Section = BladeBall_Tab.Create_Section()

BladeBall_Section.Create_Toggle({
    name = 'Auto Parry',
    flag = 'Auto_Parry',
    callback = function(state)
        auto_parry_enabled = state
        if state then
            task.spawn(AutoParry)
        end
        debugPrint("Auto Parry", state and "Enabled" or "Disabled")
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

local Rivals_Tab = Library_Window.Create_Tab({name = 'Rivals', icon = 'rbxassetid://6023426975'})
local Rivals_Section = Rivals_Tab.Create_Section()

Rivals_Section.Create_Toggle({
    name = 'Aimbot',
    flag = 'Aimbot',
    callback = function(state)
        aimbot_enabled = state
        if state then
            task.spawn(Aimbot)
        end
        debugPrint("Aimbot", state and "Enabled" or "Disabled")
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

Rivals_Section.Create_Toggle({
    name = 'ESP Wall Hack',
    flag = 'ESP_Wall_Hack',
    callback = function(state)
        esp_enabled = state
        ToggleESP()
        debugPrint("ESP", state and "Enabled" or "Disabled")
    end
})
