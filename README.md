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

-- Funções gravação
local function startRecording()
    isRecording = true
    recordedFrames = {}
end

local function stopRecording()
    isRecording = false
end

local function playRecording(frames, loop)
    if not frames or #frames == 0 then return end
    task.spawn(function()
        repeat
            for i, frame in ipairs(frames) do
                local waitTime = 0
                if i > 1 then waitTime = frame.Time - frames[i-1].Time end
                task.wait(waitTime)
                if frame.Type == "Camera" then
                    Camera.CFrame = frame.CFrame
                elseif frame.Type == "Click" then
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
        table.insert(recordedFrames,{
            Type = "Camera",
            CFrame = Camera.CFrame,
            Time = tick()
        })
    end
end)

-- Captura cliques/toques
UserInputService.InputBegan:Connect(function(input, gpe)
    if isRecording and not gpe then
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            table.insert(recordedFrames,{
                Type = "Click",
                Position = input.Position,
                Time = tick()
            })
        end
    end
end)

--// GUI Premium
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal com sombra e gradiente
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 360, 0, 520)
Frame.Position = UDim2.new(0.5, -180, 0.5, -260)
Frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0,16)
UICorner.Parent = Frame

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(80, 80, 80)
UIStroke.Parent = Frame

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1,0,0,50)
Title.Position = UDim2.new(0,0,0,0)
Title.BackgroundColor3 = Color3.fromRGB(45,45,45)
Title.Text = "🎥 Recorder Pro Premium"
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.TextSize = 24
Title.Font = Enum.Font.GothamBold
local cornerTitle = Instance.new("UICorner")
cornerTitle.CornerRadius = UDim.new(0,16)
cornerTitle.Parent = Title

-- Botão minimizar
local MinBtn = Instance.new("TextButton")
MinBtn.Parent = Title
MinBtn.Size = UDim2.new(0,36,0,36)
MinBtn.Position = UDim2.new(1,-42,0.5,-18)
MinBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.fromRGB(255,255,255)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 24
local cornerMin = Instance.new("UICorner")
cornerMin.CornerRadius = UDim.new(0,10)
cornerMin.Parent = MinBtn

-- Botão flutuante
local FloatBtn = Instance.new("TextButton")
FloatBtn.Parent = ScreenGui
FloatBtn.Size = UDim2.new(0,64,0,64)
FloatBtn.Position = UDim2.new(1,-80,1,-140)
FloatBtn.BackgroundColor3 = Color3.fromRGB(45,45,45)
FloatBtn.Text = "⚙"
FloatBtn.TextColor3 = Color3.fromRGB(255,255,255)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 32
FloatBtn.Visible = false
local cornerFloat = Instance.new("UICorner")
cornerFloat.CornerRadius = UDim.new(1,0)
cornerFloat.Parent = FloatBtn

-- Função helper para criar botões premium
local function createButton(parent,text,posY,callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9,0,0,44)
    btn.Position = UDim2.new(0.05,0,0,posY)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    btn.AutoButtonColor = true
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,12)
    corner.Parent = btn

    -- Animação simples ao clicar
    btn.MouseButton1Click:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(90,90,90)
        task.wait(0.08)
        btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
        callback()
    end)
    return btn
end

-- Botões principais
createButton(Frame,"▶ Gravar",60,function() startRecording() end)
createButton(Frame,"■ Parar",120,function() stopRecording() end)
createButton(Frame,"▶ Play",180,function() playRecording(recordedFrames,false) end)
createButton(Frame,"⟳ Loop",240,function() playRecording(recordedFrames,true) end)

-- Input salvar
local SaveBox = Instance.new("TextBox")
SaveBox.Parent = Frame
SaveBox.Size = UDim2.new(0.9,0,0,40)
SaveBox.Position = UDim2.new(0.05,0,0,300)
SaveBox.PlaceholderText = "Nome da gravação..."
SaveBox.TextColor3 = Color3.fromRGB(255,255,255)
SaveBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
local cornerInput = Instance.new("UICorner")
cornerInput.CornerRadius = UDim.new(0,12)
cornerInput.Parent = SaveBox

-- Botão salvar
local SaveBtn = createButton(Frame,"💾 Salvar",360,function()
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
Scroll.Size = UDim2.new(0.9,0,0,140)
Scroll.Position = UDim2.new(0.05,0,0,420)
Scroll.BackgroundColor3 = Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness = 8
local UIList = Instance.new("UIListLayout")
UIList.Parent = Scroll
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Padding = UDim.new(0,6)

-- Atualizar lista
function refreshList()
    Scroll:ClearAllChildren()
    UIList.Parent = Scroll
    local y = 0
    for name, frames in pairs(savedRecordings) do
        local item = Instance.new("Frame")
        item.Size = UDim2.new(1,0,0,42)
        item.BackgroundTransparency = 1
        item.Parent = Scroll

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.55,0,1,0)
        label.Position = UDim2.new(0,0,0,0)
        label.Text = name
        label.TextColor3 = Color3.fromRGB(255,255,255)
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.GothamBold
        label.TextSize = 16
        label.Parent = item

        -- Play
        local playBtn = Instance.new("TextButton")
        playBtn.Size = UDim2.new(0.13,0,1,0)
        playBtn.Position = UDim2.new(0.55,0,0,0)
        playBtn.Text = "▶"
        playBtn.TextColor3 = Color3.fromRGB(255,255,255)
        playBtn.BackgroundColor3 = Color3.fromRGB(70,130,70)
        local c1 = Instance.new("UICorner"); c1.CornerRadius=UDim.new(0,8); c1.Parent=playBtn
        playBtn.Parent = item
        playBtn.MouseButton1Click:Connect(function() playRecording(frames,false) end)

        -- Loop individual
        local loopBtn = Instance.new("TextButton")
        loopBtn.Size = UDim2.new(0.13,0,1,0)
        loopBtn.Position = UDim2.new(0.68,0,0,0)
        loopBtn.Text = "⟳"
        loopBtn.TextColor3 = Color3.fromRGB(255,255,255)
        loopBtn.BackgroundColor3 = Color3.fromRGB(180,90,90)
        local c2 = Instance.new("UICorner"); c2.CornerRadius=UDim.new(0,8); c2.Parent=loopBtn
        loopBtn.Parent = item
        loopBtn.MouseButton1Click:Connect(function() playRecording(frames,true) end)

        -- Deletar
        local delBtn = Instance.new("TextButton")
        delBtn.Size = UDim2.new(0.1,0,1,0)
        delBtn.Position = UDim2.new(0.81,0,0,0)
        delBtn.Text = "❌"
        delBtn.TextColor3 = Color3.fromRGB(255,255,255)
        delBtn.BackgroundColor3 = Color3.fromRGB(150,50,50)
        local c3 = Instance.new("UICorner"); c3.CornerRadius=UDim.new(0,8); c3.Parent=delBtn
        delBtn.Parent = item
        delBtn.MouseButton1Click:Connect(function()
            savedRecordings[name] = nil
            refreshList()
        end)

        y = y + 48
    end
    Scroll.CanvasSize = UDim2.new(0,0,y)
end

-- Minimizar / flutuante
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
