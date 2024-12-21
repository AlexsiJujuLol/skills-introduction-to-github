-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")

-- Player
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- Create Blur Effect
local blur = Instance.new("BlurEffect")
blur.Size = 15 -- Adjust blur intensity
blur.Parent = Lighting

-- Create 3D UI
local screenPart = Instance.new("Part")
screenPart.Name = "luau.dll"
screenPart.Size = Vector3.new(8, 5, 0.1)
screenPart.Anchored = true
screenPart.CanCollide = false
screenPart.Position = character.HumanoidRootPart.Position + Vector3.new(0, 5, -10)
screenPart.Parent = workspace

-- Neon Effect for Part
screenPart.Material = Enum.Material.Neon
screenPart.Color = Color3.fromRGB(0, 255, 0)

-- SurfaceGui for UI
local surfaceGui = Instance.new("SurfaceGui")
surfaceGui.Adornee = screenPart
surfaceGui.Face = Enum.NormalId.Front
surfaceGui.AlwaysOnTop = true
surfaceGui.Parent = screenPart

-- Main UI Frame
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(1, 0, 1, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 50, 20) -- Dark green background
mainFrame.BackgroundTransparency = 0.5
mainFrame.BorderSizePixel = 0
mainFrame.Parent = surfaceGui

-- Floating Luminous Dots
local function createLuminousDot()
    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 5, 0, 5) -- Size of the dot
    dot.Position = UDim2.new(math.random(), 0, math.random(), 0) -- Random start position
    dot.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Bright green
    dot.BackgroundTransparency = 0.2
    dot.BorderSizePixel = 0
    dot.Parent = mainFrame

    -- Animate Dot Floating
    local tweenInfo = TweenInfo.new(3, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1, true)
    local tween = TweenService:Create(dot, tweenInfo, {
        Position = UDim2.new(math.random(), 0, math.random(), 0)
    })
    tween:Play()
end

-- Create Multiple Dots
for _ = 1, 50 do
    createLuminousDot()
end

-- Green Glow Text
local textLabel = Instance.new("TextLabel")
textLabel.Size = UDim2.new(0.8, 0, 0.2, 0)
textLabel.Position = UDim2.new(0.1, 0, 0.4, 0)
textLabel.BackgroundTransparency = 1
textLabel.Text = "Welcome to luau.dll"
textLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
textLabel.Font = Enum.Font.SciFi
textLabel.TextScaled = true
textLabel.Parent = mainFrame

-- Glow Effect for Text
local glow = Instance.new("UIStroke")
glow.Thickness = 2
glow.Color = Color3.fromRGB(0, 255, 0)
glow.Parent = textLabel

-- Animate the UI Part Floating
local startPosition = screenPart.Position
local upTween = TweenService:Create(screenPart, TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out, -1, true), {
    Position = startPosition + Vector3.new(0, 0.5, 0)
})
upTween:Play()
