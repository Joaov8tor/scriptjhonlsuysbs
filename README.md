local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Ana GUI oluÅŸturma
local speedGui = Instance.new("ScreenGui")
speedGui.Name = "SpeedGui"
speedGui.ResetOnSpawn = false
speedGui.Parent = playerGui

-- Draggable buton oluÅŸturma
local speedButton = Instance.new("TextButton")
speedButton.Name = "SpeedButton"
speedButton.Size = UDim2.new(0, 100, 0, 50)
speedButton.Position = UDim2.new(0.5, -50, 0.9, -25)
speedButton.Text = "ATIVAR VELOCIDADE"
speedButton.Font = Enum.Font.SourceSansBold
speedButton.TextColor3 = Color3.new(1, 1, 1)
speedButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
speedButton.BorderSizePixel = 2
speedButton.Parent = speedGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
uiCorner.Parent = speedButton

-- MenÃ¼ oluÅŸturma
local menu = Instance.new("Frame")
menu.Name = "SpeedMenu"
menu.Size = UDim2.new(0, 0, 0, 120)
menu.Position = UDim2.new(0.5, 0, 0.5, 0)
menu.AnchorPoint = Vector2.new(0.5, 0.5)
menu.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
menu.BorderSizePixel = 2
menu.Visible = false
menu.Parent = speedGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 10)
menuCorner.Parent = menu

-- MenÃ¼ baÅŸlÄ±ÄŸÄ± ekleme
local menuTitle = Instance.new("TextLabel")
menuTitle.Name = "MenuTitle"
menuTitle.Size = UDim2.new(1, 0, 0, 30)
menuTitle.Position = UDim2.new(0, 0, 0, 0)
menuTitle.Font = Enum.Font.SourceSansBold
menuTitle.Text = "Suysbs - ADVANCED SPEED"
menuTitle.TextColor3 = Color3.new(1, 1, 1)
menuTitle.BackgroundTransparency = 1
menuTitle.Parent = menu

local speedInput = Instance.new("TextBox")
speedInput.Name = "SpeedInput"
speedInput.Size = UDim2.new(0.8, 0, 0.4, 0)
speedInput.Position = UDim2.new(0.1, 0, 0.4, 0)
speedInput.Font = Enum.Font.SourceSans
speedInput.PlaceholderText = "Enter Speed"
speedInput.Text = ""
speedInput.TextColor3 = Color3.new(1, 1, 1)
speedInput.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
speedInput.Parent = menu

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 5)
inputCorner.Parent = speedInput

-- RGB efekt fonksiyonu
local function updateRGB()
    local hue = tick() % 5 / 5
    local color = Color3.fromHSV(hue, 1, 1)
    
    menuTitle.TextColor3 = color
    menu.BorderColor3 = color
    speedButton.BorderColor3 = color
end

-- Buton sÃ¼rÃ¼kleme fonksiyonu
local dragging
local dragInput
local dragStart
local startPos

local function updateDrag(input)
    local delta = input.Position - dragStart
    speedButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

speedButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = speedButton.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

speedButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateDrag(input)
    end
end)

-- MenÃ¼ animasyonu ve iÅŸlevselliÄŸi
local menuOpen = false
local currentSpeed = 16

speedButton.MouseButton1Click:Connect(function()
    menuOpen = not menuOpen
    if menuOpen then
        menu.Visible = true
        TweenService:Create(menu, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 200, 0, 120)}):Play()
    else
        TweenService:Create(menu, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 120)}):Play()
        wait(0.3)
        menu.Visible = false
    end
end)

speedInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newSpeed = tonumber(speedInput.Text)
        if newSpeed and newSpeed > 0 then
            currentSpeed = newSpeed
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                player.Character.Humanoid.WalkSpeed = currentSpeed
            end
        end
    end
end)

-- Karakterin yeniden doÄŸmasÄ± durumunda hÄ±zÄ± koruma
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.WalkSpeed = currentSpeed
end)

-- RGB efekti gÃ¼ncelleme
RunService.RenderStepped:Connect(updateRGB)
