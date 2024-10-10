-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local isThrowEnabled = false
local originalPosition = nil

-- UI setup
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
local enableButton = Instance.new("TextButton", screenGui)
local disableButton = Instance.new("TextButton", screenGui)
local throwButton = Instance.new("TextButton", screenGui)

-- Enable Button setup (smaller size)
enableButton.Position = UDim2.new(0.5, -25, 0, 50)
enableButton.Size = UDim2.new(0, 50, 0, 25) -- Smaller size (50x25)
enableButton.Text = "ဖွင့်မည်"
enableButton.BackgroundColor3 = Color3.new(0, 1, 0) -- Green
enableButton.Draggable = true

-- Disable Button setup (smaller size)
disableButton.Position = UDim2.new(0.5, -25, 0, 90)
disableButton.Size = UDim2.new(0, 50, 0, 25) -- Smaller size (50x25)
disableButton.Text = "ပိတ်မည်"
disableButton.BackgroundColor3 = Color3.new(1, 0, 0) -- Red
disableButton.Draggable = true
disableButton.Visible = false -- Hide the disable button initially

-- Throw Button setup (smaller size)
throwButton.Position = UDim2.new(0.5, -25, 0, 130)
throwButton.Size = UDim2.new(0, 50, 0, 25) -- Smaller size (50x25)
throwButton.Text = "ပစ်မည်"
throwButton.BackgroundColor3 = Color3.new(1, 1, 0) -- Yellow
throwButton.Draggable = true
throwButton.Visible = false -- Hide initially, show only when enabled

-- Bomb Alert Button setup (for displaying "ဗုံးရနေပီ")
local bombAlertButton = Instance.new("TextButton", screenGui)
bombAlertButton.Position = UDim2.new(0.5, -75, 0, 170)
bombAlertButton.Size = UDim2.new(0, 150, 0, 50) -- Button size
bombAlertButton.Text = "ဗုံးမရသေး"
bombAlertButton.BackgroundColor3 = Color3.new(1, 1, 1) -- White (default)
bombAlertButton.Visible = false -- Hidden until bomb is received

-- Function to change bomb alert button colors
local function updateBombButtonColors()
    while isThrowEnabled do
        bombAlertButton.BackgroundColor3 = Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255))
        wait(0.5) -- Change color every 0.5 seconds
    end
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

-- Function to throw the "Boom" at the closest player and return to original position
local function boomThrow()
    if isThrowEnabled then
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

            -- After throwing, return to the original position
            wait(0.5) -- Small delay after the throw
            humanoidRootPart.CFrame = originalPosition -- Return to original position

            -- Update bomb alert
            bombAlertButton.Text = "ဗုံးရနေပီ"
            bombAlertButton.Visible = true
            spawn(updateBombButtonColors) -- Start the color changing
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
    bombAlertButton.Visible = false -- Hide the bomb alert button
end)

-- Throw Button functionality
throwButton.MouseButton1Click:Connect(function()
    if isThrowEnabled then
        boomThrow() -- Call the throw function when the button is clicked
    end
end)
