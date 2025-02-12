local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

-- Configurações
local settings = {
    enabled = false,
    fov = 200,
    toggleKey = Enum.KeyCode.P,
    showFov = true,
    smoothness = 1
}

-- Interface
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = player.PlayerGui

-- Status no centro da tela
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0, 200, 0, 50)
statusLabel.Position = UDim2.new(0.5, -100, 0.5, -25)
statusLabel.BackgroundTransparency = 0.5
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.Text = "Aimbot: OFF"
statusLabel.Parent = ScreenGui

-- Menu principal
local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 200, 0, 250)
menuFrame.Position = UDim2.new(0.1, 0, 0.3, 0)
menuFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = false
menuFrame.Parent = ScreenGui

-- Título do menu
local menuTitle = Instance.new("TextLabel")
menuTitle.Size = UDim2.new(1, 0, 0, 30)
menuTitle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
menuTitle.TextColor3 = Color3.new(1, 1, 1)
menuTitle.Text = "Aimbot Menu"
menuTitle.Parent = menuFrame

-- Função para criar botões do menu
local function createToggleButton(text, property, yPos)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.9, 0, 0, 30)
    button.Position = UDim2.new(0.05, 0, 0, yPos)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = text .. ": OFF"
    button.Parent = menuFrame
    
    button.MouseButton1Click:Connect(function()
        settings[property] = not settings[property]
        button.Text = text .. (settings[property] and ": ON" or ": OFF")
        button.BackgroundColor3 = settings[property] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(50, 50, 50)
    end)
    
    return button
end

-- Criar botões do menu
local enableButton = createToggleButton("Aimbot", "enabled", 40)
local fovButton = createToggleButton("Show FOV", "showFov", 80)

-- FOV Slider
local fovSlider = Instance.new("TextBox")
fovSlider.Size = UDim2.new(0.9, 0, 0, 30)
fovSlider.Position = UDim2.new(0.05, 0, 0, 120)
fovSlider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
fovSlider.TextColor3 = Color3.new(1, 1, 1)
fovSlider.Text = "FOV: " .. settings.fov
fovSlider.Parent = menuFrame

fovSlider.FocusLost:Connect(function()
    local newFov = tonumber(fovSlider.Text)
    if newFov then
        settings.fov = math.clamp(newFov, 10, 800)
        fovSlider.Text = "FOV: " .. settings.fov
    end
end)

-- Toggle menu com Right Shift
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        menuFrame.Visible = not menuFrame.Visible
    elseif input.KeyCode == settings.toggleKey then
        settings.enabled = not settings.enabled
        statusLabel.Text = "Aimbot: " .. (settings.enabled and "ON" or "OFF")
        statusLabel.BackgroundColor3 = settings.enabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    end
end)

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.NumSides = 100
fovCircle.Radius = settings.fov
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Color = Color3.new(1, 1, 1)

-- Get closest enemy
local function getClosestEnemy()
    local closest = nil
    local maxDist = settings.fov
    
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Team ~= player.Team then
            local character = otherPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                local pos = character.HumanoidRootPart.Position
                local screenPos, onScreen = camera:WorldToScreenPoint(pos)
                
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                    if dist < maxDist then
                        maxDist = dist
                        closest = character
                    end
                end
            end
        end
    end
    
    return closest
end

-- Main loop
RunService.RenderStepped:Connect(function()
    if settings.showFov then
        fovCircle.Position = Vector2.new(mouse.X, mouse.Y + 36)
        fovCircle.Radius = settings.fov
        fovCircle.Visible = true
    else
        fovCircle.Visible = false
    end

    if settings.enabled then
        local target = getClosestEnemy()
        if target then
            local pos = target.Head.Position
            local targetCFrame = CFrame.new(camera.CFrame.Position, pos)
            camera.CFrame = camera.CFrame:Lerp(targetCFrame, settings.smoothness)
        end
    end
end)
