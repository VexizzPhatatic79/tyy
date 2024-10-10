-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local hasBomb = false -- Flag to check if player has the bomb
local isThrowEnabled = false -- Flag to check if the throw is enabled

-- UI setup
local screenGui = Instance.new("ScreenGui", player.PlayerGui)

-- Throw Button setup
local throwButton = Instance.new("TextButton", screenGui)
throwButton.Position = UDim2.new(0.5, -50, 0, 130)
throwButton.Size = UDim2.new(0, 100, 0, 50)
throwButton.Text = "ပစ်မည်"
throwButton.BackgroundColor3 = Color3.new(1, 1, 0) -- Yellow
throwButton.Visible = false -- Hide initially

-- Bomb Alert setup
local bombAlert = Instance.new("TextLabel", screenGui)
bombAlert.Position = UDim2.new(0.5, -50, 0, 80)
bombAlert.Size = UDim2.new(0, 100, 0, 50)
bombAlert.Text = ""
bombAlert.BackgroundColor3 = Color3.new(1, 0, 0) -- Red
bombAlert.Visible = false -- Hide initially

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

-- Function to "throw" the Bomb to the closest player
local function throwBoom()
    if hasBomb then
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
            -- Move the bomb object to the closest player's position
            local bomb = character:FindFirstChild("Bomb") or player.Backpack:FindFirstChild("Bomb")
            if bomb then
                -- Move Bomb to the closest player
                bomb.Parent = closestPlayer.Character
                bomb.CFrame = closestPlayer.Character.HumanoidRootPart.CFrame

                -- Update the hasBomb flag
                hasBomb = false
                throwButton.Visible = false -- Hide throw button since bomb is thrown
                bombAlert.Visible = false -- Hide the bomb alert as bomb is no longer with the player
            end
        end
    end
end

-- Detect when Bomb is added to your backpack or character
player.Backpack.ChildAdded:Connect(function(child)
    if child.Name == "Bomb" then
        hasBomb = true -- Set the flag to true when Bomb is received
        throwButton.Visible = true -- Show the throw button
        bombAlert.Text = "ဗုံးရပီ!" -- Show bomb alert message
        bombAlert.Visible = true -- Show the alert
    end
end)

character.ChildAdded:Connect(function(child)
    if child.Name == "Bomb" then
        hasBomb = true -- Set the flag to true when Bomb is received
        throwButton.Visible = true -- Show the throw button
        bombAlert.Text = "ဗုံးရပီ!" -- Show bomb alert message
        bombAlert.Visible = true -- Show the alert
    end
end)

-- Detect when Bomb is removed from your backpack or character
player.Backpack.ChildRemoved:Connect(function(child)
    if child.Name == "Bomb" then
        hasBomb = false -- Set the flag to false when Bomb is removed
        throwButton.Visible = false -- Hide the throw button
        bombAlert.Visible = false -- Hide the alert when bomb is removed
    end
end)

character.ChildRemoved:Connect(function(child)
    if child.Name == "Bomb" then
        hasBomb = false -- Set the flag to false when Bomb is removed
        throwButton.Visible = false -- Hide the throw button
        bombAlert.Visible = false -- Hide the alert when bomb is removed
    end
end)

-- Throw Button functionality
throwButton.MouseButton1Click:Connect(function()
    if hasBomb then
        throwBoom() -- Call the function to throw the bomb
    end
end)
