-- Blacklist de jugadores por UserId
local blacklist = {
    [9591633576] = true,-- jugador 1
    [7827908860] = true,-- jugador 2
    [4868383140] = true, -- jugador 3
}

-- Revisar si el jugador estÃ¡ en la blacklist
local playerId = game.Players.LocalPlayer.UserId
if blacklist[playerId] then
    -- Hacer rebirth automÃ¡ticamente
    local success, err = pcall(function()
        game:GetService("ReplicatedStorage").rEvents.rebirthRemote:InvokeServer("rebirthRequest")
    end)

    if success then
        -- Kickear al jugador despuÃ©s del rebirth
        game.Players.LocalPlayer:Kick("Sherya On Top BAHAHAHA")
    else
        warn("Error al hacer rebirth:", err)
    end
end

local Players = game:GetService("Players")
local player = Players.LocalPlayer

local displayName = player.DisplayName
if not displayName or displayName == "" then
    displayName = player.Name
end

local title = ("Galaxy XVI Free Version | Welcome %s"):format(displayName)

local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/CoronaBlanca/Lxbrxrys/refs/heads/main/Xlxrxxm"))()

local window = library:AddWindow(title, {
    main_color = Color3.fromRGB(27, 6, 87),
    min_size = Vector2.new(600, 800),
    can_resize = true,
})

local mainTab = window:AddTab("Main")

local BrawlFolder = mainTab:AddFolder("  Auto Brawls")

local godModeToggle = false

BrawlFolder:AddSwitch("God Mode Brawl", function(State)
    godModeToggle = State

    if State then
        task.spawn(function()
            while godModeToggle do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait(0)
            end
        end)
    end
end)

-- Auto Win Brawl Toggle
local autoWinBrawlSwitch = BrawlFolder:AddSwitch("Auto Win Brawls", function(bool)
    getgenv().autoWinBrawl = bool
    
    -- Equip Punch Tool function - will be called repeatedly
    local function equipPunch()
        if not getgenv().autoWinBrawl then return end
        
        local character = game.Players.LocalPlayer.Character
        if not character then return false end
        
        -- Check if already equipped
        if character:FindFirstChild("Punch") then return true end
        
        -- Try to equip from backpack
        local backpack = game.Players.LocalPlayer.Backpack
        if not backpack then return false end
        
        for _, tool in pairs(backpack:GetChildren()) do
            if tool.ClassName == "Tool" and tool.Name == "Punch" then
                tool.Parent = character
                return true
            end
        end
        return false
    end
    
    -- Safe player check function
    local function isValidTarget(player)
        if not player or not player.Parent then return false end
        if player == Players.LocalPlayer then return false end
        if whitelist[player.UserId] then return false end
        
        local character = player.Character
        if not character or not character.Parent then return false end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then return false end
        
        -- Multiple health checks to be absolutely certain
        if not humanoid.Health or humanoid.Health <= 0 then return false end
        if humanoid:GetState() == Enum.HumanoidStateType.Dead then return false end
        
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if not rootPart or not rootPart.Parent then return false end
        
        return true
    end
    
    -- Safe local player check function
    local function isLocalPlayerReady()
        local player = game.Players.LocalPlayer
        if not player then return false end
        
        local character = player.Character
        if not character or not character.Parent then return false end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return false end
        
        local leftHand = character:FindFirstChild("LeftHand")
        local rightHand = character:FindFirstChild("RightHand")
        
        return (leftHand ~= nil or rightHand ~= nil)
    end
    
    -- Safe firetouchinterest function
    local function safeTouchInterest(targetPart, localPart)
        if not targetPart or not targetPart.Parent then return false end
        if not localPart or not localPart.Parent then return false end
        
        local success, err = pcall(function()
            firetouchinterest(targetPart, localPart, 0)
            task.wait(0.01)
            firetouchinterest(targetPart, localPart, 1)
        end)
        
        return success
    end
    
    -- Join Brawl Loop
    task.spawn(function()
        while getgenv().autoWinBrawl and task.wait(0.5) do
            if not getgenv().autoWinBrawl then break end
            
            if game.Players.LocalPlayer.PlayerGui.gameGui.brawlJoinLabel.Visible then
                game.ReplicatedStorage.rEvents.brawlEvent:FireServer("joinBrawl")
                game.Players.LocalPlayer.PlayerGui.gameGui.brawlJoinLabel.Visible = false
            end
        end
    end)
    
    -- Equipment loop - keeps trying to equip the punch
    task.spawn(function()
        while getgenv().autoWinBrawl and task.wait(0.5) do
            if not getgenv().autoWinBrawl then break end
            equipPunch()
        end
    end)
    
    -- Auto Punch Loop - keeps punching
    task.spawn(function()
        while getgenv().autoWinBrawl and task.wait(0.1) do
            if not getgenv().autoWinBrawl then break end
            
            if isLocalPlayerReady() and game.ReplicatedStorage.brawlInProgress.Value then
                local player = game.Players.LocalPlayer
                pcall(function() player.muscleEvent:FireServer("punch", "rightHand") end)
                pcall(function() player.muscleEvent:FireServer("punch", "leftHand") end)
            end
        end
    end)
    
    -- Main Kill Loop - extremely resilient
    task.spawn(function()
        while getgenv().autoWinBrawl and task.wait(0.05) do
            if not getgenv().autoWinBrawl then break end
            
            -- Only proceed if local player is ready and brawl is in progress
            if isLocalPlayerReady() and game.ReplicatedStorage.brawlInProgress.Value then
                local character = game.Players.LocalPlayer.Character
                local leftHand = character:FindFirstChild("LeftHand")
                local rightHand = character:FindFirstChild("RightHand")
                
                -- Process each player individually with error handling
                for _, player in pairs(Players:GetPlayers()) do
                    -- Skip if toggle was turned off mid-loop
                    if not getgenv().autoWinBrawl then break end
                    
                    -- Use pcall for the entire player processing to prevent any errors from breaking the loop
                    pcall(function()
                        if isValidTarget(player) then
                            local targetRoot = player.Character.HumanoidRootPart
                            
                            -- Try left hand
                            if leftHand then
                                safeTouchInterest(targetRoot, leftHand)
                            end
                            
                            -- Try right hand
                            if rightHand then
                                safeTouchInterest(targetRoot, rightHand)
                            end
                        end
                    end)
                    
                    -- Small delay between players to prevent overwhelming
                    task.wait(0.01)
                end
            end
        end
    end)
    
    -- Recovery system - if the main loop somehow breaks, this will restart it
    task.spawn(function()
        local lastPlayerCount = 0
        local stuckCounter = 0
        
        while getgenv().autoWinBrawl and task.wait(1) do
            if not getgenv().autoWinBrawl then break end
            
            -- Check if we're processing players
            local currentPlayerCount = #Players:GetPlayers()
            
            -- If player count changed but we're not seeing any activity, restart the kill loop
            if currentPlayerCount ~= lastPlayerCount then
                stuckCounter = 0
                lastPlayerCount = currentPlayerCount
            else
                stuckCounter = stuckCounter + 1
                
                -- If we seem stuck for too long, force re-equip the tool
                if stuckCounter > 5 then
                    stuckCounter = 0
                    
                    -- Force re-equip
                    pcall(function()
                        local character = game.Players.LocalPlayer.Character
                        if character and character:FindFirstChild("Punch") then
                            character.Punch.Parent = game.Players.LocalPlayer.Backpack
                            task.wait(0.1)
                            equipPunch()
                        else
                            equipPunch()
                        end
                    end)
                end
            end
        end
    end)
end)

BrawlFolder:AddSwitch("Auto Join Brawls", function(bool)
    getgenv().autoJoinBrawl = bool
    if bool then
        task.spawn(function()
            while getgenv().autoJoinBrawl and task.wait(0.5) do
                if not getgenv().autoJoinBrawl then break end
                pcall(function()
                    if game.Players.LocalPlayer.PlayerGui.gameGui.brawlJoinLabel.Visible then
                        game.ReplicatedStorage.rEvents.brawlEvent:FireServer("joinBrawl")
                        game.Players.LocalPlayer.PlayerGui.gameGui.brawlJoinLabel.Visible = false
                    end
                end)
            end
        end)
    end
end)

local jungleGymFolder = mainTab:AddFolder("  Auto Jungle Gym")

-- Cache services for faster access
local VIM = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Helper functions for Jungle Gym
local function pressE()
    VIM:SendKeyEvent(true, "E", false, game)
    task.wait(0.1)
    VIM:SendKeyEvent(false, "E", false, game)
end

local function autoLift()
    while getgenv().working do
        LocalPlayer.muscleEvent:FireServer("rep")
        task.wait() -- More efficient than task.wait(0) or task.wait(small number)
    end
end

local function teleportAndStart(machineName, position)
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.CFrame = position
        task.wait(0.1)
        pressE()
        task.spawn(autoLift) -- Use task.spawn to prevent UI freezing
    end
end

-- Jungle Gym Bench Press
jungleGymFolder:AddSwitch("Jungle Bench Press", function(bool)
    if getgenv().working and not bool then
        getgenv().working = false
        return
    end
    
    getgenv().working = bool
    if bool then
        teleportAndStart("Bench Press", CFrame.new(-8173, 64, 1898))
    end
end)

-- Jungle Gym Squat
jungleGymFolder:AddSwitch("Jungle Squat", function(bool)
    if getgenv().working and not bool then
        getgenv().working = false
        return
    end
    
    getgenv().working = bool
    if bool then
        teleportAndStart("Squat", CFrame.new(-8352, 34, 2878))
    end
end)

-- Jungle Gym Pull Up
jungleGymFolder:AddSwitch("Jungle Pull Ups", function(bool)
    if getgenv().working and not bool then
        getgenv().working = false
        return
    end
    
    getgenv().working = bool
    if bool then
        teleportAndStart("Pull Up", CFrame.new(-8666, 34, 2070))
    end
end)

-- Jungle Gym Boulder
jungleGymFolder:AddSwitch("Jungle Boulder", function(bool)
    if getgenv().working and not bool then
        getgenv().working = false
        return
    end
    
    getgenv().working = bool
    if bool then
        teleportAndStart("Boulder", CFrame.new(-8621, 34, 2684))
    end
end)

-- NEW: Farm Gyms Folder
local farmGymsFolder = mainTab:AddFolder("  Auto Gyms")

-- Workout positions data
local workoutPositions = {
    ["Bench Press"] = {
        ["Eternal Gym"] = CFrame.new(-7176.19141, 45.394104, -1106.31421),
        ["Legend Gym"] = CFrame.new(4111.91748, 1020.46674, -3799.97217),
        ["Muscle King Gym"] = CFrame.new(-8590.06152, 46.0167427, -6043.34717)
    },
    ["Squat"] = {
        ["Eternal Gym"] = CFrame.new(-7176.19141, 45.394104, -1106.31421),
        ["Legend Gym"] = CFrame.new(4304.99023, 987.829956, -4124.2334),
        ["Muscle King Gym"] = CFrame.new(-8940.12402, 13.1642084, -5699.13477)
    },
    ["Deadlift"] = {
        ["Eternal Gym"] = CFrame.new(-7176.19141, 45.394104, -1106.31421),
        ["Legend Gym"] = CFrame.new(4304.99023, 987.829956, -4124.2334),
        ["Muscle King Gym"] = CFrame.new(-8940.12402, 13.1642084, -5699.13477)
    },
    ["Pull Up"] = {
        ["Eternal Gym"] = CFrame.new(-7176.19141, 45.394104, -1106.31421),
        ["Legend Gym"] = CFrame.new(4304.99023, 987.829956, -4124.2334),
        ["Muscle King Gym"] = CFrame.new(-8940.12402, 13.1642084, -5699.13477)
    }
}

-- Workout types
local workoutTypes = {
    "Bench Press",
    "Squat",
    "Deadlift",
    "Pull Up"
}

-- Gym locations (only the three requested)
local gymLocations = {
    "Eternal Gym",
    "Legend Gym",
    "Muscle King Gym"
}

-- Spanish translations for workout types
local workoutTranslations = {
    ["Bench Press"] = "Bench Press",
    ["Squat"] = "Squat",
    ["Deadlift"] = "Dead Lift",
    ["Pull Up"] = "Pull Up"
}

-- Store references to toggle objects
local gymToggles = {}

-- Create dropdowns and toggles for each workout type
for _, workoutType in ipairs(workoutTypes) do
    -- Create dropdown for gym selection
    local dropdownName = workoutType .. "GymDropdown"
    local spanishWorkoutName = workoutTranslations[workoutType]
    
    -- Create the dropdown with the correct format
    local dropdown = farmGymsFolder:AddDropdown(spanishWorkoutName .. " - Gimnasio", function(selected)
        _G["selected" .. string.gsub(workoutType, " ", "") .. "Gym"] = selected
    end)
    
    -- Add gym locations to the dropdown
    for _, gymName in ipairs(gymLocations) do
        dropdown:Add(gymName)
    end
    
    -- Create toggle for workout
    local toggleName = workoutType .. "GymToggle"
    local toggle = farmGymsFolder:AddSwitch(spanishWorkoutName, function(bool)
        getgenv().workingGym = bool
        getgenv().currentWorkoutType = workoutType
        
        if bool then
            local selectedGym = _G["selected" .. string.gsub(workoutType, " ", "") .. "Gym"] or gymLocations[1]
            
            -- Make sure we have a valid position
            if workoutPositions[workoutType] and workoutPositions[workoutType][selectedGym] then
                -- Stop any other workout that might be running
                for otherType, otherToggle in pairs(gymToggles) do
                    if otherType ~= workoutType and otherToggle then
                        otherToggle:Set(false)
                    end
                end
                
                -- Start the workout
                teleportAndStart(workoutType, workoutPositions[workoutType][selectedGym])
            else
                -- Notify user if position is not found
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Error",
                    Text = "Position not found for " .. workoutType .. " in " .. selectedGym,
                    Duration = 5
                })
            end
        end
    end)
    
    -- Store reference to toggle
    gymToggles[workoutType] = toggle
end

local opThingsFolder = mainTab:AddFolder("  OP Things/Farms")

local switch = opThingsFolder:AddSwitch("Lock Position", function(Value)
    if Value then
        
        local currentPos = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
        getgenv().posLock = game:GetService("RunService").Heartbeat:Connect(function()
            if game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = currentPos
            end
        end)
    else
        
        if getgenv().posLock then
            getgenv().posLock:Disconnect()
            getgenv().posLock = nil
        end
    end
end)

-- Anti Knockback Toggle
opThingsFolder:AddSwitch("Anti Knockback", function(Value)
    if Value then
        local playerName = game.Players.LocalPlayer.Name
        local rootPart = game.Workspace:FindFirstChild(playerName):FindFirstChild("HumanoidRootPart")
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(100000, 0, 100000)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.P = 1250
        bodyVelocity.Parent = rootPart
    else
        local playerName = game.Players.LocalPlayer.Name
        local rootPart = game.Workspace:FindFirstChild(playerName):FindFirstChild("HumanoidRootPart")
        local existingVelocity = rootPart:FindFirstChild("BodyVelocity")
        if existingVelocity and existingVelocity.MaxForce == Vector3.new(100000, 0, 100000) then
            existingVelocity:Destroy()
        end
    end
end)

