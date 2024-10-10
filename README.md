-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local isThrowEnabled = false
local originalPosition = nil -- Store original position when throw starts
local hasBomb = false -- Flag to check if player has the bomb

-- UI setup
local screenGui = Instance.new("ScreenGui", player.PlayerGui)

-- Enable Button setup
local enableButton = Instance.new("TextButton", screenGui)
enableButton.Position = UDim2.new(0.5, -25, 0, 50)
enableButton.Size = UDim2.new(0, 50, 0, 25)
enableButton.Text = "ဖွင့်မည်"
enableButton.BackgroundColor3 = Color3.new(0, 1, 0) -- Green

-- Disable Button setup
local disableButton = Instance.new("TextButton", screenGui)
disableButton.Position = UDim2.new(0.5, -25, 0, 90)
disableButton.Size = UDim2.new(0, 50, 0, 25)
disableButton.Text = "ပိတ်မည်"
disableButton.BackgroundColor3 = Color3.new(1, 0, 0) -- Red
disableButton.Visible = false -- Hide the disable button initially

-- Throw Button setup
local throwButton = Instance.new("TextButton", screenGui)
throwButton.Position = UDim2.new(0.5, -25, 0, 130)
throwButton.Size = UDim2.new(0, 50, 0, 25)
throwButton.Text = "ပစ်မည်"
throwButton.BackgroundColor3 = Color3.new(1, 1, 0) -- Yellow
throwButton.Visible = false -- Hide initially, show only when enabled

-- Alert Button setup
local alertButton = Instance.new("TextButton", screenGui)
alertButton.Position = UDim2.new(0.5, -25, 0, 170)
alertButton.Size = UDim2.new(0, 50, 0, 25)
alertButton.Text = ""
alertButton.BackgroundColor3 = Color3.new(1, 0, 0) -- Red
alertButton.Visible = false -- Initially hidden

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

-- Function to throw the Bomb at the closest player
local function boomThrow()
    if isThrowEnabled and hasBomb then
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
            -- Save the current position before throwing
            originalPosition = humanoidRootPart.CFrame -- Save the player's current position before throwing

            -- Move to closest player's position and throw
            humanoidRootPart.CFrame = CFrame.new(closestPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))

            -- Adjust the actual throwing logic here
            local throwEvent = game.ReplicatedStorage:FindFirstChild("ThrowEvent") -- Make sure the event is correct
            if throwEvent then
                throwEvent:FireServer(closestPlayer.Character) -- Fire the event towards the closest player
            end

            -- Show alert only if the player has the Bomb
            if hasBomb then
                alertButton.Text = "ဗုံးရနေပါပြီ!" -- Alert message
                alertButton.Visible = true -- Show alert

                -- Hide the alert after the Bomb is thrown
                wait(2) -- Wait for 2 seconds before hiding the alert
                alertButton.Visible = false -- Hide the alert
            end

            wait(0.5) -- Small delay after the throw
            humanoidRootPart.CFrame = originalPosition -- Return to original position
        end
    end
end

-- Enable Button functionality
enableButton.MouseButton1Click:Connect(function()
    isThrowEnabled = true
    enableButton.Visible = false
    disableButton.Visible = true
    throwButton.Visible = true
end)

-- Disable Button functionality
disableButton.MouseButton1Click:Connect(function()
    isThrowEnabled = false
    enableButton.Visible = true
    disableButton.Visible = false
    throwButton.Visible = false
end)

-- Throw Button functionality
throwButton.MouseButton1Click:Connect(function()
    if isThrowEnabled then
        boomThrow() -- Call the throw function when the button is clicked
    end
end)

-- Function to detect Bomb being added to player's inventory
player.Backpack.ChildAdded:Connect(function(child)
    if child.Name == "Bomb" then -- Change "Bomb" to the actual name of the Bomb Object
        hasBomb = true -- Set the flag to true when Bomb is received
        alertButton.Text = "ဗုံးရနေပါပြီ!" -- Show alert message
        alertButton.Visible = true -- Show the alert button
    end
end)

-- Function to detect Bomb being removed from player's inventory
player.Backpack.ChildRemoved:Connect(function(child)
    if child.Name == "Bomb" then -- Change "Bomb" to the actual name of the Bomb Object
        hasBomb = false -- Set the flag to false when Bomb is removed
        alertButton.Visible = false -- Hide the alert if Bomb is removed
    end
end)
