local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Vim = game:GetService("VirtualInputManager")

local isRecording = false
local frames = {}

-- Inicia gravação
local function startRecording()
    isRecording = true
    frames = {}
end

-- Para gravação
local function stopRecording()
    isRecording = false
end

-- Reproduz
local function playRecording()
    if #frames == 0 then return end

    task.spawn(function()
        local startTime = frames[1].Time
        for i, frame in ipairs(frames) do
            local delay = frame.Time - (frames[i-1] and frames[i-1].Time or startTime)
            task.wait(delay)

            if frame.Type == "Camera" then
                workspace.CurrentCamera.CFrame = frame.CFrame
            elseif frame.Type == "Click" then
                -- simula toque/click
                Vim:SendMouseButtonEvent(frame.Position.X, frame.Position.Y, 0, true, game, 1)
                task.wait(0.05)
                Vim:SendMouseButtonEvent(frame.Position.X, frame.Position.Y, 0, false, game, 1)
            end
        end
    end)
end

-- Loop de captura (câmera)
RunService.RenderStepped:Connect(function()
    if isRecording then
        table.insert(frames, {
            Type = "Camera",
            CFrame = workspace.CurrentCamera.CFrame,
            Time = tick()
        })
    end
end)

-- Captura toques/clicks
UserInputService.InputBegan:Connect(function(input, gpe)
    if isRecording and not gpe then
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            table.insert(frames, {
                Type = "Click",
                Position = input.Position,
                Time = tick()
            })
        end
    end
end)