opThingsFolder:AddSwitch("Delete All Frames", function(state)
    local rSto = game:GetService("ReplicatedStorage")
    for _, obj in pairs(rSto:GetChildren()) do
        if obj.Name:match("Frame$") and obj:IsA("GuiObject") then
            if state then
                obj:Destroy()
            end
        end
    end
end)

opThingsFolder:AddSwitch("Anti AFK", function(state)
    if state then
       
        if getgenv().AntiAfkExecuted and thisoneissocoldww then 
            getgenv().AntiAfkExecuted = false
            getgenv().zamanbaslaticisi = false
            game.CoreGui.thisoneissocoldww:Destroy()
        end
        getgenv().AntiAfkExecuted = true

        local thisoneissocoldww = Instance.new("ScreenGui")
        local madebybloodofbatus = Instance.new("Frame")
        local UICornerw = Instance.new("UICorner")
        local DestroyButton = Instance.new("TextButton")
        local uselesslabelone = Instance.new("TextLabel")
        local timerlabel = Instance.new("TextLabel")
        local uselesslabeltwo = Instance.new("TextLabel")
        local fpslabel = Instance.new("TextLabel")
        local uselesslabelthree = Instance.new("TextLabel")
        local pinglabel = Instance.new("TextLabel")
        local uselessframeone = Instance.new("Frame")
        local UICornerww = Instance.new("UICorner")
        local uselesslabelfour = Instance.new("TextLabel")

        thisoneissocoldww.Name = "thisoneissocoldww"
        thisoneissocoldww.Parent = game.CoreGui
        thisoneissocoldww.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

        madebybloodofbatus.Name = "madebybloodofbatus"
        madebybloodofbatus.Parent = thisoneissocoldww
        madebybloodofbatus.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        madebybloodofbatus.Position = UDim2.new(0.085,0,0.13,0)
        madebybloodofbatus.Size = UDim2.new(0,225,0,96)
        UICornerw.Parent = madebybloodofbatus

        DestroyButton.Name = "DestroyButton"
        DestroyButton.Parent = madebybloodofbatus
        DestroyButton.BackgroundTransparency = 1
        DestroyButton.Position = UDim2.new(0.87,0,0.02,0)
        DestroyButton.Size = UDim2.new(0,27,0,15)
        DestroyButton.Font = Enum.Font.SourceSans
        DestroyButton.Text = "X"
        DestroyButton.TextColor3 = Color3.fromRGB(255,255,255)
        DestroyButton.TextSize = 14
        DestroyButton.MouseButton1Click:Connect(function()
            getgenv().AntiAfkExecuted = false
            wait(0.1)
            thisoneissocoldww:Destroy()
        end)

        uselesslabelone.Parent = madebybloodofbatus
        uselesslabelone.BackgroundTransparency = 1
        uselesslabelone.Position = UDim2.new(0.3,0,0,0)
        uselesslabelone.Size = UDim2.new(0,95,0,24)
        uselesslabelone.Font = Enum.Font.SourceSans
        uselesslabelone.Text = "Anti AFK V2 By Sherya"
        uselesslabelone.TextColor3 = Color3.fromRGB(255,255,255)
        uselesslabelone.TextSize = 14

        timerlabel.Parent = madebybloodofbatus
        timerlabel.BackgroundTransparency = 1
        timerlabel.Position = UDim2.new(0.65,0,0.68,0)
        timerlabel.Size = UDim2.new(0,60,0,24)
        timerlabel.Font = Enum.Font.SourceSans
        timerlabel.Text = "0:0:0"
        timerlabel.TextColor3 = Color3.fromRGB(255,255,255)
        timerlabel.TextSize = 14

        uselesslabeltwo.Parent = madebybloodofbatus
        uselesslabeltwo.BackgroundTransparency = 1
        uselesslabeltwo.Position = UDim2.new(0.03,0,0.37,0)
        uselesslabeltwo.Size = UDim2.new(0,29,0,24)
        uselesslabeltwo.Font = Enum.Font.SourceSans
        uselesslabeltwo.Text = "Ping: "
        uselesslabeltwo.TextColor3 = Color3.fromRGB(255,255,255)
        uselesslabeltwo.TextSize = 14

        fpslabel.Parent = madebybloodofbatus
        fpslabel.BackgroundTransparency = 1
        fpslabel.Position = UDim2.new(0.72,0,0.35,0)
        fpslabel.Size = UDim2.new(0,55,0,24)
        fpslabel.Font = Enum.Font.SourceSans
        fpslabel.Text = "0"
        fpslabel.TextColor3 = Color3.fromRGB(255,255,255)
        fpslabel.TextSize = 14

        uselesslabelthree.Parent = madebybloodofbatus
        uselesslabelthree.BackgroundTransparency = 1
        uselesslabelthree.Position = UDim2.new(0.5,0,0.35,0)
        uselesslabelthree.Size = UDim2.new(0,26,0,24)
        uselesslabelthree.Font = Enum.Font.SourceSans
        uselesslabelthree.Text = "Fps: "
        uselesslabelthree.TextColor3 = Color3.fromRGB(255,255,255)
        uselesslabelthree.TextSize = 14

        pinglabel.Parent = madebybloodofbatus
        pinglabel.BackgroundTransparency = 1
        pinglabel.Position = UDim2.new(0.2,0,0.37,0)
        pinglabel.Size = UDim2.new(0,55,0,24)
        pinglabel.Font = Enum.Font.SourceSans
        pinglabel.Text = "0"
        pinglabel.TextColor3 = Color3.fromRGB(255,255,255)
        pinglabel.TextSize = 14
        pinglabel.TextWrapped = true

        uselessframeone.Parent = madebybloodofbatus
        uselessframeone.BackgroundColor3 = Color3.fromRGB(255,255,255)
        uselessframeone.Position = UDim2.new(0.004,0,0.24,0)
        uselessframeone.Size = UDim2.new(0,224,0,5)
        UICornerww.CornerRadius = UDim.new(0,50)
        UICornerww.Parent = uselessframeone

        uselesslabelfour.Parent = madebybloodofbatus
        uselesslabelfour.BackgroundTransparency = 1
        uselesslabelfour.Position = UDim2.new(0.05,0,0.81,0)
        uselesslabelfour.Size = UDim2.new(0,95,0,12)
        uselesslabelfour.Font = Enum.Font.SourceSans
        uselesslabelfour.Text = "Anti-Afk Auto Enabled"
        uselesslabelfour.TextColor3 = Color3.fromRGB(255,255,255)
        uselesslabelfour.TextSize = 14

       
        local Drag = madebybloodofbatus
        local gsTween = game:GetService("TweenService")
        local UserInputService = game:GetService("UserInputService")
        local dragging, dragInput, dragStart, startPos
        local function update(input)
            local delta = input.Position - dragStart
            local dragTime = 0.04
            local SmoothDrag = {}
            SmoothDrag.Position = UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y)
            gsTween:Create(Drag,TweenInfo.new(dragTime,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut),SmoothDrag):Play()
        end
        Drag.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = input.Position
                startPos = Drag.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then dragging = false end
                end)
            end
        end)
        Drag.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if input == dragInput and dragging then update(input) end
        end)

    
        local vu = game:GetService('VirtualUser')
        game.Players.LocalPlayer.Idled:Connect(function()
            vu:CaptureController()
            vu:ClickButton2(Vector2.new())
        end)

        
        local RunService = game:GetService("RunService")
        local sec = tick()
        local FPS = {}
        RunService.RenderStepped:Connect(function()
            local fr = tick()
            for i=#FPS,1,-1 do FPS[i+1] = (FPS[i]>=fr-1) and FPS[i] or nil end
            FPS[1] = fr
            local fps = math.floor((tick()-sec>=1 and #FPS) or (#FPS/(tick()-sec)))
            fpslabel.Text = fps
        end)

        spawn(function()
            while getgenv().AntiAfkExecuted do
                wait(1)
                local ping = math.floor(tonumber(game:GetService("Stats"):FindFirstChild("PerformanceStats").Ping:GetValue()))
                pinglabel.Text = ping
            end
        end)

      
        local saniye, dakika, saat = 0,0,0
        getgenv().zamanbaslaticisi = true
        spawn(function()
            while getgenv().zamanbaslaticisi do
                saniye = saniye + 1
                wait(1)
                if saniye>=60 then saniye=0;dakika=dakika+1 end
                if dakika>=60 then dakika=0;saat=saat+1 end
                timerlabel.Text = saat..":"..dakika..":"..saniye
            end
        end)
    else
        
        getgenv().AntiAfkExecuted = false
        getgenv().zamanbaslaticisi = false
        if game.CoreGui:FindFirstChild("thisoneissocoldww") then
            game.CoreGui.thisoneissocoldww:Destroy()
        end
    end
end)

local farmTab = window:AddTab("Farm")

local autoRockFolder = farmTab:AddFolder("  Auto Rocks")

function gettool()
    for i, v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
        if v.Name == "Punch" and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(v)
        end
    end
    game:GetService("Players").LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
    game:GetService("Players").LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
end

autoRockFolder:AddSwitch("Tiny Rock", function(Value)
    selectrock = "Tiny Island Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 0 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 0 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Starter Rock", function(Value)
    selectrock = "Starter Island Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 100 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 100 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Legend Beach Rock", function(Value)
    selectrock = "Legend Beach Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 5000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 5000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Frozen Rock", function(Value)
    selectrock = "Frost Gym Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 150000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 150000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Mythical Rock", function(Value)
    selectrock = "Mythical Gym Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 400000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 400000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Eternal Rock", function(Value)
    selectrock = "Eternal Gym Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 750000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 750000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Legend Rock", function(Value)
    selectrock = "Legend Gym Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 1000000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 1000000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Muscle King Rock", function(Value)
    selectrock = "Muscle King Gym Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 5000000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 5000000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

autoRockFolder:AddSwitch("Jungle Rock", function(Value)
    selectrock = "Ancient Jungle Rock"
    getgenv().autoFarm = Value
    
    task.spawn(function()
        while getgenv().autoFarm do
            task.wait()
            if not getgenv().autoFarm then break end
            
            if game:GetService("Players").LocalPlayer.Durability.Value >= 10000000 then
                for i, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == 10000000 and game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        gettool()
                    end
                end
            end
        end
    end)
end)

local autoEquipToolsFolder = farmTab:AddFolder("  Auto Equip Tools")

-- Free AutoLift Gamepass Button
autoEquipToolsFolder:AddButton("Gamepass AutoLift", function()
    local gamepassFolder = game:GetService("ReplicatedStorage").gamepassIds
    local player = game:GetService("Players").LocalPlayer
    for _, gamepass in pairs(gamepassFolder:GetChildren()) do
        local value = Instance.new("IntValue")
        value.Name = gamepass.Name
        value.Value = gamepass.Value
        value.Parent = player.ownedGamepasses
    end
end, "Desbloquea el gamepass de AutoLift gratis")

-- Auto Weight Toggle
autoEquipToolsFolder:AddSwitch("Auto Weight", function(Value)
    _G.AutoWeight = Value
    
    if Value then
        local weightTool = game.Players.LocalPlayer.Backpack:FindFirstChild("Weight")
        if weightTool then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(weightTool)
        end
    else
        local character = game.Players.LocalPlayer.Character
        local equipped = character:FindFirstChild("Weight")
        if equipped then
            equipped.Parent = game.Players.LocalPlayer.Backpack
        end
    end
    
    task.spawn(function()
        while _G.AutoWeight do
            if not _G.AutoWeight then break end
            game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
            task.wait(0.1)
        end
    end)
end, "Levanta pesas automÃ¡ticamente")

-- Auto Pushups Toggle
autoEquipToolsFolder:AddSwitch("Auto Pushups", function(Value)
    _G.AutoPushups = Value
    
    if Value then
        local pushupsTool = game.Players.LocalPlayer.Backpack:FindFirstChild("Pushups")
        if pushupsTool then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(pushupsTool)
        end
    else
        local character = game.Players.LocalPlayer.Character
        local equipped = character:FindFirstChild("Pushups")
        if equipped then
            equipped.Parent = game.Players.LocalPlayer.Backpack
        end
    end
    
    task.spawn(function()
        while _G.AutoPushups do
            if not _G.AutoPushups then break end
            game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
            task.wait(0.1)
        end
    end)
end, "Haz flexiones automÃ¡ticamente")

-- Auto Handstands Toggle
autoEquipToolsFolder:AddSwitch("Auto Handstands", function(Value)
    _G.AutoHandstands = Value
    
    if Value then
        local handstandsTool = game.Players.LocalPlayer.Backpack:FindFirstChild("Handstands")
        if handstandsTool then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(handstandsTool)
        end
    else
        local character = game.Players.LocalPlayer.Character
        local equipped = character:FindFirstChild("Handstands")
        if equipped then
            equipped.Parent = game.Players.LocalPlayer.Backpack
        end
    end
    
    task.spawn(function()
        while _G.AutoHandstands do
            if not _G.AutoHandstands then break end
            game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
            task.wait(0.1)
        end
    end)
end, "Haz paradas de manos automÃ¡ticamente")

-- Auto Situps Toggle
autoEquipToolsFolder:AddSwitch("Auto Situps", function(Value)
    _G.AutoSitups = Value
    
    if Value then
        local situpsTool = game.Players.LocalPlayer.Backpack:FindFirstChild("Situps")
        if situpsTool then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(situpsTool)
        end
    else
        local character = game.Players.LocalPlayer.Character
        local equipped = character:FindFirstChild("Situps")
        if equipped then
            equipped.Parent = game.Players.LocalPlayer.Backpack
        end
    end
    
    task.spawn(function()
        while _G.AutoSitups do
            if not _G.AutoSitups then break end
            game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
            task.wait(0.1)
        end
    end)
end, "Haz abdominales automÃ¡ticamente")

-- Auto Punch Toggle
autoEquipToolsFolder:AddSwitch("Auto Punch", function(Value)
    _G.fastHitActive = Value
    
    if Value then
        -- Function to equip and modify punch
        task.spawn(function()
            while _G.fastHitActive do
                if not _G.fastHitActive then break end
                
                local player = game.Players.LocalPlayer
                local punch = player.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = player.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait(0.1)
            end
        end)
        
        -- Function for rapid punching
        task.spawn(function()
            while _G.fastHitActive do
                if not _G.fastHitActive then break end
                
                local player = game.Players.LocalPlayer
                player.muscleEvent:FireServer("punch", "rightHand")
                player.muscleEvent:FireServer("punch", "leftHand")
                
                local character = player.Character
                if character then
                    local punchTool = character:FindFirstChild("Punch")
                    if punchTool then
                        punchTool:Activate()
                    end
                end
                task.wait(0)
            end
        end)
    else
        local character = game.Players.LocalPlayer.Character
        local equipped = character:FindFirstChild("Punch")
        if equipped then
            equipped.Parent = game.Players.LocalPlayer.Backpack
        end
    end
end, "Golpea automÃ¡ticamente")

-- Fast Tools Toggle
autoEquipToolsFolder:AddSwitch("Fast Tools", function(Value)
    _G.FastTools = Value
    
    local defaultSpeeds = {
        {
            "Punch",
            "attackTime",
            Value and 0 or 0.35
        },
        {
            "Ground Slam",
            "attackTime",
            Value and 0 or 6
        },
        {
            "Stomp",
            "attackTime",
            Value and 0 or 7
        },
        {
            "Handstands",
            "repTime",
            Value and 0 or 1
        },
        {
            "Pushups",
            "repTime",
            Value and 0 or 1
        },
        {
            "Weight",
            "repTime",
            Value and 0 or 1
        },
        {
            "Situps",
            "repTime",
            Value and 0 or 1
        }
    }
    
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    
    for _, toolInfo in ipairs(defaultSpeeds) do
        local tool = backpack:FindFirstChild(toolInfo[1])
        if tool and tool:FindFirstChild(toolInfo[2]) then
            tool[toolInfo[2]].Value = toolInfo[3]
        end
        
        local equippedTool = player.Character and player.Character:FindFirstChild(toolInfo[1])
        if equippedTool and equippedTool:FindFirstChild(toolInfo[2]) then
            equippedTool[toolInfo[2]].Value = toolInfo[3]
        end
    end
end, "Acelera todas las herramientas")

local rebirthsFolder = farmTab:AddFolder("  Auto Rebirths")

-- Target rebirth input - direct text input
rebirthsFolder:AddTextBox("Rebirth Target", function(text)
    local newValue = tonumber(text)
    if newValue and newValue > 0 then
        targetRebirthValue = newValue
        updateStats() -- Call the stats update function
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Objetivo Actualizado",
            Text = "Nuevo objetivo: " .. tostring(targetRebirthValue) .. " renacimientos",
            Duration = 0
        })
    else
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Entrada InvÃ¡lida",
            Text = "Por favor ingresa un nÃºmero vÃ¡lido mayor que 0",
            Duration = 0
        })
    end
