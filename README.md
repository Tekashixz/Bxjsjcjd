local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Vim = game:GetService("VirtualInputManager")

-- Variáveis
local isRecording = false
local recordedFrames = {}
local savedRecordings = {}
local menuVisible = true

-- Início e parada de gravação
local function startRecording()
    isRecording = true
    recordedFrames = {}
end

local function stopRecording()
    isRecording = false
end

-- Reprodução
local function playRecording(frames, loop)
    if not frames or #frames == 0 then return end
    task.spawn(function()
        repeat
            for i, frame in ipairs(frames) do
                local waitTime = 0
                if i > 1 then
                    waitTime = frame.Time - frames[i-1].Time
                end
                task.wait(waitTime)
                if frame.Type == "Camera" then
                    Camera.CFrame = frame.CFrame
                elseif frame.Type == "Click" then
                    -- Simula click/toque
                    Vim:SendMouseButtonEvent(frame.Position.X, frame.Position.Y, 0, true, game, 1)
                    task.wait(0.05)
                    Vim:SendMouseButtonEvent(frame.Position.X, frame.Position.Y, 0, false, game, 1)
                end
            end
        until not loop
    end)
end

-- Captura câmera
RunService.RenderStepped:Connect(function()
    if isRecording then
        table.insert(recordedFrames, {
            Type = "Camera",
            CFrame = Camera.CFrame,
            Time = tick()
        })
    end
end)

-- Captura clicks/toques
UserInputService.InputBegan:Connect(function(input, gpe)
    if isRecording and not gpe then
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            table.insert(recordedFrames, {
                Type = "Click",
                Position = input.Position,
                Time = tick()
            })
        end
    end
end)

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 350, 0, 500)
Frame.Position = UDim2.new(0.5, -175, 0.5, -250)
Frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 14)
UICorner.Parent = Frame

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1,0,0,40)
Title.Position = UDim2.new(0,0,0,0)
Title.BackgroundColor3 = Color3.fromRGB(50,50,50)
Title.Text = "🎥 Recorder Pro"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
local cornerTitle = Instance.new("UICorner")
cornerTitle.CornerRadius = UDim.new(0,12)
cornerTitle.Parent = Title

-- Botão minimizar
local MinBtn = Instance.new("TextButton")
MinBtn.Parent = Title
MinBtn.Size = UDim2.new(0,30,0,30)
MinBtn.Position = UDim2.new(1,-35,0.5,-15)
MinBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.new(1,1,1)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 20
local cornerMin = Instance.new("UICorner")
cornerMin.CornerRadius = UDim.new(0,8)
cornerMin.Parent = MinBtn

-- Botão flutuante
local FloatBtn = Instance.new("TextButton")
FloatBtn.Parent = ScreenGui
FloatBtn.Size = UDim2.new(0,60,0,60)
FloatBtn.Position = UDim2.new(1,-70,1,-120)
FloatBtn.BackgroundColor3 = Color3.fromRGB(45,45,45)
FloatBtn.Text = "⚙"
FloatBtn.TextColor3 = Color3.new(1,1,1)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 28
FloatBtn.Visible = false
local cornerFloat = Instance.new("UICorner")
cornerFloat.CornerRadius = UDim.new(1,0)
cornerFloat.Parent = FloatBtn

-- Função helper botões
local function createButton(parent,text,posY,callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9,0,0,40)
    btn.Position = UDim2.new(0.05,0,0,posY)
    btn.BackgroundColor3 = Color3.fromRGB(55,55,55)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,8)
    corner.Parent = btn
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Botões principais
createButton(Frame,"▶ Gravar",50,function() startRecording() end)
createButton(Frame,"■ Parar",100,function() stopRecording() end)
createButton(Frame,"▶ Play",150,function() playRecording(recordedFrames,false) end)
createButton(Frame,"⟳ Loop",200,function() playRecording(recordedFrames,true) end)

