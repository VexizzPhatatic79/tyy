-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local isThrowEnabled = false
local originalPosition = humanoidRootPart.CFrame -- Store the original position

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
            humanoidRootPart.CFrame = CFrame.new(closestPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))

            -- Adjust the actual throwing logic here
            local throwEvent = game.ReplicatedStorage:FindFirstChild("ThrowEvent") -- Make sure the event is correct
            if throwEvent then
                throwEvent:FireServer(closestPlayer.Character) -- Fire the event towards the closest player
            end

            -- After throwing, return to the original position
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
    originalPosition = humanoidRootPart.CFrame -- Save original position when enabled
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