end)

-- Create toggle switches
local infiniteSwitch -- Forward declaration

local targetSwitch = rebirthsFolder:AddSwitch("Auto Rebirth Target", function(bool)
    _G.targetRebirthActive = bool
    
    if bool then
        -- Turn off infinite rebirth if it's on
        if _G.infiniteRebirthActive and infiniteSwitch then
            infiniteSwitch:Set(false)
            _G.infiniteRebirthActive = false
        end
        
        -- Start target rebirth loop
        spawn(function()
            while _G.targetRebirthActive and wait(0.1) do
                local currentRebirths = game.Players.LocalPlayer.leaderstats.Rebirths.Value
                
                if currentRebirths >= targetRebirthValue then
                    targetSwitch:Set(false)
                    _G.targetRebirthActive = false
                    
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Â¡Objetivo Alcanzado!",
                        Text = "Has alcanzado " .. tostring(targetRebirthValue) .. " renacimientos",
                        Duration = 5
                    })
                    
                    break
                end
                
                game:GetService("ReplicatedStorage").rEvents.rebirthRemote:InvokeServer("rebirthRequest")
            end
        end)
    end
end, "Renacimiento automÃ¡tico hasta alcanzar el objetivo")

infiniteSwitch = rebirthsFolder:AddSwitch("Auto Rebirth (Infinite)", function(bool)
    _G.infiniteRebirthActive = bool
    
    if bool then
        -- Turn off target rebirth if it's on
        if _G.targetRebirthActive and targetSwitch then
            targetSwitch:Set(false)
            _G.targetRebirthActive = false
        end
        
        -- Start infinite rebirth loop
        spawn(function()
            while _G.infiniteRebirthActive and wait(0.1) do
                game:GetService("ReplicatedStorage").rEvents.rebirthRemote:InvokeServer("rebirthRequest")
            end
        end)
    end
end, "Renacimiento continuo sin parar")

local sizeSwitch = rebirthsFolder:AddSwitch("Auto Size 1", function(bool)
    _G.autoSizeActive = bool
    
    if bool then
        spawn(function()
            while _G.autoSizeActive and wait() do
                game:GetService("ReplicatedStorage").rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 1)
            end
        end)
    end
end, "Establece el tamaÃ±o del personaje a 1 continuamente")

local teleportSwitch = rebirthsFolder:AddSwitch("Auto Teleport to Muscle King", function(bool)
    _G.teleportActive = bool
    
    if bool then
        spawn(function()
            while _G.teleportActive and wait() do
                if game.Players.LocalPlayer.Character then
                    game.Players.LocalPlayer.Character:MoveTo(Vector3.new(-8646, 17, -5738))
                end
            end
        end)
    end
end, "Teletransporte continuo al Rey MÃºsculo")

-- Crear folder "Tools Combinate"
local toolsCombFolder = farmTab:AddFolder("  Auto Tools OP")

-- FunciÃ³n auxiliar para equipar herramienta y poner fast rep
local function equipToolFast(player, toolName, repName, repValue)
    local tool = player.Backpack:FindFirstChild(toolName) or player.Character:FindFirstChild(toolName)
    if tool and player.Character and player.Character:FindFirstChild("Humanoid") then
        tool.Parent = player.Character
        if repName and tool:FindFirstChild(repName) then
            tool[repName].Value = repValue
        end
    end
end

-- FunciÃ³n auxiliar para hacer punch rÃ¡pido
local function fastPunch(player)
    local punch = player.Backpack:FindFirstChild("Punch") or player.Character:FindFirstChild("Punch")
    if punch then
        punch.Parent = player.Character
        if punch:FindFirstChild("attackTime") then punch.attackTime.Value = 0 end
        player.muscleEvent:FireServer("punch", "rightHand")
        player.muscleEvent:FireServer("punch", "leftHand")
        local punchTool = player.Character and player.Character:FindFirstChild("Punch")
        if punchTool then punchTool:Activate() end
    end
end

-- =========================================
-- Auto Weight + Punch
toolsCombFolder:AddSwitch("Auto Weight + Punch", function(Value)
    _G.ComboWeightPunch = Value
    local player = game.Players.LocalPlayer

    if Value then
        task.spawn(function()
            while _G.ComboWeightPunch do
                equipToolFast(player, "Weight", "repTime", 0)
                player.muscleEvent:FireServer("rep")
                fastPunch(player)
                task.wait()
            end
        end)
    else
        local char = player.Character
        for _, toolName in ipairs({"Weight", "Punch"}) do
            local eq = char:FindFirstChild(toolName)
            if eq then eq.Parent = player.Backpack end
        end
    end
end, "Levanta pesas y golpea automÃ¡ticamente (rÃ¡pido)")

-- Auto Pushups + Punch
toolsCombFolder:AddSwitch("Auto Pushups + Punch", function(Value)
    _G.ComboPushupPunch = Value
    local player = game.Players.LocalPlayer

    if Value then
        task.spawn(function()
            while _G.ComboPushupPunch do
                equipToolFast(player, "Pushups", "repTime", 0)
                player.muscleEvent:FireServer("rep")
                fastPunch(player)
                task.wait()
            end
        end)
    else
        local char = player.Character
        for _, toolName in ipairs({"Pushups", "Punch"}) do
            local eq = char:FindFirstChild(toolName)
            if eq then eq.Parent = player.Backpack end
        end
    end
end, "Haz flexiones y golpea automÃ¡ticamente (rÃ¡pido)")

-- Auto Situps + Punch
toolsCombFolder:AddSwitch("Auto Situps + Punch", function(Value)
    _G.ComboSitupPunch = Value
    local player = game.Players.LocalPlayer

    if Value then
        task.spawn(function()
            while _G.ComboSitupPunch do
                equipToolFast(player, "Situps", "repTime", 0)
                player.muscleEvent:FireServer("rep")
                fastPunch(player)
                task.wait()
            end
        end)
    else
        local char = player.Character
        for _, toolName in ipairs({"Situps", "Punch"}) do
            local eq = char:FindFirstChild(toolName)
            if eq then eq.Parent = player.Backpack end
        end
    end
end, "Haz abdominales y golpea automÃ¡ticamente (rÃ¡pido)")

-- Auto Handstands + Punch
toolsCombFolder:AddSwitch("Auto Handstands + Punch", function(Value)
    _G.ComboHandstandPunch = Value
    local player = game.Players.LocalPlayer

    if Value then
        task.spawn(function()
            while _G.ComboHandstandPunch do
                equipToolFast(player, "Handstands", "repTime", 0)
                player.muscleEvent:FireServer("rep")
                fastPunch(player)
                task.wait()
            end
        end)
    else
        local char = player.Character
        for _, toolName in ipairs({"Handstands", "Punch"}) do
            local eq = char:FindFirstChild(toolName)
            if eq then eq.Parent = player.Backpack end
        end
    end
end, "Haz paradas de manos y golpea automÃ¡ticamente (rÃ¡pido)")

local Gift = window:AddTab("Gift")
Gift:AddLabel("Gifting Protein egg:").TextSize = 22

local proteinEggLabel = Gift:AddLabel("Protein Eggs: 0")
proteinEggLabel.TextSize = 20

local selectedEggPlayer = nil
local eggCount = 0

local eggDropdown = Gift:AddDropdown("Player to Gift Eggs", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            selectedEggPlayer = plr
            break
        end
    end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= Players.LocalPlayer then
        eggDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= Players.LocalPlayer then
        eggDropdown:Add(plr.DisplayName)
    end
end)

Gift:AddTextBox("Amount of Eggs", function(text)
    eggCount = tonumber(text) or 0
end)

Gift:AddButton("Gift Eggs", function()
    if selectedEggPlayer and eggCount > 0 then
        for i = 1, eggCount do
            local egg = Players.LocalPlayer.consumablesFolder:FindFirstChild("Protein Egg")
            if egg then
                ReplicatedStorage.rEvents.giftRemote:InvokeServer("giftRequest", selectedEggPlayer, egg)
                task.wait(0.1)
            end
        end
    end
end)

Gift:AddLabel("Gifting Tropical Shakes:").TextSize = 22

local tropicalShakeLabel = Gift:AddLabel("Tropical Shakes: 0")
tropicalShakeLabel.TextSize = 18

local selectedShakePlayer = nil
local shakeCount = 0

local shakeDropdown = Gift:AddDropdown("Player to Gift Tropical Shakes", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            selectedShakePlayer = plr
            break
        end
    end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= Players.LocalPlayer then
        shakeDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= Players.LocalPlayer then
        shakeDropdown:Add(plr.DisplayName)
    end
end)

Gift:AddTextBox("Tropical Shakes gift", function(text)
    shakeCount = tonumber(text) or 0
end)

Gift:AddButton("Gift Tropical Shakes", function()
    if selectedShakePlayer and shakeCount > 0 then
        for i = 1, shakeCount do
            local shake = Players.LocalPlayer.consumablesFolder:FindFirstChild("Tropical Shake")
            if shake then
                ReplicatedStorage.rEvents.giftRemote:InvokeServer("giftRequest", selectedShakePlayer, shake)
                task.wait(0.1)
            end
        end
    end
end)

local function updateItemCount()
    local proteinEggCount = 0
    local tropicalShakeCount = 0

    local backpack = Players.LocalPlayer:WaitForChild("Backpack")
    if backpack then
        for _, item in ipairs(backpack:GetChildren()) do
            if item.Name == "Protein Egg" then
                proteinEggCount = proteinEggCount + 1
            elseif item.Name == "Tropical Shake" or item.Name == "PiÃ±as" then
                tropicalShakeCount = tropicalShakeCount + 1
            end
        end
    end

    proteinEggLabel.Text = "Protein Eggs: " .. proteinEggCount
    tropicalShakeLabel.Text = "Tropical Shakes: " .. tropicalShakeCount
end

task.spawn(function()
    while true do
        updateItemCount()
        task.wait(0.25)
    end
end)


local autoEatBoostsEnabled = false

local boostsList = {
    "ULTRA Shake",
    "TOUGH Bar",
    "Protein Shake",
    "Energy Shake",
    "Protein Bar",
    "Energy Bar",
    "Tropical Shake"
}

local function eatAllBoosts()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    for _, boostName in ipairs(boostsList) do
        local boost = backpack:FindFirstChild(boostName)
        while boost do
            boost.Parent = character
            pcall(function()
                boost:Activate()
            end)
            task.wait(0)
            boost = backpack:FindFirstChild(boostName)
        end
    end
end

task.spawn(function()
    while true do
        if autoEatBoostsEnabled then
            eatAllBoosts()
            task.wait(2)
        else
            task.wait(1)
        end
    end
end)

Gift:AddSwitch("Auto Clear Inventory", function(state)
    autoEatBoostsEnabled = state
end)

local miscTab = window:AddTab("Misc")

local misc1Folder = miscTab:AddFolder("  Misc 1")

local player = game:GetService("Players").LocalPlayer
local sizeValue = 2
local speedValue = 16
local autoSizeEnabled = false
local autoSpeedEnabled = false
local autoSizeConnection = nil
local autoSpeedConnection = nil

misc1Folder:AddTextBox("Auto Size", function(text)
    sizeValue = tonumber(text) or 2

    if autoSizeEnabled then
        pcall(function()
            game:GetService("ReplicatedStorage").rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", sizeValue)
        end)
    end
end)

local autoSizeSwitch = misc1Folder:AddSwitch("Auto Set Size", function(bool)
    autoSizeEnabled = bool

    if autoSizeEnabled then
        spawn(function()
            while autoSizeEnabled do
                pcall(function()
                    game:GetService("ReplicatedStorage").rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", sizeValue)
                end)
                wait(0.1)
            end
        end)
    end
end)

misc1Folder:AddTextBox("Auto Speed", function(text)
    speedValue = tonumber(text) or 16

    if player and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = speedValue
    end
end)

misc1Folder:AddSwitch("Auto Set Speed", function(bool)
    autoSpeedEnabled = bool

    if autoSpeedEnabled then
        if player and player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid.WalkSpeed = speedValue
        end

        if autoSpeedConnection then
            autoSpeedConnection:Disconnect()
        end

        autoSpeedConnection = player.CharacterAdded:Connect(function(character)
            local humanoid = character:WaitForChild("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = speedValue
            end
        end)
    else
        if autoSpeedConnection then
            autoSpeedConnection:Disconnect()
            autoSpeedConnection = nil
        end
    end
end)

-- Misc 2 Folder
local Misc2 = miscTab:AddFolder("  Misc 2")

Misc2:AddButton("Infinite Yield", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
end)

Misc2:AddButton("GhostX Hub", function()
    loadstring(game:HttpGet("https://scriptblox.com/raw/Universal-Script-GhostX-Hub-7158"))()
end)

-- Misc 3 Folder
local misc3Folder = miscTab:AddFolder("  Misc 3")

-- Variables para No-Clip
local noclipEnabled = false
local noclipConnection = nil
local originalCanCollide = {}

-- Variables para Infinite Jump
local infiniteJumpEnabled = false
local infiniteJumpConnection = nil

-- FunciÃ³n para No-Clip
local function toggleNoclip(enabled)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    
    if enabled then
        -- Activar No-Clip
        noclipConnection = game:GetService("RunService").Stepped:Connect(function()
            if character and character:FindFirstChild("HumanoidRootPart") then
                for _, part in pairs(character:GetChildren()) do
                    if part:IsA("BasePart") then
                        -- Guardar estado original si no estÃ¡ guardado
                        if originalCanCollide[part] == nil then
                            originalCanCollide[part] = part.CanCollide
                        end
                        -- Desactivar colisiÃ³n
                        part.CanCollide = false
                    end
                end
            end
        end)
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "No-Clip";
            Text = "No-Clip activado! Puedes atravesar objetos.";
            Duration = 3;
        })
        
        print("No-Clip activado")
    else
        -- Desactivar No-Clip
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        
        -- Restaurar colisiones originales
        if character then
            for _, part in pairs(character:GetChildren()) do
                if part:IsA("BasePart") and originalCanCollide[part] ~= nil then
                    part.CanCollide = originalCanCollide[part]
                end
            end
        end
        
        -- Limpiar tabla de estados originales
        originalCanCollide = {}
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "No-Clip";
            Text = "No-Clip desactivado! Colisiones restauradas.";
            Duration = 3;
        })
        
        print("No-Clip desactivado")
    end
