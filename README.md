-- Quantum Services
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

-- Quantum Variables
local player = Players.LocalPlayer
local balls = {}
local lastRefreshTime = os.time()
local reach = 10
local magnetStrength = 0
local reachCircle = nil

-- Function to refresh the list of balls in the workspace
local function refreshBalls(force)
    if not force and lastRefreshTime + 2 > os.time() then 
        print("Quantum refresh too early")
        return 
    end
    lastRefreshTime = os.time()
    table.clear(balls)
    for _, v in pairs(Workspace:GetDescendants()) do
        if v.Name == "TPS" or v.Name == "ESA" or v.Name == "MRS" or v.Name == "PRS" or v.Name == "MPS" then
            table.insert(balls, v)
        end
    end
end

-- Initial refresh of balls
refreshBalls(true)

-- Function to create GUI for reach adjustment
local function createReachGUI()
    local screenGui = Instance.new("ScreenGui", player.PlayerGui)
    screenGui.Name = "QuantumReachGUI"
    
    local frame = Instance.new("Frame", screenGui)
    frame.Size = UDim2.new(0.2, 0, 0.1, 0)
    frame.Position = UDim2.new(0.01, 0, 0.01, 0)
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BackgroundTransparency = 0.5
    frame.BorderSizePixel = 0

    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0.1, 0)

    local textLabel = Instance.new("TextLabel", frame)
    textLabel.Size = UDim2.new(1, 0, 0.5, 0)
    textLabel.Text = "Quantum Reach: " .. reach
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextScaled = true
    textLabel.TextColor3 = Color3.new(1, 1, 1)
    textLabel.BackgroundTransparency = 1

    local decrementButton = Instance.new("TextButton", frame)
    decrementButton.Size = UDim2.new(0.5, 0, 0.5, 0)
    decrementButton.Position = UDim2.new(0, 0, 0.5, 0)
    decrementButton.Text = "-"
    decrementButton.Font = Enum.Font.SourceSansBold
    decrementButton.TextScaled = true
    decrementButton.TextColor3 = Color3.new(1, 1, 1)
    decrementButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    
    local incrementButton = Instance.new("TextButton", frame)
    incrementButton.Size = UDim2.new(0.5, 0, 0.5, 0)
    incrementButton.Position = UDim2.new(0.5, 0, 0.5, 0)
    incrementButton.Text = "+"
    incrementButton.Font = Enum.Font.SourceSansBold
    incrementButton.TextScaled = true
    incrementButton.TextColor3 = Color3.new(1, 1, 1)
    incrementButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)

    decrementButton.MouseButton1Click:Connect(function()
        reach = math.max(1, reach - 1) -- Ensure reach doesn't go below 1
        textLabel.Text = "Quantum Reach: " .. reach
        createReachCircle()
    end)

    incrementButton.MouseButton1Click:Connect(function()
        reach = reach + 1
        textLabel.Text = "Quantum Reach: " .. reach
        createReachCircle()
    end)
end

-- Function to create a visible reach circle
local function createReachCircle()
    if reachCircle then
        reachCircle.Size = Vector3.new(reach * 2, reach * 2, reach * 2)
    else
        reachCircle = Instance.new("Part", Workspace)
        reachCircle.Shape = Enum.PartType.Ball
        reachCircle.Size = Vector3.new(reach * 2, reach * 2, reach * 2)
        reachCircle.Anchored = true
        reachCircle.CanCollide = false
        reachCircle.Transparency = 0.8
        reachCircle.Material = Enum.Material.ForceField
        reachCircle.Color = Color3.new(0, 0, 1)

        RunService.RenderStepped:Connect(function()
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                reachCircle.Position = player.Character.HumanoidRootPart.Position
            end
        end)
    end
end

-- Function to handle quantum input
local function onQuantumInputBegan(input, gameProcessedEvent)
    -- Ignore certain key inputs
    local ignoredKeys = {
        [Enum.KeyCode.W] = true,
        [Enum.KeyCode.A] = true,
        [Enum.KeyCode.S] = true,
        [Enum.KeyCode.D] = true,
        [Enum.KeyCode.Space] = true
    }

    if ignoredKeys[input.KeyCode] then
        return
    end

    if not gameProcessedEvent then
        if input.KeyCode == Enum.KeyCode.Comma then
            reach = math.max(1, reach - 1) -- Ensure reach doesn't go below 1
            StarterGui:SetCore("SendNotification", {
                Title = "QuantumReach",
                Text = "Quantum reach set to: " .. reach,
                Duration = 0.5,
            })
            createReachCircle()
        elseif input.KeyCode == Enum.KeyCode.Period then
            reach = reach + 1
            StarterGui:SetCore("SendNotification", {
                Title = "QuantumReach",
                Text = "Quantum reach set to: " .. reach,
                Duration = 0.5,
            })
            createReachCircle()
        else
            refreshBalls(false)
            for _, v in pairs(player.Character["Right Leg"]:GetDescendants()) do
                if v.Name == "TouchInterest" and v.Parent then
                    for _, e in pairs(balls) do
                        if (e.Position - player.Character["Right Leg"].Position).magnitude < reach then
                            firetouchinterest(e, v.Parent, 0)
                            firetouchinterest(e, v.Parent, 1)
                            break
                        end
                    end
                end
            end
        end
    end
end

-- Connect the quantum input began event to the handler function
UserInputService.InputBegan:Connect(onQuantumInputBegan)

-- Function to apply a subtle magnetic effect to the ball
local function applyMagneticEffect()
    RunService.RenderStepped:Connect(function()
        for _, ball in pairs(balls) do
            if ball and ball.Parent and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (ball.Position - player.Character.HumanoidRootPart.Position).magnitude
                if distance < reach then
                    local direction = (player.Character.HumanoidRootPart.Position - ball.Position).unit
                    ball.Velocity = ball.Velocity + direction * magnetStrength
                end
            end
        end
    end)
end

-- Function to ensure a second touch to the ball for better interaction
local function ensureSecondTouch()
    RunService.RenderStepped:Connect(function()
        for _, v in pairs(player.Character["Right Leg"]:GetDescendants()) do
            if v.Name == "TouchInterest" and v.Parent then
                for _, e in pairs(balls) do
                    if (e.Position - player.Character["Right Leg"].Position).magnitude < reach then
                        firetouchinterest(e, v.Parent, 0)
                        firetouchinterest(e, v.Parent, 1)
                        break
                    end
                end
            end
        end
    end)
end

-- Create initial reach circle and GUI
createReachCircle()
createReachGUI()
applyMagneticEffect()
ensureSecondTouch()

print("=====================")
print("script by nikky")
print("=====================")
