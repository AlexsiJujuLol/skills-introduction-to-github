local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local RemoteEvent = ReplicatedStorage:WaitForChild("TestRemoteEvent")
local Player = Players.LocalPlayer

-- Create a blur effect
local blurEffect = Instance.new("BlurEffect")
blurEffect.Size = 0 -- Start with no blur
blurEffect.Parent = Workspace.CurrentCamera

-- Create a part to display the 3D UI
local uiPart = Instance.new("Part")
uiPart.Size = Vector3.new(6, 4, 1)
uiPart.Anchored = true
uiPart.CanCollide = false
uiPart.Position = Workspace.CurrentCamera.CFrame.Position + Workspace.CurrentCamera.CFrame.LookVector * 10
uiPart.Parent = Workspace

-- Attach SurfaceGui to the part
local surfaceGui = Instance.new("SurfaceGui")
surfaceGui.Face = Enum.NormalId.Front
surfaceGui.Adornee = uiPart
surfaceGui.AlwaysOnTop = true
surfaceGui.Parent = uiPart

-- Create a button on the SurfaceGui
local parryButton = Instance.new("TextButton")
parryButton.Size = UDim2.new(0.9, 0, 0.3, 0) -- Adjust button size
parryButton.Position = UDim2.new(0.05, 0, 0.35, 0) -- Center on the SurfaceGui
parryButton.Text = "Parry"
parryButton.Font = Enum.Font.FredokaOne
parryButton.TextSize = 36
parryButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
parryButton.TextColor3 = Color3.new(1, 1, 1)
parryButton.Parent = surfaceGui

-- Add a creamy sound effect to the UI part
local creamySound = Instance.new("Sound")
creamySound.SoundId = "rbxassetid://YOUR_SOUND_ID" -- Replace with your creamy sound's asset ID
creamySound.Volume = 1
creamySound.PlayOnRemove = false
creamySound.Parent = uiPart

-- Add glowing points (ParticleEmitter)
local particleEmitter = Instance.new("ParticleEmitter")
particleEmitter.LightEmission = 1
particleEmitter.Size = NumberSequence.new(0.1, 0.5) -- Small glowing particles
particleEmitter.Texture = "rbxassetid://258128463" -- Use a glowing circle texture
particleEmitter.Lifetime = NumberRange.new(1, 3)
particleEmitter.Rate = 50
particleEmitter.Speed = NumberRange.new(0.5, 1)
particleEmitter.VelocitySpread = 180
particleEmitter.Color = ColorSequence.new(Color3.fromRGB(255, 255, 128)) -- Soft yellow glow
particleEmitter.Parent = uiPart

-- Button click functionality to trigger parry and play sound
parryButton.MouseButton1Click:Connect(function()
    RemoteEvent:FireServer("Parry")
    creamySound:Play()
    
    -- Activate blur effect
    for i = 1, 10 do
        blurEffect.Size = i * 2 -- Gradually increase blur
        wait(0.05)
    end
end)

-- Rotate the UI to always face the player
game:GetService("RunService").RenderStepped:Connect(function()
    uiPart.CFrame = CFrame.new(uiPart.Position, Workspace.CurrentCamera.CFrame.Position)
end)

-- Deactivate blur when leaving
Player.CharacterRemoving:Connect(function()
    for i = 10, 1, -1 do
        blurEffect.Size = i * 2 -- Gradually decrease blur
        wait(0.05)
    end
    blurEffect:Destroy()
end)