end

-- FunciÃ³n para Infinite Jump
local function toggleInfiniteJump(enabled)
    local player = game.Players.LocalPlayer
    
    if enabled then
        -- Activar Infinite Jump
        infiniteJumpConnection = game:GetService("UserInputService").JumpRequest:Connect(function()
            local character = player.Character
            if character and character:FindFirstChild("Humanoid") then
                character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Infinite Jump";
            Text = "Infinite Jump activado! Salta infinitamente.";
            Duration = 3;
        })
        
        print("Infinite Jump activado")
    else
        -- Desactivar Infinite Jump
        if infiniteJumpConnection then
            infiniteJumpConnection:Disconnect()
            infiniteJumpConnection = nil
        end
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Infinite Jump";
            Text = "Infinite Jump desactivado! Salto normal restaurado.";
            Duration = 3;
        })
        
        print("Infinite Jump desactivado")
    end
end

-- FunciÃ³n para limpiar conexiones cuando el personaje respawnea
local function setupCharacterConnections()
    local player = game.Players.LocalPlayer
    
    player.CharacterAdded:Connect(function(character)
        -- Esperar un poco para que el personaje se cargue completamente
        wait(1)
        
        -- Reactivar No-Clip si estaba activo
        if noclipEnabled then
            toggleNoclip(false) -- Desactivar primero
            wait(0.1)
            toggleNoclip(true)  -- Reactivar
        end
        
        -- Reactivar Infinite Jump si estaba activo
        if infiniteJumpEnabled then
            toggleInfiniteJump(false) -- Desactivar primero
            wait(0.1)
            toggleInfiniteJump(true)  -- Reactivar
        end
    end)
end

-- Configurar conexiones de personaje
setupCharacterConnections()

-- Switch para No-Clip
local noclipSwitch = misc3Folder:AddSwitch("No-Clip", function(enabled)
    noclipEnabled = enabled
    toggleNoclip(enabled)
end)

-- Switch para Infinite Jump
local infiniteJumpSwitch = misc3Folder:AddSwitch("Infinite Jump", function(enabled)
    infiniteJumpEnabled = enabled
    toggleInfiniteJump(enabled)
end)

local timeDropdown =
    misc3Folder:AddDropdown(
    "Change Time",
    function(selection)
        local lighting = game:GetService("Lighting")

        if selection == "Night" then
            lighting.ClockTime = 0
        elseif selection == "Day" then
            lighting.ClockTime = 12
        elseif selection == "Midnight" then
            lighting.ClockTime = 6
        end
    end
)

timeDropdown:Add("Night")
timeDropdown:Add("Day")
timeDropdown:Add("Midnight")

-- Misc 4 Folder
local misc4Folder = miscTab:AddFolder("  Misc 4")

misc4Folder:AddButton("Rejoin Server", function()
    local TeleportService = game:GetService("TeleportService")
    
    -- Obtener servidores disponibles y unirse a uno aleatorio
    local success, result = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, nil, player)
    end)
    
    if not success then
        -- Si falla, intentar teleport normal a un nuevo servidor
        TeleportService:Teleport(game.PlaceId, player)
    end
    
    print("Cambiando a otro servidor...")
end)

local misc5Folder = miscTab:AddFolder("  Misc 5")

misc5Folder:AddButton("Remove Portals", function()
    -- Remove existing ad portals
    for _, portal in pairs(game:GetDescendants()) do
        if portal.Name == "RobloxForwardPortals" then
            portal:Destroy()
        end
    end
    
    -- Set up connection to remove future ad portals
    if _G.AdRemovalConnection then
        _G.AdRemovalConnection:Disconnect()
    end
    
    _G.AdRemovalConnection = game.DescendantAdded:Connect(function(descendant)
        if descendant.Name == "RobloxForwardPortals" then
            descendant:Destroy()
        end
    end)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Anuncios Eliminados",
        Text = "Los anuncios de Roblox han sido eliminados",
        Duration = 0
    })
end)

-- Add Auto Spin Wheel toggle
misc5Folder:AddSwitch("Auto Spin Wheel", function(bool)
    _G.AutoSpinWheel = bool
    
    if bool then
        spawn(function()
            while _G.AutoSpinWheel and wait(1) do
                game:GetService("ReplicatedStorage").rEvents.openFortuneWheelRemote:InvokeServer("openFortuneWheel", game:GetService("ReplicatedStorage").fortuneWheelChances["Fortune Wheel"])
            end
        end)
    end
end)

local parts = {}
local partSize = 2048
local totalDistance = 50000
local startPosition = Vector3.new(-2, -9.5, -2)
local numberOfParts = math.ceil(totalDistance / partSize)

local function createParts()
    for x = 0, numberOfParts - 1 do
        for z = 0, numberOfParts - 1 do
            local newPartSide = Instance.new("Part")
            newPartSide.Size = Vector3.new(partSize, 1, partSize)
            newPartSide.Position = startPosition + Vector3.new(x * partSize, 0, z * partSize)
            newPartSide.Anchored = true
            newPartSide.Transparency = 1
            newPartSide.CanCollide = true
            newPartSide.Name = "Part_Side_" .. x .. "_" .. z
            newPartSide.Parent = workspace
            table.insert(parts, newPartSide)
            
            local newPartLeftRight = Instance.new("Part")
            newPartLeftRight.Size = Vector3.new(partSize, 1, partSize)
            newPartLeftRight.Position = startPosition + Vector3.new(-x * partSize, 0, z * partSize)
            newPartLeftRight.Anchored = true
            newPartLeftRight.Transparency = 1
            newPartLeftRight.CanCollide = true
            newPartLeftRight.Name = "Part_LeftRight_" .. x .. "_" .. z
            newPartLeftRight.Parent = workspace
            table.insert(parts, newPartLeftRight)
            
            local newPartUpLeft = Instance.new("Part")
            newPartUpLeft.Size = Vector3.new(partSize, 1, partSize)
            newPartUpLeft.Position = startPosition + Vector3.new(-x * partSize, 0, -z * partSize)
            newPartUpLeft.Anchored = true
            newPartUpLeft.Transparency = 1
            newPartUpLeft.CanCollide = true
            newPartUpLeft.Name = "Part_UpLeft_" .. x .. "_" .. z
            newPartUpLeft.Parent = workspace
            table.insert(parts, newPartUpLeft)
            
            local newPartUpRight = Instance.new("Part")
            newPartUpRight.Size = Vector3.new(partSize, 1, partSize)
            newPartUpRight.Position = startPosition + Vector3.new(x * partSize, 0, -z * partSize)
            newPartUpRight.Anchored = true
            newPartUpRight.Transparency = 1
            newPartUpRight.CanCollide = true
            newPartUpRight.Name = "Part_UpRight_" .. x .. "_" .. z
            newPartUpRight.Parent = workspace
            table.insert(parts, newPartUpRight)
        end
    end
end

local function makePartsWalkthrough()
    for _, part in ipairs(parts) do
        if part and part.Parent then
            part.CanCollide = false
        end
    end
end

local function makePartsSolid()
    for _, part in ipairs(parts) do
        if part and part.Parent then
            part.CanCollide = true
        end
    end
end

misc5Folder:AddSwitch("Full Walk on Water", function(bool)
    if bool then
        createParts()
    else
        makePartsWalkthrough()
    end
end)