-- Input para salvar
local SaveBox = Instance.new("TextBox")
SaveBox.Parent = Frame
SaveBox.Size = UDim2.new(0.9,0,0,35)
SaveBox.Position = UDim2.new(0.05,0,0,250)
SaveBox.PlaceholderText = "Nome da gravação..."
SaveBox.TextColor3 = Color3.new(1,1,1)
SaveBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
local cornerInput = Instance.new("UICorner")
cornerInput.CornerRadius = UDim.new(0,8)
cornerInput.Parent = SaveBox

-- Botão salvar
local SaveBtn = createButton(Frame,"💾 Salvar",300,function()
    local name = SaveBox.Text
    if name ~= "" and #recordedFrames > 0 then
        savedRecordings[name] = table.clone(recordedFrames)
        SaveBox.Text = ""
        refreshList()
    end
end)

-- Lista de gravações
local Scroll = Instance.new("ScrollingFrame")
Scroll.Parent = Frame
Scroll.Size = UDim2.new(0.9,0,0,150)
Scroll.Position = UDim2.new(0.05,0,0,350)
Scroll.BackgroundColor3 = Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness = 6
local UIList = Instance.new("UIListLayout")
UIList.Parent = Scroll
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Padding = UDim.new(0,5)

-- Atualizar lista de gravações
function refreshList()
    Scroll:ClearAllChildren()
    UIList.Parent = Scroll
    local y = 0
    for name, frames in pairs(savedRecordings) do
        local item = Instance.new("Frame")
        item.Size = UDim2.new(1,0,0,40)
        item.BackgroundTransparency = 1
        item.Parent = Scroll

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6,0,1,0)
        label.Position = UDim2.new(0,0,0,0)
        label.Text = name
        label.TextColor3 = Color3.new(1,1,1)
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.GothamBold
        label.TextSize = 16
        label.Parent = item

        local playBtn = Instance.new("TextButton")
        playBtn.Size = UDim2.new(0.13,0,1,0)
        playBtn.Position = UDim2.new(0.6,0,0,0)
        playBtn.Text = "▶"
        playBtn.TextColor3 = Color3.new(1,1,1)
        playBtn.BackgroundColor3 = Color3.fromRGB(70,120,70)
        local c1 = Instance.new("UICorner")
        c1.CornerRadius = UDim.new(0,6)
        c1.Parent = playBtn
        playBtn.Parent = item
        playBtn.MouseButton1Click:Connect(function() playRecording(frames,false) end)

        local loopBtn = Instance.new("TextButton")
        loopBtn.Size = UDim2.new(0.13,0,1,0)
        loopBtn.Position = UDim2.new(0.73,0,0,0)
        loopBtn.Text = "⟳"
        loopBtn.TextColor3 = Color3.new(1,1,1)
        loopBtn.BackgroundColor3 = Color3.fromRGB(120,70,70)
        local c2 = Instance.new("UICorner")
        c2.CornerRadius = UDim.new(0,6)
        c2.Parent = loopBtn
        loopBtn.Parent = item
        loopBtn.MouseButton1Click:Connect(function() playRecording(frames,true) end)

        local delBtn = Instance.new("TextButton")
        delBtn.Size = UDim2.new(0.1,0,1,0)
        delBtn.Position = UDim2.new(0.86,0,0,0)
        delBtn.Text = "❌"
        delBtn.TextColor3 = Color3.new(1,1,1)
        delBtn.BackgroundColor3 = Color3.fromRGB(150,50,50)
        local c3 = Instance.new("UICorner")
        c3.CornerRadius = UDim.new(0,6)
        c3.Parent = delBtn
        delBtn.Parent = item
        delBtn.MouseButton1Click:Connect(function()
            savedRecordings[name] = nil
            refreshList()
        end)

        y = y + 45
    end
    Scroll.CanvasSize = UDim2.new(0,0,y)
end

-- Botões minimizar/flutuante
MinBtn.MouseButton1Click:Connect(function()
    menuVisible = false
    Frame.Visible = false
    FloatBtn.Visible = true
end)
FloatBtn.MouseButton1Click:Connect(function()
    menuVisible = true
    Frame.Visible = true
    FloatBtn.Visible = false
    refreshList()
end)
