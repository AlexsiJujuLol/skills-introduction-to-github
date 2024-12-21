-- Services
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Camera = game:GetService("Workspace").CurrentCamera
local RemoteEvent = ReplicatedStorage:WaitForChild("TestRemoteEvent")

local Player = Players.LocalPlayer
local character = Player.Character or Player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local playerGui = Player:WaitForChild("PlayerGui")

-- Create a blur effect
local blurEffect = Instance.new("BlurEffect")
blurEffect.Size = 0  -- Start with no blur
blurEffect.Parent = Camera

-- UI Setup (3D UI)
local function create3DUI()
    -- Create the BillboardGui (3D UI container)
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Size = UDim2.new(0, 300, 0, 100)  -- Adjust size of the UI
    billboardGui.StudsOffset = Vector3.new(0, 5, 0)  -- Position the UI above the character
    billboardGui.Adornee = character:WaitForChild("Head")  -- Attach to player's head
    billboardGui.Parent = Workspace

    -- Create a Frame for background with transparency
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)  -- Fill the entire BillboardGui
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)  -- Black background
    frame.BackgroundTransparency = 0.6  -- Semi-transparent black background
    frame.Parent = billboardGui

    -- Create a TextLabel for content
    local textLabel = Instance.new("TextLabel")
    textLabel.Text = "Smooth 3D UI with Animation"
    textLabel.Size = UDim2.new(1, 0, 1, 0)  -- Fill the frame
    textLabel.BackgroundTransparency = 1  -- No background for the label
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)  -- White text
    textLabel.TextScaled = true  -- Text will scale to fit
    textLabel.Parent = frame

    -- Apply smooth animations to the TextLabel
    local tweenInfo = TweenInfo.new(5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, -1, true)  -- Loop the animation
    local goal = {Rotation = 360}  -- Rotate the UI smoothly
    local tween = TweenService:Create(textLabel, tweenInfo, goal)
    tween:Play()
end

-- BladeBall Setup
local function createBladeBall()
    local bladeBall = Instance.new("Model")
    bladeBall.Name = "BladeBall"
    bladeBall.Parent = Workspace

    local ballPart = Instance.new("Part")
    ballPart.Name = "BallPart"
    ballPart.Shape = Enum.PartType.Ball
    ballPart.Size = Vector3.new(2, 2, 2)
    ballPart.Position = Camera.CFrame.Position + Vector3.new(10, 5, 0)  -- Spawn above the player
    ballPart.Anchored = false
    ballPart.CanCollide = true
    ballPart.Parent = bladeBall

    -- Apply velocity to simulate the ball's movement
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
    bodyVelocity.Velocity = Vector3.new(math.random(-50, 50), 0, math.random(-50, 50)) * 10
    bodyVelocity.Parent = ballPart

    -- Return BladeBall for further manipulation if needed
    return bladeBall
end

-- Auto-parry spam mechanism
local autoParryEnabled = false
local parryInterval = 0.1  -- Interval for auto-parry spam (in seconds)

local function triggerParry()
    RemoteEvent:FireServer("Parry")
end

local function autoParry()
    while autoParryEnabled do
        -- Check if a BladeBall is in range for parry
        for _, ball in pairs(Workspace:GetChildren()) do
            if ball:IsA("Model") and ball.Name == "BladeBall" then
                local ballPart = ball:FindFirstChild("BallPart")
                if ballPart and (humanoidRootPart.Position - ballPart.Position).Magnitude <= 10 then
                    triggerParry()
                    wait(parryInterval)
                end
            end
        end
        wait(0.1) -- Check periodically
    end
end

-- Button to toggle auto-parry
local toggleAutoParryButton = Instance.new("TextButton")
toggleAutoParryButton.Size = UDim2.new(0.9, 0, 0.3, 0)
toggleAutoParryButton.Position = UDim2.new(0.05, 0, 0.7, 0)
toggleAutoParryButton.Text = "Auto Parry: Off"
toggleAutoParryButton.Font = Enum.Font.FredokaOne
toggleAutoParryButton.TextSize = 36
toggleAutoParryButton.BackgroundColor3 = Color3.fromRGB(85, 255, 85)
toggleAutoParryButton.TextColor3 = Color3.new(1, 1, 1)
toggleAutoParryButton.Parent = playerGui:WaitForChild("ScreenGui")

toggleAutoParryButton.MouseButton1Click:Connect(function()
    autoParryEnabled = not autoParryEnabled
    if autoParryEnabled then
        toggleAutoParryButton.Text = "Auto Parry: On"
        spawn(autoParry)  -- Start auto parry when enabled
    else
        toggleAutoParryButton.Text = "Auto Parry: Off"
    end
end)

-- Create BladeBalls periodically
spawn(function()
    while true do
        wait(5) -- Create a new BladeBall every 5 seconds
        createBladeBall()
    end
end)

-- UI and Animation
local function playWelcomeAnimation()
    local textLabel = Instance.new("TextLabel")
    textLabel.Text = "Welcome to BladeBall"
    textLabel.Size = UDim2.new(0.9, 0, 0.3, 0)
    textLabel.Position = UDim2.new(0.05, 0, 0.35, 0)
    textLabel.TextSize = 36
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.BackgroundTransparency = 1
    textLabel.Parent = playerGui:WaitForChild("ScreenGui")

    -- Animate the text
    local tweenInfo = TweenInfo.new(3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, 0, false)
    local goal = {Position = UDim2.new(0.05, 0, 0.2, 0)}
    local tween = TweenService:Create(textLabel, tweenInfo, goal)
    tween:Play()
    
    -- Wait before removing the text
    wait(3)
    textLabel:Destroy()
end

-- Call the welcome animation when the player joins
playWelcomeAnimation()

-- Rotation and Movement of the UI
game:GetService("RunService").RenderStepped:Connect(function()
    -- Always face the camera
    -- You can add smooth movement or animations as required for your UI part
end)