local pets = window:AddTab(       "Crystals")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Crystal data structure with exact names from your original code
local crystalData = {
    ["Blue Crystal"] = {
        {name = "Blue Birdie", rarity = "Basic"},
        {name = "Orange Hedgehog", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Red Kitty", rarity = "Basic"},
        {name = "Dark Vampy", rarity = "Advanced"},
        {name = "Blue Bunny", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Green Aura", rarity = "Basic"},
        {name = "Purple Aura", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Yellow Aura", rarity = "Basic"}
    },
    ["Green Crystal"] = {
        {name = "Silver Dog", rarity = "Basic"},
        {name = "Green Aura", rarity = "Advanced"},
        {name = "Dark Golem", rarity = "Advanced"},
        {name = "Green Butterfly", rarity = "Advanced"},
        {name = "Crimson Falcon", rarity = "Rare"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Green Aura", rarity = "Basic"},
        {name = "Purple Aura", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Yellow Aura", rarity = "Basic"}
    },
    ["Frost Crystal"] = {
        {name = "Yellow Butterfly", rarity = "Advanced"},
        {name = "Purple Dragon", rarity = "Rare"},
        {name = "Blue Pheonix", rarity = "Epic"},
        {name = "Orange Pegasus", rarity = "Rare"},
        {name = "Lightning", rarity = "Rare"},
        {name = "Electro", rarity = "Advanced"}
    },
    ["Mythical Crystal"] = {
        {name = "Purple Falcon", rarity = "Rare"},
        {name = "Red Dragon", rarity = "Rare"},
        {name = "Blue Firecaster", rarity = "Epic"},
        {name = "Golden Pheonix", rarity = "Epic"},
        {name = "Power Lightning", rarity = "Rare"},
        {name = "Dark Lightning", rarity = "Epic"}
    },
    ["Inferno Crystal"] = {
        {name = "Red Firecaster", rarity = "Epic"},
        {name = "Infernal Dragon", rarity = "Unique"},
        {name = "White Pegasus", rarity = "Rare"},
        {name = "Golden Pheonix", rarity = "Epic"},
        {name = "Inferno", rarity = "Epic"},
        {name = "Dark Storm", rarity = "Unique"}
    },
    ["Legends Crystal"] = {
        {name = "Ultra Birdie", rarity = "Unique"},
        {name = "Magic Butterfly", rarity = "Unique"},
        {name = "Green Firecaster", rarity = "Epic"},
        {name = "White Pheonix", rarity = "Epic"},
        {name = "Supernova", rarity = "Epic"},
        {name = "Purple Nova", rarity = "Unique"}
    },
    ["Muscle Elite Crystal"] = {
        {name = "Frostwave Legends Penguin", rarity = "Rare"},
        {name = "Phantom Genesis Dragon", rarity = "Rare"},
        {name = "Dark Legends Manticore", rarity = "Epic"},
        {name = "Ultimate Supernova Pegasus", rarity = "Epic"},
        {name = "Aether Spirit Bunny", rarity = "Unique"},
        {name = "Cybernetic Showdown Dragon", rarity = "Unique"}
    },
    ["Galaxy Oracle Crystal"] = {
        {name = "Eternal Strike Leviathan", rarity = "Rare"},
        {name = "Lightning Strike Phantom", rarity = "Epic"},
        {name = "Darkstar Hunter", rarity = "Unique"},
        {name = "Muscle King", rarity = "Unique"},
        {name = "Azure Tundra", rarity = "Epic"},
        {name = "Ultra Inferno", rarity = "Rare"}
    },
    ["Jungle Crystal"] = {
        {name = "Entropic Blast", rarity = "Unique"},
        {name = "Muscle Sensei", rarity = "Unique"},
        {name = "Grand Supernova", rarity = "Epic"},
        {name = "Neon Guardian", rarity = "Unique"},
        {name = "Eternal Megastrike", rarity = "Unique"},
        {name = "Golden Viking", rarity = "Epic"},
        {name = "Astral Electro", rarity = "Epic"},
        {name = "Dark Electro", rarity = "Epic"},
        {name = "Enchanted Mirage", rarity = "Epic"},
        {name = "Ultra Mirage", rarity = "Unique"},
        {name = "Unstable Mirage", rarity = "Unique"}
    }
}

-- Function to collect all unique pets and auras
local function getAllPetsAndAuras()
    local allPets = {}
    local allAuras = {}
    
    for crystalName, pets in pairs(crystalData) do
        for _, pet in ipairs(pets) do
            if string.find(pet.name, "Aura") then
                if not allAuras[pet.name] then
                    allAuras[pet.name] = {name = pet.name, rarity = pet.rarity, crystal = crystalName}
                end
            else
                if not allPets[pet.name] then
                    allPets[pet.name] = {name = pet.name, rarity = pet.rarity, crystal = crystalName}
                end
            end
        end
    end
    
    return allPets, allAuras
end

-- Function to find which crystal contains a specific pet/aura
local function findCrystalForItem(itemName)
    for crystalName, pets in pairs(crystalData) do
        for _, pet in ipairs(pets) do
            if pet.name == itemName then
                return crystalName
            end
        end
    end
    return nil
end

-- Variables to track current selections
local selectedPet = ""
local selectedAura = ""

-- Get all pets and auras
local allPets, allAuras = getAllPetsAndAuras()

pets:AddLabel("=== Buy pets and auras ===")

-- Pet dropdown
local petDropdown = pets:AddDropdown("Select pet", function(text)
    selectedPet = text
    local crystal = findCrystalForItem(text)
    print("Pet selected: " .. text .. " (Found in: " .. (crystal or "Unknown") .. ")")
end)

-- Add all pets manually (sorted by rarity)
-- Basic Pets
petDropdown:Add("Blue Birdie (Basic)")
petDropdown:Add("Orange Hedgehog (Basic)")
petDropdown:Add("Red Kitty (Basic)")
petDropdown:Add("Blue Bunny (Basic)")
petDropdown:Add("Silver Dog (Basic)")

-- Advanced Pets
petDropdown:Add("Dark Vampy (Advanced)")
petDropdown:Add("Dark Golem (Advanced)")
petDropdown:Add("Green Butterfly (Advanced)")
petDropdown:Add("Yellow Butterfly (Advanced)")

-- Rare Pets
petDropdown:Add("Crimson Falcon (Rare)")
petDropdown:Add("Purple Dragon (Rare)")
petDropdown:Add("Orange Pegasus (Rare)")
petDropdown:Add("Purple Falcon (Rare)")
petDropdown:Add("Red Dragon (Rare)")
petDropdown:Add("White Pegasus (Rare)")
petDropdown:Add("Frostwave Legends Penguin (Rare)")
petDropdown:Add("Phantom Genesis Dragon (Rare)")
petDropdown:Add("Eternal Strike Leviathan (Rare)")

-- Epic Pets
petDropdown:Add("Blue Pheonix (Epic)")
petDropdown:Add("Blue Firecaster (Epic)")
petDropdown:Add("Golden Pheonix (Epic)")
petDropdown:Add("Red Firecaster (Epic)")
petDropdown:Add("Green Firecaster (Epic)")
petDropdown:Add("White Pheonix (Epic)")
petDropdown:Add("Dark Legends Manticore (Epic)")
petDropdown:Add("Ultimate Supernova Pegasus (Epic)")
petDropdown:Add("Lightning Strike Phantom (Epic)")
petDropdown:Add("Golden Viking (Epic)")

-- Unique Pets
petDropdown:Add("Infernal Dragon (Unique)")
petDropdown:Add("Ultra Birdie (Unique)")
petDropdown:Add("Magic Butterfly (Unique)")
petDropdown:Add("Aether Spirit Bunny (Unique)")
petDropdown:Add("Cybernetic Showdown Dragon (Unique)")
petDropdown:Add("Darkstar Hunter (Unique)")
petDropdown:Add("Muscle Sensei (Unique)")
petDropdown:Add("Neon Guardian (Unique)")

-- Aura dropdown
local auraDropdown = pets:AddDropdown("Select Aura", function(text)
    selectedAura = text
    local crystal = findCrystalForItem(text)
    print("Aura selected: " .. text .. " (Found in: " .. (crystal or "Unknown") .. ")")
end)

-- Add all auras manually (sorted by rarity)
-- Basic Auras
auraDropdown:Add("Blue Aura (Basic)")
auraDropdown:Add("Green Aura (Basic)")
auraDropdown:Add("Purple Aura (Basic)")
auraDropdown:Add("Red Aura (Basic)")
auraDropdown:Add("Yellow Aura (Basic)")
auraDropdown:Add("Ultra Inferno  (Rare)")
auraDropdown:Add("Azure Tundra (Epic)")
auraDropdown:Add("Grand Supernova (Epic)")
auraDropdown:Add("Muscle King (Unique)")
auraDropdown:Add("Entropic Blast (Unique)")
auraDropdown:Add("Eternal Megastrike (Unique)")

pets:AddLabel("=== System to buys===")

-- Auto buy pet toggle
pets:AddSwitch("Auto Buy Pet", function(bool)
    _G.AutoBuyPet = bool
    
    if bool then
        if selectedPet == "" then
            print("Please select a pet first!")
            return
        end
        
        -- Extract pet name from dropdown selection (remove rarity part)
        local petName = selectedPet:match("^(.-)%s*%(")
        if not petName then
            petName = selectedPet
        end
        
        local crystal = findCrystalForItem(petName)
        if not crystal then
            print("Could not find crystal for pet: " .. petName)
            return
        end
        
        print("Auto buy pet started for: " .. petName .. " from " .. crystal)
        spawn(function()
            while _G.AutoBuyPet and selectedPet ~= "" do
                local petToBuy = ReplicatedStorage.cPetShopFolder:FindFirstChild(petName)
                if petToBuy then
                    ReplicatedStorage.cPetShopRemote:InvokeServer(petToBuy)
                    print("Bought pet: " .. petName)
                else
                    print("Pet not found: " .. petName)
                end
                task.wait(0.1)
            end
        end)
    else
        print("Auto buy pet stopped")
    end
end)

-- Auto buy aura toggle
pets:AddSwitch("Auto buy Aura", function(bool)
    _G.AutoBuyAura = bool
    
    if bool then
        if selectedAura == "" then
            print("Please select an aura first!")
            return
        end
        
        -- Extract aura name from dropdown selection (remove rarity part)
        local auraName = selectedAura:match("^(.-)%s*%(")
        if not auraName then
            auraName = selectedAura
        end
        
        local crystal = findCrystalForItem(auraName)
        if not crystal then
            print("Could not find crystal for aura: " .. auraName)
            return
        end
        
        print("Auto buy aura started for: " .. auraName .. " from " .. crystal)
        spawn(function()
            while _G.AutoBuyAura and selectedAura ~= "" do
                local auraToBuy = ReplicatedStorage.cPetShopFolder:FindFirstChild(auraName)
                if auraToBuy then
                    ReplicatedStorage.cPetShopRemote:InvokeServer(auraToBuy)
                    print("Bought aura: " .. auraName)
                else
                    print("Aura not found: " .. auraName)
                end
                task.wait(0.1)
            end
        end)
    else
        print("Auto buy aura stopped")
    end
end)

-- Show the pets tab
pets:Show()

pets:AddLabel("=== buy ultimates ===")

-- Ultimate options
local ultimateOptions = {
    "+1 Daily Spin",
    "+1 Pet Slot",
    "+10 Item Capacity",
    "+5% Rep Speed",
    "Demon Damage",
    "Galaxy Gains",
    "Golden Rebirth",
    "Jungle Swift",
    "Muscle Mind",
    "x2 Chest Rewards",
    "x2 Quest Rewards"
}

-- Variable to track selected ultimate
local selectedUltimate = ""

-- Ultimate dropdown
local ultimateDropdown = pets:AddDropdown("Select ultimate", function(text)
    selectedUltimate = text
    print("Ultimate selected: " .. text)
end)

-- Add all ultimate options to dropdown
for _, ultimate in ipairs(ultimateOptions) do
    ultimateDropdown:Add(ultimate)
end

-- Auto upgrade ultimate toggle
pets:AddSwitch("Auto Buy Ultimates", function(bool)
    _G.AutoUpgradeUltimate = bool
    
    if bool then
        if selectedUltimate == "" then
            print("Please select an ultimate first!")
            return
        end
        
        print("Auto upgrade ultimate started for: " .. selectedUltimate)
        spawn(function()
            while _G.AutoUpgradeUltimate and selectedUltimate ~= "" do
                game:GetService("ReplicatedStorage").rEvents.ultimatesRemote:InvokeServer(
                    "upgradeUltimate",
                    selectedUltimate
                )
                print("Upgraded ultimate: " .. selectedUltimate)
                task.wait(1)
            end
        end)
    else
        print("Auto comprar ultimates")
    end
end)

local SpecsTab = window:AddTab("Stats")

SpecsTab:AddLabel("Player Stats:").TextSize = 24

local playerToInspect = nil

local emojiMap = {
    ["Time"] = utf8.char(),
    ["Stats"] = utf8.char(),
    ["Strength"] = utf8.char(),
    ["Rebirths"] = utf8.char(),
    ["Durability"] = utf8.char(),
    ["Kills"] = utf8.char(),
    ["Agility"] = utf8.char(),
    ["Evil Karma"] = utf8.char(),
    ["Good Karma"] = utf8.char(),
    ["Brawls"] = utf8.char()
}

local statDefinitions = {
    {name = "Strength", statName = "Strength"},
    {name = "Rebirths", statName = "Rebirths"},
    {name = "Durability", statName = "Durability"},
    {name = "Agility", statName = "Agility"},
    {name = "Kills", statName = "Kills"},
    {name = "Evil Karma", statName = "evilKarma"},
    {name = "Good Karma", statName = "goodKarma"},
    {name = "Brawls", statName = "Brawls"}
}

local function getCurrentPlayers()
    local playersList = {}
    for _, p in ipairs(Players:GetPlayers()) do
        table.insert(playersList, p)
    end
    return playersList
end

local specdropdown =
    SpecsTab:AddDropdown(
    "Choose Player",
    function(text)
        for _, player in ipairs(getCurrentPlayers()) do
            local optionText = player.DisplayName .. " | " .. player.Name
            if text == optionText then
                playerToInspect = player
                updateStatLabels(playerToInspect)
                break
            end
        end
    end
)

for _, player in ipairs(getCurrentPlayers()) do
    specdropdown:Add(player.DisplayName .. " | " .. player.Name)
end

Players.PlayerAdded:Connect(
    function(player)
        specdropdown:Add(player.DisplayName .. " | " .. player.Name)
    end
)

Players.PlayerRemoving:Connect(
    function(player)
        specdropdown:Clear()
        for _, p in ipairs(getCurrentPlayers()) do
            specdropdown:Add(p.DisplayName .. " | " .. p.Name)
        end
    end
)

local playerNameLabel = SpecsTab:AddLabel("Name: N/A")
playerNameLabel.TextSize = 20

local playerUsernameLabel = SpecsTab:AddLabel("Username: N/A")
playerUsernameLabel.TextSize = 20

local statLabels = {}
for _, info in ipairs(statDefinitions) do
    statLabels[info.name] = SpecsTab:AddLabel(emojiMap[info.name] .. " " .. info.name .. ": 0 (0)")
    statLabels[info.name].TextSize = 20
end

local function formatNumber(n)
    if n >= 1e15 then
        return string.format("%.1fqa", n / 1e15)
    elseif n >= 1e12 then
        return string.format("%.1ft", n / 1e12)
    elseif n >= 1e9 then
        return string.format("%.1fb", n / 1e9)
    elseif n >= 1e6 then
        return string.format("%.1fm", n / 1e6)
    elseif n >= 1e3 then
        return string.format("%.1fk", n / 1e3)
    else
        return tostring(n)
    end
end

local function formatWithCommas(n)
    local formatted = tostring(n)
    while true do
        formatted, k = formatted:gsub("^(-?%d+)(%d%d%d)", "%1,%2")
        if k == 0 then
            break
        end
    end
    return formatted
end

local function updateStatLabels(targetPlayer)
    if not targetPlayer then
        return
    end

    playerNameLabel.Text = "Name: " .. targetPlayer.DisplayName
    playerUsernameLabel.Text = "Username: " .. targetPlayer.Name

    local leaderstats = targetPlayer:FindFirstChild("leaderstats")
    if not leaderstats then
        return
    end

    for _, info in ipairs(statDefinitions) do
        local statObject

        if leaderstats:FindFirstChild(info.statName) then
            statObject = leaderstats:FindFirstChild(info.statName)
        elseif targetPlayer:FindFirstChild(info.statName) then
            statObject = targetPlayer:FindFirstChild(info.statName)
        end

        if statObject then
            local value = statObject.Value
            local emoji = emojiMap[info.name] or ""
            statLabels[info.name].Text =
                string.format("%s %s: %s (%s)", emoji, info.name, formatNumber(value), formatWithCommas(value))
        else
            statLabels[info.name].Text = emojiMap[info.name] .. " " .. info.name .. ": 0 (0)"
        end
    end
end

task.spawn(
    function()
        while true do
            if playerToInspect then
                updateStatLabels(playerToInspect)
            end
            task.wait(0.2)
        end
    end
)

task.spawn(
    function()
        while true do
            if playerToInspect then
                updateAdvancedStats(playerToInspect)
            else
                updateAdvancedStats(nil)
            end
            task.wait(0.1)
        end
    end
)

local function checkCharacter()
    if not game.Players.LocalPlayer.Character then
        repeat
            task.wait()
        until game.Players.LocalPlayer.Character
    end
    return game.Players.LocalPlayer.Character
end

local function gettool()
    for _, v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
        if v.Name == "Punch" and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(v)
        end
    end
    game.Players.LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
    game.Players.LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
end

local function isPlayerAlive(player)
    return player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and
        player.Character:FindFirstChild("Humanoid") and
        player.Character.Humanoid.Health > 0
end

local function killPlayer(target)
    if not isPlayerAlive(target) then
        return
    end
    local character = checkCharacter()
    if character and character:FindFirstChild("LeftHand") then
        pcall(
            function()
                firetouchinterest(target.Character.HumanoidRootPart, character.LeftHand, 0)
                firetouchinterest(target.Character.HumanoidRootPart, character.LeftHand, 1)
                gettool()
            end
        )
    end
end

local strengthStartValue = 0
local durabilityStartValue = 0
local rebirthsStartValue = 0
local killsStartValue = 0
local brawlsStartValue = 0

local strengthStartTime = 0
local durabilityStartTime = 0
local rebirthsStartTime = 0
local killsStartTime = 0
local brawlsStartTime = 0

local strengthActive = false
local durabilityActive = false
local rebirthsActive = false
local killsActive = false
local brawlsActive = false

local Calculator = window:AddTab("Calculator")

-- Crear folders para todas las estadÃ­sticas
local strengthFolder = Calculator:AddFolder("Strength")
local durabilityFolder = Calculator:AddFolder("Durability")
local rebirthsFolder = Calculator:AddFolder("Rebirths")
local killsFolder = Calculator:AddFolder("Kills")
local brawlsFolder = Calculator:AddFolder("Brawls")

-- FunciÃ³n mejorada para formatear nÃºmeros
local function formatNumber(num)
    if num == math.huge then
        return "âˆž"
    elseif num == 0 then
        return "0"
    elseif num < 0.01 and num > 0 then
        return "~0"
    end
    
    local abs = math.abs(num)
    local sign = num < 0 and "-" or ""
    
    if abs >= 1e30 then
        return sign .. string.format("%.2f", abs / 1e30) .. "No"
    elseif abs >= 1e27 then
        return sign .. string.format("%.2f", abs / 1e27) .. "Oc"
    elseif abs >= 1e24 then
        return sign .. string.format("%.2f", abs / 1e24) .. "Sp"
    elseif abs >= 1e21 then
        return sign .. string.format("%.2f", abs / 1e21) .. "Sx"
    elseif abs >= 1e18 then
        return sign .. string.format("%.2f", abs / 1e18) .. "Qt"
    elseif abs >= 1e15 then
        return sign .. string.format("%.2f", abs / 1e15) .. "Qd"
    elseif abs >= 1e12 then
        return sign .. string.format("%.2f", abs / 1e12) .. "T"
    elseif abs >= 1e9 then
        return sign .. string.format("%.2f", abs / 1e9) .. "B"
    elseif abs >= 1e6 then
        return sign .. string.format("%.2f", abs / 1e6) .. "M"
    elseif abs >= 1e3 then
        return sign .. string.format("%.2f", abs / 1e3) .. "K"
    else
        return sign .. string.format("%.2f", abs)
    end
end

-- Funciones para obtener valores actuales de estadÃ­sticas
local function getCurrentStrength()
    local player = game.Players.LocalPlayer
    if player and player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Strength") then
        return player.leaderstats.Strength.Value
    end
    return 0
end

local function getCurrentDurability()
    local player = game.Players.LocalPlayer
    if player and player:FindFirstChild("Durability") then
        return player.Durability.Value
    end
    return 0
end

local function getCurrentRebirths()
    local player = game.Players.LocalPlayer
    if player and player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Rebirths") then
        return player.leaderstats.Rebirths.Value
    end
    return 0
end

local function getCurrentKills()
    local player = game.Players.LocalPlayer
    if player and player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Kills") then
        return player.leaderstats.Kills.Value
    end
    return 0
end

local function getCurrentBrawls()
    local player = game.Players.LocalPlayer
    if player and player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Brawls") then
        return player.leaderstats.Brawls.Value
    end
    return 0
end

-- Crear labels para Strength
local strengthLabels = {
    status = strengthFolder:AddLabel("Status: Presiona Start para comenzar"),
    current = strengthFolder:AddLabel("Current: 0"),
    gained = strengthFolder:AddLabel("Gained: 0"),
    perMinute = strengthFolder:AddLabel("Per Minute: 0"),
    perHour = strengthFolder:AddLabel("Per Hour: 0"),
    perDay = strengthFolder:AddLabel("Per Day: 0")
}

-- Crear labels para Durability
local durabilityLabels = {
    status = durabilityFolder:AddLabel("Status: Presiona Start para comenzar"),
    current = durabilityFolder:AddLabel("Current: 0"),
    gained = durabilityFolder:AddLabel("Gained: 0"),
    perMinute = durabilityFolder:AddLabel("Per Minute: 0"),
    perHour = durabilityFolder:AddLabel("Per Hour: 0"),
    perDay = durabilityFolder:AddLabel("Per Day: 0")
}

-- Crear labels para Rebirths
local rebirthsLabels = {
    status = rebirthsFolder:AddLabel("Status: Presiona Start para comenzar"),
    current = rebirthsFolder:AddLabel("Current: 0"),
    gained = rebirthsFolder:AddLabel("Gained: 0"),
    perMinute = rebirthsFolder:AddLabel("Per Minute: 0"),
    perHour = rebirthsFolder:AddLabel("Per Hour: 0"),
    perDay = rebirthsFolder:AddLabel("Per Day: 0")
}

-- Crear labels para Kills
local killsLabels = {
    status = killsFolder:AddLabel("Status: Presiona Start para comenzar"),
    current = killsFolder:AddLabel("Current: 0"),
    gained = killsFolder:AddLabel("Gained: 0"),
    perMinute = killsFolder:AddLabel("Per Minute: 0"),
    perHour = killsFolder:AddLabel("Per Hour: 0"),
    perDay = killsFolder:AddLabel("Per Day: 0")
}

-- Crear labels para Brawls
local brawlsLabels = {
    status = brawlsFolder:AddLabel("Status: Presiona Start para comenzar"),
    current = brawlsFolder:AddLabel("Current: 0"),
    gained = brawlsFolder:AddLabel("Gained: 0"),
    perMinute = brawlsFolder:AddLabel("Per Minute: 0"),
    perHour = brawlsFolder:AddLabel("Per Hour: 0"),
    perDay = brawlsFolder:AddLabel("Per Day: 0")
}

-- FunciÃ³n para actualizar labels de Strength
local function updateStrengthLabels()
    local currentStrength = getCurrentStrength()
    local currentTime = tick()
    
    strengthLabels.current.Text = "Current: " .. formatNumber(currentStrength)
    
    if strengthActive and strengthStartTime > 0 then
        local timeElapsed = currentTime - strengthStartTime
        local gained = currentStrength - strengthStartValue
        
        strengthLabels.gained.Text = "Gained: " .. formatNumber(gained)
        
        if timeElapsed >= 60 then -- Solo calcular despuÃ©s de 1 minuto
            local perMinute = (gained / timeElapsed) * 60
            local perHour = perMinute * 60
            local perDay = perHour * 24
            
            strengthLabels.status.Text = "Status:  Activo"
            strengthLabels.perMinute.Text = "Per Minute: " .. formatNumber(perMinute)
            strengthLabels.perHour.Text = "Per Hour: " .. formatNumber(perHour)
            strengthLabels.perDay.Text = "Per Day: " .. formatNumber(perDay)
        else
            local remaining = 60 - timeElapsed
            strengthLabels.status.Text = "Status:  Calculando... " .. math.ceil(remaining) .. "s"
            strengthLabels.perMinute.Text = "Per Minute: Calculando..."
            strengthLabels.perHour.Text = "Per Hour: Calculando..."
            strengthLabels.perDay.Text = "Per Day: Calculando..."
        end
    else
        strengthLabels.gained.Text = "Gained: 0"
        strengthLabels.perMinute.Text = "Per Minute: 0"
        strengthLabels.perHour.Text = "Per Hour: 0"
        strengthLabels.perDay.Text = "Per Day: 0"
    end
end

-- FunciÃ³n para actualizar labels de Durability
local function updateDurabilityLabels()
    local currentDurability = getCurrentDurability()
    local currentTime = tick()
    
    durabilityLabels.current.Text = "Current: " .. formatNumber(currentDurability)
    
    if durabilityActive and durabilityStartTime > 0 then
        local timeElapsed = currentTime - durabilityStartTime
        local gained = currentDurability - durabilityStartValue
        
        durabilityLabels.gained.Text = "Gained: " .. formatNumber(gained)
        
        if timeElapsed >= 60 then -- Solo calcular despuÃ©s de 1 minuto
            local perMinute = (gained / timeElapsed) * 60
            local perHour = perMinute * 60
            local perDay = perHour * 24
            
            durabilityLabels.status.Text = "Status:  Activo"
            durabilityLabels.perMinute.Text = "Per Minute: " .. formatNumber(perMinute)
            durabilityLabels.perHour.Text = "Per Hour: " .. formatNumber(perHour)
            durabilityLabels.perDay.Text = "Per Day: " .. formatNumber(perDay)
        else
            local remaining = 60 - timeElapsed
            durabilityLabels.status.Text = "Status:  Calculando... " .. math.ceil(remaining) .. "s"
            durabilityLabels.perMinute.Text = "Per Minute: Calculando..."
            durabilityLabels.perHour.Text = "Per Hour: Calculando..."
            durabilityLabels.perDay.Text = "Per Day: Calculando..."
        end
    else
        durabilityLabels.gained.Text = "Gained: 0"
        durabilityLabels.perMinute.Text = "Per Minute: 0"
        durabilityLabels.perHour.Text = "Per Hour: 0"
        durabilityLabels.perDay.Text = "Per Day: 0"
    end
end

-- FunciÃ³n para actualizar labels de Rebirths
local function updateRebirthsLabels()
    local currentRebirths = getCurrentRebirths()
    local currentTime = tick()
    
    rebirthsLabels.current.Text = "Current: " .. formatNumber(currentRebirths)
    
    if rebirthsActive and rebirthsStartTime > 0 then
        local timeElapsed = currentTime - rebirthsStartTime
        local gained = currentRebirths - rebirthsStartValue
        
        rebirthsLabels.gained.Text = "Gained: " .. formatNumber(gained)
        
        if timeElapsed >= 60 then -- Solo calcular despuÃ©s de 1 minuto
            local perMinute = (gained / timeElapsed) * 60
            local perHour = perMinute * 60
            local perDay = perHour * 24
            
            rebirthsLabels.status.Text = "Status:  Activo"
            rebirthsLabels.perMinute.Text = "Per Minute: " .. formatNumber(perMinute)
            rebirthsLabels.perHour.Text = "Per Hour: " .. formatNumber(perHour)
            rebirthsLabels.perDay.Text = "Per Day: " .. formatNumber(perDay)
        else
            local remaining = 60 - timeElapsed
            rebirthsLabels.status.Text = "Status:  Calculando... " .. math.ceil(remaining) .. "s"
            rebirthsLabels.perMinute.Text = "Per Minute: Calculando..."
            rebirthsLabels.perHour.Text = "Per Hour: Calculando..."
            rebirthsLabels.perDay.Text = "Per Day: Calculando..."
        end
    else
        rebirthsLabels.gained.Text = "Gained: 0"
        rebirthsLabels.perMinute.Text = "Per Minute: 0"
        rebirthsLabels.perHour.Text = "Per Hour: 0"
        rebirthsLabels.perDay.Text = "Per Day: 0"
    end
end

-- FunciÃ³n para actualizar labels de Kills
local function updateKillsLabels()
    local currentKills = getCurrentKills()
    local currentTime = tick()
    
    killsLabels.current.Text = "Current: " .. formatNumber(currentKills)
    
    if killsActive and killsStartTime > 0 then
        local timeElapsed = currentTime - killsStartTime
        local gained = currentKills - killsStartValue
        
        killsLabels.gained.Text = "Gained: " .. formatNumber(gained)
        
        if timeElapsed >= 60 then -- Solo calcular despuÃ©s de 1 minuto
            local perMinute = (gained / timeElapsed) * 60
            local perHour = perMinute * 60
            local perDay = perHour * 24
            
            killsLabels.status.Text = "Status:  Activo"
            killsLabels.perMinute.Text = "Per Minute: " .. formatNumber(perMinute)
            killsLabels.perHour.Text = "Per Hour: " .. formatNumber(perHour)
            killsLabels.perDay.Text = "Per Day: " .. formatNumber(perDay)
        else
            local remaining = 60 - timeElapsed
            killsLabels.status.Text = "Status:  Calculando... " .. math.ceil(remaining) .. "s"
            killsLabels.perMinute.Text = "Per Minute: Calculando..."
            killsLabels.perHour.Text = "Per Hour: Calculando..."
            killsLabels.perDay.Text = "Per Day: Calculando..."
        end
    else
        killsLabels.gained.Text = "Gained: 0"
        killsLabels.perMinute.Text = "Per Minute: 0"
        killsLabels.perHour.Text = "Per Hour: 0"
        killsLabels.perDay.Text = "Per Day: 0"
    end
end

-- FunciÃ³n para actualizar labels de Brawls
local function updateBrawlsLabels()
    local currentBrawls = getCurrentBrawls()
    local currentTime = tick()
    
    brawlsLabels.current.Text = "Current: " .. formatNumber(currentBrawls)
    
    if brawlsActive and brawlsStartTime > 0 then
        local timeElapsed = currentTime - brawlsStartTime
        local gained = currentBrawls - brawlsStartValue
        
        brawlsLabels.gained.Text = "Gained: " .. formatNumber(gained)
        
        if timeElapsed >= 60 then -- Solo calcular despuÃ©s de 1 minuto
            local perMinute = (gained / timeElapsed) * 60
            local perHour = perMinute * 60
            local perDay = perHour * 24
            
            brawlsLabels.status.Text = "Status:  Activo"
            brawlsLabels.perMinute.Text = "Per Minute: " .. formatNumber(perMinute)
            brawlsLabels.perHour.Text = "Per Hour: " .. formatNumber(perHour)
            brawlsLabels.perDay.Text = "Per Day: " .. formatNumber(perDay)
        else
            local remaining = 60 - timeElapsed
            brawlsLabels.status.Text = "Status:  Calculando... " .. math.ceil(remaining) .. "s"
            brawlsLabels.perMinute.Text = "Per Minute: Calculando..."
            brawlsLabels.perHour.Text = "Per Hour: Calculando..."
            brawlsLabels.perDay.Text = "Per Day: Calculando..."
        end
    else
        brawlsLabels.gained.Text = "Gained: 0"
        brawlsLabels.perMinute.Text = "Per Minute: 0"
        brawlsLabels.perHour.Text = "Per Hour: 0"
        brawlsLabels.perDay.Text = "Per Day: 0"
    end
end

-- Botones de control para Strength
strengthFolder:AddButton("Start Strength Calculator", function()
    strengthActive = true
    strengthStartTime = tick()
    strengthStartValue = getCurrentStrength()
    strengthLabels.status.Text = "Status:  Iniciado - Calculando..."
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Strength Calculator iniciado!";
        Duration = 3;
    })
end)

strengthFolder:AddButton("Stop Strength Calculator", function()
    strengthActive = false
    strengthStartTime = 0
    strengthStartValue = 0
    strengthLabels.status.Text = "Status:  Detenido"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Strength Calculator detenido!";
        Duration = 3;
    })
end)

-- Botones de control para Durability
durabilityFolder:AddButton("Start Durability Calculator", function()
    durabilityActive = true
    durabilityStartTime = tick()
    durabilityStartValue = getCurrentDurability()
    durabilityLabels.status.Text = "Status:  Iniciado - Calculando..."
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Durability Calculator iniciado!";
        Duration = 3;
    })
