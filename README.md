-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local isBombTagger = false
local originalPosition = nil

-- UI setup
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
local mainButton = Instance.new("TextButton", screenGui)
local destroyButton = Instance.new("TextButton", screenGui)

mainButton.Position = UDim2.new(0.5, -50, 0, 50)
mainButton.Size = UDim2.new(0, 100, 0, 50)
mainButton.Text = "ဖွင့်မည်"
mainButton.BackgroundColor3 = Color3.new(1, 1, 1) -- White
mainButton.Draggable = true

destroyButton.Position = UDim2.new(0.5, 75, 0, 70)
destroyButton.Size = UDim2.new(0, 40, 0, 40)
destroyButton.Text = "ဖျောက်မည်"
destroyButton.BackgroundColor3 = Color3.new(1, 1, 1) -- White
destroyButton.Draggable = true

-- Function to check if player has the bomb
local function checkBombStatus()
    -- Replace with actual condition to check bomb status
    return character:FindFirstChild("Bomb") ~= nil
end

-- Function to find the closest player
local function getClosestPlayer()
    local closestPlayer = nil
    local minDistance = math.huge
    for _, otherPlayer in pairs(game.Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local distance = (humanoidRootPart.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
            if distance < minDistance then
                minDistance = distance
                closestPlayer = otherPlayer
            end
        end
    end
    return closestPlayer
end

-- Main function to handle teleportation
local function handleTeleportation()
    while isBombTagger do
        if checkBombStatus() then
            if not originalPosition then
                originalPosition = humanoidRootPart.Position
            end
            local closestPlayer = getClosestPlayer()
            if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
                humanoidRootPart.CFrame = closestPlayer.Character.HumanoidRootPart.CFrame
            end
        elseif originalPosition then
            humanoidRootPart.CFrame = CFrame.new(originalPosition)
            originalPosition = nil
        end
        wait(0.1)
    end
end

-- Button functionality
mainButton.MouseButton1Click:Connect(function()
    isBombTagger = not isBombTagger
    if isBombTagger then
        mainButton.Text = "ဖွင့်မည်"
        mainButton.BackgroundColor3 = Color3.new(0, 0, 1) -- Blue
        handleTeleportation()
    else
        mainButton.Text = "ပိတ်မည်"
        mainButton.BackgroundColor3 = Color3.new(1, 1, 1) -- White
    end
end)

-- Destroy button functionality
destroyButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)
