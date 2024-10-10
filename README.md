-- LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local isThrowEnabled = false

-- UI setup
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
local enableButton = Instance.new("TextButton", screenGui)
local disableButton = Instance.new("TextButton", screenGui)
local throwButton = Instance.new("TextButton", screenGui)

-- Enable Button setup
enableButton.Position = UDim2.new(0.5, -50, 0, 50)
enableButton.Size = UDim2.new(0, 100, 0, 50)
enableButton.Text = "ဖွင့်မည်"
enableButton.BackgroundColor3 = Color3.new(0, 1, 0) -- Green
enableButton.Draggable = true

-- Disable Button setup
disableButton.Position = UDim2.new(0.5, -50, 0, 110)
disableButton.Size = UDim2.new(0, 100, 0, 50)
disableButton.Text = "ပိတ်မည်"
disableButton.BackgroundColor3 = Color3.new(1, 0, 0) -- Red
disableButton.Draggable = true
disableButton.Visible = false -- Hide the disable button initially

-- Throw Button setup
throwButton.Position = UDim2.new(0.5, -50, 0, 170)
throwButton.Size = UDim2.new(0, 100, 0, 50)
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
            print("Distance to " .. otherPlayer.Name .. ": " .. distance) -- Debugging purpose
            if distance < minDistance then
                minDistance = distance
                closestPlayer = otherPlayer
            end
        end
    end
    print("Closest Player: " .. (closestPlayer and closestPlayer.Name or "None"))
    return closestPlayer
end

-- Function to throw the "Boom" at the closest player
local function boomThrow()
    if isThrowEnabled then
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
            -- Move to the closest player's position and throw boom
            print("Moving to " .. closestPlayer.Name .. "'s position")
            humanoidRootPart.CFrame = CFrame.new(closestPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))

            -- Adjust the actual throwing logic here
            local throwEvent = game.ReplicatedStorage:FindFirstChild("ThrowEvent") -- Make sure the event is correct
            if throwEvent then
                throwEvent:FireServer(closestPlayer.Character) -- Fire the event towards the closest player
                print("Throw event fired at " .. closestPlayer.Name)
            else
                print("Throw event not found in ReplicatedStorage")
            end
        else
            print("No closest player found to throw!")
        end
    end
end

-- Enable Button functionality
enableButton.MouseButton1Click:Connect(function()
    isThrowEnabled = true
    enableButton.Visible = false
    disableButton.Visible = true
    throwButton.Visible = true
    print("Throwing enabled")
end)

-- Disable Button functionality
disableButton.MouseButton1Click:Connect(function()
    isThrowEnabled = false
    enableButton.Visible = true
    disableButton.Visible = false
    throwButton.Visible = false
    print("Throwing disabled")
end)

-- Throw Button functionality
throwButton.MouseButton1Click:Connect(function()
    if isThrowEnabled then
        boomThrow() -- Call the throw function when the button is clicked
    else
        print("Throw is disabled!")
    end
end)