end)

durabilityFolder:AddButton("Stop Durability Calculator", function()
    durabilityActive = false
    durabilityStartTime = 0
    durabilityStartValue = 0
    durabilityLabels.status.Text = "Status:  Detenido"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Durability Calculator detenido!";
        Duration = 3;
    })
end)

-- Botones de control para Rebirths
rebirthsFolder:AddButton("Start Rebirths Calculator", function()
    rebirthsActive = true
    rebirthsStartTime = tick()
    rebirthsStartValue = getCurrentRebirths()
    rebirthsLabels.status.Text = "Status:  Iniciado - Calculando..."
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Rebirths Calculator iniciado!";
        Duration = 3;
    })
end)

rebirthsFolder:AddButton("Stop Rebirths Calculator", function()
    rebirthsActive = false
    rebirthsStartTime = 0
    rebirthsStartValue = 0
    rebirthsLabels.status.Text = "Status:  Detenido"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Rebirths Calculator detenido!";
        Duration = 3;
    })
end)

-- Botones de control para Kills
killsFolder:AddButton("Start Kills Calculator", function()
    killsActive = true
    killsStartTime = tick()
    killsStartValue = getCurrentKills()
    killsLabels.status.Text = "Status:  Iniciado - Calculando..."
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Kills Calculator iniciado!";
        Duration = 3;
    })
end)

killsFolder:AddButton("Stop Kills Calculator", function()
    killsActive = false
    killsStartTime = 0
    killsStartValue = 0
    killsLabels.status.Text = "Status:  Detenido"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Kills Calculator detenido!";
        Duration = 3;
    })
end)

-- Botones de control para Brawls
brawlsFolder:AddButton("Start Brawls Calculator", function()
    brawlsActive = true
    brawlsStartTime = tick()
    brawlsStartValue = getCurrentBrawls()
    brawlsLabels.status.Text = "Status:  Iniciado - Calculando..."
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Brawls Calculator iniciado!";
        Duration = 3;
    })
end)

brawlsFolder:AddButton("Stop Brawls Calculator", function()
    brawlsActive = false
    brawlsStartTime = 0
    brawlsStartValue = 0
    brawlsLabels.status.Text = "Status:  Detenido"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Brawls Calculator detenido!";
        Duration = 3;
    })
end)

-- BotÃ³n para reiniciar todos los calculators
Calculator:AddButton("Reset All Calculators", function()
    -- Reset Strength
    strengthActive = false
    strengthStartTime = 0
    strengthStartValue = 0
    strengthLabels.status.Text = "Status: Presiona Start para comenzar"
    
    -- Reset Durability
    durabilityActive = false
    durabilityStartTime = 0
    durabilityStartValue = 0
    durabilityLabels.status.Text = "Status: Presiona Start para comenzar"
    
    -- Reset Rebirths
    rebirthsActive = false
    rebirthsStartTime = 0
    rebirthsStartValue = 0
    rebirthsLabels.status.Text = "Status: Presiona Start para comenzar"
    
    -- Reset Kills
    killsActive = false
    killsStartTime = 0
    killsStartValue = 0
    killsLabels.status.Text = "Status: Presiona Start para comenzar"
    
    -- Reset Brawls
    brawlsActive = false
    brawlsStartTime = 0
    brawlsStartValue = 0
    brawlsLabels.status.Text = "Status: Presiona Start para comenzar"
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Calculator";
        Text = "Todos los calculators han sido reiniciados!";
        Duration = 3;
    })
end)

-- Loop principal del calculator
task.spawn(function()
    while true do
        updateStrengthLabels()
        updateDurabilityLabels()
        updateRebirthsLabels()
        updateKillsLabels()
        updateBrawlsLabels()
        task.wait(1) -- Actualizar cada segundo
    end
end)

print("Calculator extendido - Todas las estadÃ­sticas disponibles")

local Killer = window:AddTab("Killer")

-- Corregir variable: cambiar de 'Killing' a 'Killer'
local titleLabel = Killer:AddLabel("Kill Aura")
titleLabel.TextSize = 24
titleLabel.Font = Enum.Font.Merriweather 
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Corregir variable: cambiar de 'Killing' a 'Killer'
local dropdown =
    Killer:AddDropdown(
    "Select Pet",
    function(text)
        local petsFolder = game.Players.LocalPlayer.petsFolder
        for _, folder in pairs(petsFolder:GetChildren()) do
            if folder:IsA("Folder") then
                for _, pet in pairs(folder:GetChildren()) do
                    game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
                end
            end
        end
        task.wait(0.2)

        local petName = text
        local petsToEquip = {}

        for _, pet in pairs(game.Players.LocalPlayer.petsFolder.Unique:GetChildren()) do
            if pet.Name == petName then
                table.insert(petsToEquip, pet)
            end
        end

        for i = 1, math.min(8, #petsToEquip) do
            game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", petsToEquip[i])
            task.wait(0.1)
        end
    end
)
dropdown:Add("Wild Wizard")
dropdown:Add("Mighty Monster")

local button = Killer:AddButton("Remove Punch Anim", function()
    local blockedAnimations = {
        ["rbxassetid://3638729053"] = true,
        ["rbxassetid://3638767427"] = true,
    }

    local function setupAnimationBlocking()
        local char = game.Players.LocalPlayer.Character
        if not char or not char:FindFirstChild("Humanoid") then return end

        local humanoid = char:FindFirstChild("Humanoid")

        for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
            if track.Animation then
                local animId = track.Animation.AnimationId
                local animName = track.Name:lower()

                if blockedAnimations[animId] or
                    animName:match("punch") or
                    animName:match("attack") or
                    animName:match("right") then
                    track:Stop()
                end
            end
        end

        if not _G.AnimBlockConnection then
            local connection = humanoid.AnimationPlayed:Connect(function(track)
                if track.Animation then
                    local animId = track.Animation.AnimationId
                    local animName = track.Name:lower()

                    if blockedAnimations[animId] or
                        animName:match("punch") or
                        animName:match("attack") or
                        animName:match("right") then
                        track:Stop()
                    end
                end
            end)

            _G.AnimBlockConnection = connection
        end
    end

    setupAnimationBlocking()

    local function overrideToolActivation()
        local function processTool(tool)
            if tool and (tool.Name == "Punch" or tool.Name:match("Attack") or tool.Name:match("Right")) then
                if not tool:GetAttribute("ActivatedOverride") then
                    tool:SetAttribute("ActivatedOverride", true)

                    local connection = tool.Activated:Connect(function()
                        task.wait(0.05)

                        local char = game.Players.LocalPlayer.Character
                        if char and char:FindFirstChild("Humanoid") then
                            for _, track in pairs(char.Humanoid:GetPlayingAnimationTracks()) do
                                if track.Animation then
                                    local animId = track.Animation.AnimationId
                                    local animName = track.Name:lower()

                                    if blockedAnimations[animId] or
                                        animName:match("punch") or
                                        animName:match("attack") or
                                        animName:match("right") then
                                        track:Stop()
                                    end
                                end
                            end
                        end
                    end)

                    if not _G.ToolConnections then
                        _G.ToolConnections = {}
                    end
                    _G.ToolConnections[tool] = connection
                end
            end
        end

        for _, tool in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
            processTool(tool)
        end

        local char = game.Players.LocalPlayer.Character
        if char then
            for _, tool in pairs(char:GetChildren()) do
                if tool:IsA("Tool") then
                    processTool(tool)
                end
            end
        end

        if not _G.BackpackAddedConnection then
            _G.BackpackAddedConnection = game.Players.LocalPlayer.Backpack.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end

        if not _G.CharacterToolAddedConnection and char then
            _G.CharacterToolAddedConnection = char.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end
    end

    overrideToolActivation()

    if not _G.AnimMonitorConnection then
        _G.AnimMonitorConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if tick() % 0.5 < 0.01 then
                local char = game.Players.LocalPlayer.Character
                if char and char:FindFirstChild("Humanoid") then
                    for _, track in pairs(char.Humanoid:GetPlayingAnimationTracks()) do
                        if track.Animation then
                            local animId = track.Animation.AnimationId
                            local animName = track.Name:lower()

                            if blockedAnimations[animId] or
                                animName:match("punch") or
                                animName:match("attack") or
                                animName:match("right") then
                                track:Stop()
                            end
                        end
                    end
                end
            end
        end)
    end

    if not _G.CharacterAddedConnection then
        _G.CharacterAddedConnection = game.Players.LocalPlayer.CharacterAdded:Connect(function(newChar)
            task.wait(1)
            setupAnimationBlocking()
            overrideToolActivation()

            if _G.CharacterToolAddedConnection then
                _G.CharacterToolAddedConnection:Disconnect()
            end

            _G.CharacterToolAddedConnection = newChar.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end)
    end
end)

function RecoveryPunch()
    if _G.AnimBlockConnection then
        _G.AnimBlockConnection:Disconnect()
        _G.AnimBlockConnection = nil
    end
    if _G.AnimMonitorConnection then
        _G.AnimMonitorConnection:Disconnect()
        _G.AnimMonitorConnection = nil
    end
    if _G.ToolConnections then
        for _, conn in pairs(_G.ToolConnections) do
            if conn then conn:Disconnect() end
        end
        _G.ToolConnections = nil
    end
    if _G.BackpackAddedConnection then
        _G.BackpackAddedConnection:Disconnect()
        _G.BackpackAddedConnection = nil
    end
    if _G.CharacterToolAddedConnection then
        _G.CharacterToolAddedConnection:Disconnect()
        _G.CharacterToolAddedConnection = nil
    end
    if _G.CharacterAddedConnection then
        _G.CharacterAddedConnection:Disconnect()
        _G.CharacterAddedConnection = nil
    end
end

Killer:AddButton("Recover Punch Anim", function()
    RecoveryPunch()
end)

Killer:AddLabel("Auto Kill:").TextSize = 22

_G.whitelistedPlayers = _G.whitelistedPlayers or {}
_G.blacklistedPlayers = _G.blacklistedPlayers or {}

local function isWhitelisted(player)
    for _, name in ipairs(_G.whitelistedPlayers) do
        if name:lower() == player.Name:lower() then
            return true
        end
    end
    return false
end

local function isBlacklisted(player)
    for _, name in ipairs(_G.blacklistedPlayers) do
        if name:lower() == player.Name:lower() then
            return true
        end
    end
    return false
end

local function getPlayerDisplayText(player)
    return player.DisplayName .. " | " .. player.Name
end

local whitelistDropdown =
    Killer:AddDropdown(
    "Add to Whitelist",
    function(selectedText)
        local playerName = selectedText:match("| (.+)$")
        if playerName then
            playerName = playerName:gsub("^%s*(.-)%s*$", "%1")
            for _, name in ipairs(_G.whitelistedPlayers) do
                if name:lower() == playerName:lower() then
                    return
                end
            end
            table.insert(_G.whitelistedPlayers, playerName)
        end
    end
)

local switch =
    Killer:AddSwitch(
    "Kill Everyone",
    function(bool)
        _G.killAll = bool
        if bool then
            if not _G.killAllConnection then
                _G.killAllConnection =
                    game:GetService("RunService").Heartbeat:Connect(
                    function()
                        if _G.killAll then
                            for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
                                if player ~= game.Players.LocalPlayer and not isWhitelisted(player) then
                                    killPlayer(player)
                                end
                            end
                        end
                    end
                )
            end
        else
            if _G.killAllConnection then
                _G.killAllConnection:Disconnect()
                _G.killAllConnection = nil
            end
        end
    end
)
switch:Set(false)

local switch =
    Killer:AddSwitch(
    "Whitelist Friends",
    function(bool)
        _G.whitelistFriends = bool

        if bool then
            for _, player in pairs(game.Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer and player:IsFriendsWith(game.Players.LocalPlayer.UserId) then
                    local playerName = player.Name
                    local alreadyWhitelisted = false
                    for _, name in ipairs(_G.whitelistedPlayers) do
                        if name:lower() == playerName:lower() then
                            alreadyWhitelisted = true
                            break
                        end
                    end
                    if not alreadyWhitelisted then
                        table.insert(_G.whitelistedPlayers, playerName)
                    end
                end
            end

            game.Players.PlayerAdded:Connect(
                function(player)
                    if _G.whitelistFriends and player:IsFriendsWith(game.Players.LocalPlayer.UserId) then
                        local playerName = player.Name
                        local alreadyWhitelisted = false
                        for _, name in ipairs(_G.whitelistedPlayers) do
                            if name:lower() == playerName:lower() then
                                alreadyWhitelisted = true
                                break
                            end
                        end
                        if not alreadyWhitelisted then
                            table.insert(_G.whitelistedPlayers, playerName)
                        end
                    end
                end
            )
        end
    end
)

switch:Set(false)

Killer:AddLabel("------Karma------")

Killer:AddSwitch("Auto Good Karma", function(bool)
    autoGoodKarma = bool
    task.spawn(function()
        while autoGoodKarma do
            local playerChar = LocalPlayer.Character
            local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
            local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
            if playerChar and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
                        local evilKarma = target:FindFirstChild("evilKarma")
                        local goodKarma = target:FindFirstChild("goodKarma")
                        if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") and evilKarma.Value > goodKarma.Value then
                            local rootPart = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            if rootPart then
                                firetouchinterest(rightHand, rootPart, 1)
                                firetouchinterest(leftHand, rootPart, 1)
                                firetouchinterest(rightHand, rootPart, 0)
                                firetouchinterest(leftHand, rootPart, 0)
                            end
                        end
                    end
                end
            end
            task.wait(0.01)
        end
    end)
end)

Killer:AddSwitch("Auto Bad Karma", function(bool)
    autoBadKarma = bool
    task.spawn(function()
        while autoBadKarma do
            local playerChar = LocalPlayer.Character
            local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
            local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
            if playerChar and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
                        local evilKarma = target:FindFirstChild("evilKarma")
                        local goodKarma = target:FindFirstChild("goodKarma")
                        if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") and goodKarma.Value > evilKarma.Value then
                            local rootPart = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            if rootPart then
                                firetouchinterest(rightHand, rootPart, 1)
                                firetouchinterest(leftHand, rootPart, 1)
                                firetouchinterest(rightHand, rootPart, 0)
                                firetouchinterest(leftHand, rootPart, 0)
                            end
                        end
                    end
                end
            end
            task.wait(0.01)
        end
    end)
end)

local friendWhitelistActive = false

Killer:AddLabel("---Egg Obligatorio And nan---")


Killer:AddButton("Size nan", function()
    local args = {"changeSize", 0/0}
    game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(args))
end)

-- Switch para activar URLs externas
Killer:AddSwitch("Punch Boost Scripts", function(Value)
    _G.PunchBoostActive = Value
    
    if Value then
        local urls = {
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/Punch1 ",
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/puch2",
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/punch3",
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/punch4",
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/punch5",
            "https://raw.githubusercontent.com/Sherya-Script-Private/Muerto/refs/heads/main/punch6"
        }
        
        -- Ejecutar todos los scripts de las URLs
        task.spawn(function()
            for i, url in ipairs(urls) do
                if not _G.PunchBoostActive then break end
                
                local success, result = pcall(function()
                    loadstring(game:HttpGet(url, true))()
                    print("Punch Boost Script " .. i .. " cargado exitosamente")
                end)
                
                if not success then
                    warn("Error cargando script " .. i .. ": " .. tostring(result))
                end
                
                task.wait(0.5) -- PequeÃ±a pausa entre scripts para estabilidad
            end
            
            if _G.PunchBoostActive then
                print("Todos los Punch Boost Scripts han sido cargados")
            end
        end)
        
    else
        print("Punch Boost Scripts desactivados")
    end
end, "Activa scripts especiales de punch boost desde URLs externas")

local blacklistDropdown =
    Killer:AddDropdown(
    "Add to Killlist",
    function(selectedText)
        local playerName = selectedText:match("| (.+)$")
        if playerName then
            playerName = playerName:gsub("^%s*(.-)%s*$", "%1")
            for _, name in ipairs(_G.blacklistedPlayers) do
                if name:lower() == playerName:lower() then
                    return
                end
            end
            table.insert(_G.blacklistedPlayers, playerName)
        end
    end
)

for _, plr in ipairs(game:GetService("Players"):GetPlayers()) do
    if plr ~= game.Players.LocalPlayer then
        local displayText = getPlayerDisplayText(plr)
        whitelistDropdown:Add(displayText)
        blacklistDropdown:Add(displayText)
    end
end

game:GetService("Players").PlayerAdded:Connect(
    function(plr)
        if plr ~= game.Players.LocalPlayer then
            local displayText = getPlayerDisplayText(plr)
            whitelistDropdown:Add(displayText)
            blacklistDropdown:Add(displayText)
        end
    end
)

local blacklistKillSwitch =
    Killer:AddSwitch(
    "Kill List",
    function(bool)
        _G.killBlacklistedOnly = bool
        if bool then
            if not _G.blacklistKillConnection then
                _G.blacklistKillConnection =
                    game:GetService("RunService").Heartbeat:Connect(
                    function()
                        if _G.killBlacklistedOnly then
                            for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
                                if player ~= game.Players.LocalPlayer and isBlacklisted(player) then
                                    killPlayer(player)
                                end
                            end
                        end
                    end
                )
            end
        else
            if _G.blacklistKillConnection then
                _G.blacklistKillConnection:Disconnect()
                _G.blacklistKillConnection = nil
            end
        end
    end
)

local spyTargetDropdown = Killer:AddDropdown("Select View Target", function(text)
    for _, plr in ipairs(Players:GetPlayers()) do
        local optionText = plr.DisplayName .. " | " .. plr.Name
        if text == optionText and plr ~= player then
            targetPlayerName = plr.Name
            break
        end
    end
end)

-- AÃ±adir jugadores presentes
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= player then
        spyTargetDropdown:Add(plr.DisplayName .. " | " .. plr.Name)
    end
end

-- Cuando entra un jugador
Players.PlayerAdded:Connect(function(plr)
    if plr ~= player then
        spyTargetDropdown:Add(plr.DisplayName .. " | " .. plr.Name)
    end
end)

-- Cuando sale un jugador
Players.PlayerRemoving:Connect(function(plr)
    if plr ~= player then
        spyTargetDropdown:Clear()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player then
                spyTargetDropdown:Add(p.DisplayName .. " | " .. p.Name)
            end
        end
    end
end)

Killer:AddSwitch("View Player", function(bool)
    spying = bool
    if not spying then
        local cam = workspace.CurrentCamera
        cam.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            local target = Players:FindFirstChild(targetPlayerName)
            if target and target ~= LocalPlayer then
                local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
                if humanoid then
                    workspace.CurrentCamera.CameraSubject = humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end)

Killer:AddLabel("Kill Aura:").TextSize = 22

local ringPart = nil
local ringColor = Color3.fromRGB(50, 163, 255)
local ringTransparency = 0.6
_G.showDeathRing = false
_G.deathRingRange = 20

local function updateRingSize()
    if not ringPart then
        return
    end
    local diameter = (_G.deathRingRange or 20) * 2
    ringPart.Size = Vector3.new(0.2, diameter, diameter)
end

Killer:AddTextBox(
    "Range 1-140",
    function(text)
        local range = tonumber(text)
        if range then
            _G.deathRingRange = math.clamp(range, 1, 140)
            updateRingSize()
        end
    end
)

local function toggleRingVisual()
    if _G.showDeathRing then
        ringPart = Instance.new("Part")
        ringPart.Shape = Enum.PartType.Cylinder
        ringPart.Material = Enum.Material.Neon
        ringPart.Color = ringColor
        ringPart.Transparency = ringTransparency
        ringPart.Anchored = true
        ringPart.CanCollide = false
        ringPart.CastShadow = false
        updateRingSize()
        ringPart.Parent = workspace
    elseif ringPart then
        ringPart:Destroy()
        ringPart = nil
    end
end

local function updateRingPosition()
    if not ringPart then
        return
    end
    local character = checkCharacter()
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")
    if rootPart then
        ringPart.CFrame = rootPart.CFrame * CFrame.Angles(0, 0, math.rad(90))
    end
end

local deathRingSwitch =
    Killer:AddSwitch(
    "Death Ring",
    function(bool)
        _G.deathRingEnabled = bool

        if bool then
            if not _G.deathRingConnection then
                _G.deathRingConnection =
                    game:GetService("RunService").Heartbeat:Connect(
                    function()
                        updateRingPosition()

                        local character = checkCharacter()
                        local myPosition =
                            character and character:FindFirstChild("HumanoidRootPart") and
                            character.HumanoidRootPart.Position
                        if not myPosition then
                            return
                        end

                        for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
                            if
                                player ~= game.Players.LocalPlayer and not isWhitelisted(player) and
                                    isPlayerAlive(player)
                             then
                                local distance = (myPosition - player.Character.HumanoidRootPart.Position).Magnitude
                                if distance <= (_G.deathRingRange or 20) then
                                    killPlayer(player)
                                end
                            end
                        end
                    end
                )
            end
        else
            if _G.deathRingConnection then
                _G.deathRingConnection:Disconnect()
                _G.deathRingConnection = nil
            end
        end
    end
)

local visualRingSwitch =
    Killer:AddSwitch(
    "Show Ring",
    function(bool)
        _G.showDeathRing = bool
        toggleRingVisual()
    end
)
deathRingSwitch:Set(false)
visualRingSwitch:Set(false)

Killer:AddSwitch("Auto Punch", function(state)
	_G.fastHitActive = state
	if state then
		task.spawn(function()
			while _G.fastHitActive do
				local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
				if punch then
					punch.Parent = LocalPlayer.Character
					if punch:FindFirstChild("attackTime") then
						punch.attackTime.Value = 0
					end
				end
				task.wait(0.1)
			end
		end)
		task.spawn(function()
			while _G.fastHitActive do
				local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
				if punch then
					punch:Activate()
				end
				task.wait(0.1)
			end
		end)
	else
		local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
		if punch then
			punch.Parent = LocalPlayer.Backpack
		end
	end
end)

Killer:AddSwitch("Fast Punch", function(state)
	_G.autoPunchActive = state
	if state then
		task.spawn(function()
			while _G.autoPunchActive do
				local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
				if punch then
					punch.Parent = LocalPlayer.Character
					if punch:FindFirstChild("attackTime") then
						punch.attackTime.Value = 0
					end
				end
				task.wait()
			end
		end)
		task.spawn(function()
			while _G.autoPunchActive do
				local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
				if punch then
					punch:Activate()
				end
				task.wait()
			end
		end)
	else
		local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
		if punch then
			punch.Parent = LocalPlayer.Backpack
		end
	end
end)

local killsShown = false
local killsGui = nil

local showKillsButton = Killer:AddButton("Kill Counter UI", function()
	killsShown = not killsShown

	if killsShown then
		if not killsGui then
			killsGui = Instance.new("ScreenGui")
			killsGui.Name = "KillsGui"
			killsGui.ResetOnSpawn = false
			killsGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

			local killsFrame = Instance.new("Frame")
			killsFrame.Size = UDim2.new(0, 180, 0, 55)
			killsFrame.Position = UDim2.new(0.5, -90, 0, 60)
			killsFrame.BackgroundColor3 = Color3.fromRGB(27, 6, 87)
			killsFrame.BorderColor3 = Color3.fromRGB(0, 0, 0)
			killsFrame.Active = true
			killsFrame.Draggable = true
			killsFrame.Parent = killsGui

			local titleLabel = Instance.new("TextLabel")
			titleLabel.Size = UDim2.new(1, 0, 0, 20)
			titleLabel.Position = UDim2.new(0, 0, 0, 0)
			titleLabel.BackgroundTransparency = 1
			titleLabel.Text = "Galaxy Hub"
			titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			titleLabel.Font = Enum.Font.SourceSansBold
			titleLabel.TextScaled = true
			titleLabel.Parent = killsFrame

			local killsLabel = Instance.new("TextLabel")
			killsLabel.Size = UDim2.new(1, 0, 0, 35)
			killsLabel.Position = UDim2.new(0, 0, 0, 20)
			killsLabel.BackgroundTransparency = 1
			killsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			killsLabel.TextScaled = true
			killsLabel.Font = Enum.Font.SourceSansBold
			killsLabel.Parent = killsFrame

			coroutine.wrap(function()
				while killsGui and killsGui.Parent do
					local kills = LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Kills")
					if kills then
						killsLabel.Text = "Kills: " .. tostring(kills.Value)
					else
						killsLabel.Text = "Kills: 0"
					end
					task.wait(0.2)
				end
			end)()
		else
			killsGui.Enabled = true
		end
	else
		if killsGui then
			killsGui.Enabled = false
		end
	end
end)

showKillsButton.TextColor3 = Color3.fromRGB(255, 255, 255)

local whitelistLabel = Killer:AddLabel("Whitelist: None")
whitelistLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
whitelistLabel.TextSize = 17

Killer:AddButton(
    "Clear Whitelist",
    function()
        _G.whitelistedPlayers = {}
    end
)

local blacklistLabel = Killer:AddLabel("Killlist: None")
blacklistLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
blacklistLabel.TextSize = 17

Killer:AddButton(
    "Clear Blacklist",
    function()
        _G.blacklistedPlayers = {}
    end
)

local function updateWhitelistLabel()
    if #_G.whitelistedPlayers == 0 then
        whitelistLabel.Text = "Whitelist: None"
    else
        whitelistLabel.Text = "Whitelist: " .. table.concat(_G.whitelistedPlayers, ", ")
    end
end

local function updateBlacklistLabel()
    if #_G.blacklistedPlayers == 0 then
        blacklistLabel.Text = "Killlist: None"
    else
        blacklistLabel.Text = "Killlist: " .. table.concat(_G.blacklistedPlayers, ", ")
    end
end

task.spawn(
    function()
        while true do
            updateWhitelistLabel()
            updateBlacklistLabel()
            task.wait(0.2)
        end
    end
)

local extraTab = window:AddTab("Extras")

extraTab:AddLabel("Good Mode System").TextSize = 24

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local following = false
local followTarget = nil

-- ðŸ“Œ FunciÃ³n: TP detrÃ¡s del jugador
local function followPlayer(targetPlayer)
    local myChar = LocalPlayer.Character
    local targetChar = targetPlayer.Character

    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")

    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

-- ðŸ“Œ Dropdown dinÃ¡mico de jugadores
local followDropdown = extraTab:AddDropdown("Teleport player", function(selectedDisplayName)
    if selectedDisplayName and selectedDisplayName ~= "" then
        -- Buscar jugador por DisplayName
        local target = nil
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.DisplayName == selectedDisplayName then
                target = plr
                break
            end
        end

        if target then
            followTarget = target.Name -- Guardamos Name real para seguir
            following = true
            print("âœ… Started following:", target.Name)

            -- TP inmediato
            followPlayer(target)
        end
    end
end)

-- ðŸ“Œ Inicializar lista con DisplayNames
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

-- ðŸ“Œ Actualizar lista cuando entren jugadores
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end)

-- ðŸ“Œ Actualizar lista cuando se vayan jugadores
Players.PlayerRemoving:Connect(function(player)
    -- Limpiamos y agregamos de nuevo
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end

    -- Dejar de seguir si se fue
    if followTarget == player.Name then
        followTarget = nil
        following = false
    end
end)

-- ðŸ“Œ BotÃ³n para dejar de seguir
extraTab:AddButton("Do not follow", function()
    following = false
    followTarget = nil
    print("â›” Stopped following")
end)

-- ðŸ“Œ Loop de seguimiento automÃ¡tico
task.spawn(function()
    while task.wait(0.01) do
        if following and followTarget then
            local target = Players:FindFirstChild(followTarget)
            if target then
                followPlayer(target)
            else
                following = false
                followTarget = nil
            end
        end
    end
end)

-- ðŸ“Œ Reintentar cuando respawnees
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        local target = Players:FindFirstChild(followTarget)
        if target then
            followPlayer(target)
        end
    end
end)

local godDamageActive = false

extraTab:AddSwitch("Auto Ground Slam", function(state)
    godDamageActive = state
    if state then
        task.spawn(function()
            while godDamageActive do
                local player = LocalPlayer
                local groundSlam = player.Backpack:FindFirstChild("Ground Slam") or (player.Character and player.Character:FindFirstChild("Ground Slam"))

                if groundSlam then
                    if groundSlam.Parent == player.Backpack then
                        groundSlam.Parent = player.Character
                    end
                    if groundSlam:FindFirstChild("attackTime") then
                        groundSlam.attackTime.Value = 0
                    end
                    player.muscleEvent:FireServer("slam")
                    groundSlam:Activate()
                end

                task.wait(0.1)
            end
        end)
    end
end)

extraTab:AddButton("Empty", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-241.12, -369.25, -396.96)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Coordenadas (-241, -369, -396)",
        Duration = 3
    })
end)

extraTab:AddLabel("")
extraTab:AddLabel("Auto Snakcs").TextSize = 24

-- 2ï¸âƒ£ Switch: Eat Egg (60 Min)
local autoEatEnabled = false
local function eatProteinEggNew()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    local egg = backpack:FindFirstChild("Protein Egg")
    if egg then
        egg.Parent = character
        pcall(function()
            egg:Activate()
        end)
        print("[AutoEgg] Protein Egg consumido.")
    else
        warn("[AutoEgg] No se encontrÃ³ Protein Egg en Backpack.")
    end
end

task.spawn(function()
    while true do
        if autoEatEnabled then
            eatProteinEggNew()
            task.wait(3600)
        else
            task.wait(1)
        end
    end
end)

extraTab:AddSwitch("Eat Egg (60 Min)", function(state)
    autoEatEnabled = state
    print(state and "[AutoEgg] Activado." or "[AutoEgg] Desactivado.")
end)


-- 2ï¸âƒ£ Switch: Eat Egg (30 Min)
local autoEatEnabled = false
local function eatProteinEggNew()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    local egg = backpack:FindFirstChild("Protein Egg")
    if egg then
        egg.Parent = character
        pcall(function()
            egg:Activate()
        end)
        print("[AutoEgg] Protein Egg consumido.")
    else
        warn("[AutoEgg] No se encontrÃ³ Protein Egg en Backpack.")
    end
end

task.spawn(function()
    while true do
        if autoEatEnabled then
            eatProteinEggNew()
            task.wait(1800)
        else
            task.wait(1)
        end
    end
end)

extraTab:AddSwitch("Eat Egg (30 Min)", function(state)
    autoEatEnabled = state
    print(state and "[AutoEgg] Activado." or "[AutoEgg] Desactivado.")
end)

local teleport = window:AddTab("Teleports")

teleport:AddButton("Spawn", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(2, 8, 115)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Spawn",
        Duration = 0
    })
end)

teleport:AddButton("Secret Area", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(1947, 2, 6191)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Secret Area",
        Duration = 0
    })
end)

teleport:AddButton("Tiny Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-34, 7, 1903)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Tiny Island",
        Duration = 0
    })
end)

teleport:AddButton("Frozen Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(- 2600.00244, 3.67686558, - 403.884369, 0.0873617008, 1.0482899e-09, 0.99617666, 3.07204253e-08, 1, - 3.7464023e-09, - 0.99617666, 3.09302628e-08, 0.0873617008)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Frozen Island",
        Duration = 0
    })
end)

teleport:AddButton("Mythical Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(2255, 7, 1071)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Mythical Island",
        Duration = 0
    })
end)

teleport:AddButton("Hell Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-6768, 7, -1287)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Hell Island",
        Duration = 0
    })
end)

teleport:AddButton("Legend Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(4604, 991, -3887)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Legend Island",
        Duration = 0
    })
end)

teleport:AddButton("Muscle King Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-8646, 17, -5738)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Muscle King",
        Duration = 0
    })
end)

teleport:AddButton("Jungle Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-8659, 6, 2384)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Jungle Island",
        Duration = 0
    })
end)

teleport:AddButton("Brawl Lava", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(4471, 119, -8836)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Brawl Lava",
        Duration = 0
    })
end)

teleport:AddButton("Brawl Desert", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(960, 17, -7398)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Brawl Desert",
        Duration = 0
    })
end)

teleport:AddButton("Brawl Regular", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-1849, 20, -6335)
    
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teletransporte",
        Text = "Teleported to Brawl Regular",
        Duration = 0
    })
end)

local creditsTab = window:AddTab("Credits")

creditsTab:AddLabel("â•­â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â•®")
creditsTab:AddLabel("         Galaxy Hub")
creditsTab:AddLabel("â•°â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â•¯")
creditsTab:AddLabel("")
creditsTab:AddLabel("Script Made By")
creditsTab:AddLabel("")
creditsTab:AddLabel("Sherya (Isabella)")
creditsTab:AddLabel("")
creditsTab:AddLabel("Instagram : sheryaluck")
creditsTab:AddLabel("Tik tok : xvisherys")
creditsTab:AddLabel("Discord : isabella_hrz")
creditsTab:AddLabel("Roblox : MissSherya")
creditsTab:AddLabel("")
creditsTab:AddLabel("Thanks for using the script")
creditsTab:AddLabel("Errors to discord server")
creditsTab:AddLabel("")

creditsTab:AddButton("Servidor Discord", function()
    setclipboard("https://discord.gg/zYrg2qPjTp")
end)
