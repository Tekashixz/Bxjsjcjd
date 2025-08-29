local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Vim = game:GetService("VirtualInputManager")

-- Variáveis
local isRecording = false
local recordedFrames = {}
local savedRecordings = {}
local recordStartTime = 0

-- Timer
local function formatTime(seconds)
    local mins = math.floor(seconds/60)
    local secs = math.floor(seconds%60)
    return string.format("%02d:%02d", mins, secs)
end

-- Feedback
local function showNotification(text,color)
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0,220,0,40)
    notif.Position = UDim2.new(0.5,-110,0.1,0)
    notif.BackgroundColor3 = color or Color3.fromRGB(70,70,70)
    notif.TextColor3 = Color3.new(1,1,1)
    notif.Text = text
    notif.TextSize = 18
    notif.Font = Enum.Font.GothamBold
    notif.BackgroundTransparency = 0.1
    notif.Parent = LocalPlayer:WaitForChild("PlayerGui")
    local uicorner = Instance.new("UICorner",notif)
    uicorner.CornerRadius = UDim.new(0,12)
    TweenService:Create(notif,TweenInfo.new(0.5),{Position=UDim2.new(0.5,-110,0.05,0)}):Play()
    task.delay(1.5,function()
        TweenService:Create(notif,TweenInfo.new(0.5),{Position=UDim2.new(0.5,-110,0.01,0);Transparency=1}):Play()
        task.wait(0.5)
        notif:Destroy()
    end)
end

-- Gravação
local function startRecording()
    isRecording = true
    recordedFrames = {}
    recordStartTime = tick()
    showNotification("🎬 Gravação iniciada",Color3.fromRGB(70,130,180))
end

local function stopRecording()
    isRecording = false
    showNotification("⏹ Gravação parada",Color3.fromRGB(220,50,50))
end

-- Reprodução respeitando tempo
local function playRecording(frames,loop)
    if not frames or #frames==0 then return end
    task.spawn(function()
        repeat
            local startTime = tick()
            for i,frame in ipairs(frames) do
                local waitTime = (i>1 and frame.Time - frames[i-1].Time) or 0
                local delta = waitTime - (tick()-startTime)
                if delta>0 then task.wait(delta) end
                if frame.Type=="Camera" then
                    Camera.CFrame = frame.CFrame
                elseif frame.Type=="Click" then
                    if frame.GuiObject then
                        local pos = frame.GuiObject.AbsolutePosition + frame.GuiObject.AbsoluteSize/2
                        Vim:SendMouseButtonEvent(pos.X,pos.Y,0,true,game,1)
                        task.wait(0.05)
                        Vim:SendMouseButtonEvent(pos.X,pos.Y,0,false,game,1)
                    else
                        Vim:SendMouseButtonEvent(frame.Position.X,frame.Position.Y,0,true,game,1)
                        task.wait(0.05)
                        Vim:SendMouseButtonEvent(frame.Position.X,frame.Position.Y,0,false,game,1)
                    end
                end
            end
        until not loop
    end)
end

-- Captura câmera
RunService.RenderStepped:Connect(function()
    if isRecording then
        table.insert(recordedFrames,{Type="Camera",CFrame=Camera.CFrame,Time=tick()})
    end
end)

-- Captura cliques/toques e GuiObjects
UserInputService.InputBegan:Connect(function(input,gpe)
    if isRecording and not gpe then
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            local guiObj = nil
            local mousePos = input.Position
            for _, gui in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if gui:IsA("GuiObject") and gui.AbsolutePosition and gui.AbsoluteSize then
                    local pos = gui.AbsolutePosition
                    local size = gui.AbsoluteSize
                    if mousePos.X >= pos.X and mousePos.X <= pos.X+size.X and mousePos.Y >= pos.Y and mousePos.Y <= pos.Y+size.Y then
                        guiObj = gui
                        break
                    end
                end
            end
            table.insert(recordedFrames,{Type="Click",Position=input.Position,GuiObject=guiObj,Time=tick()})
        end
    end
end)

-- GUI Panorâmica
local ScreenGui = Instance.new("ScreenGui",LocalPlayer:WaitForChild("PlayerGui"))
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0,500,0,300)
Frame.Position = UDim2.new(0.5,-250,0.5,-150)
Frame.BackgroundColor3 = Color3.fromRGB(28,28,28)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
local UICorner = Instance.new("UICorner",Frame)
UICorner.CornerRadius = UDim.new(0,18)
local UIStroke = Instance.new("UIStroke",Frame)
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(90,90,90)

-- Timer Label
local TimerLabel = Instance.new("TextLabel",Frame)
TimerLabel.Size = UDim2.new(0.2,0,0,36)
TimerLabel.Position = UDim2.new(0.05,0,0.05,0)
TimerLabel.BackgroundTransparency = 1
TimerLabel.Text = "00:00"
TimerLabel.TextColor3 = Color3.fromRGB(255,255,255)
TimerLabel.TextSize = 20
TimerLabel.Font = Enum.Font.GothamBold
TimerLabel.TextXAlignment = Enum.TextXAlignment.Left

spawn(function()
    while true do
        if isRecording then
            TimerLabel.Text = formatTime(tick()-recordStartTime)
        end
        task.wait(0.2)
    end
end)

-- Botão helper
local function createButton(parent,text,posX,posY,callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.18,0,0,36)
    btn.Position = UDim2.new(0,posX,0,posY)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.AutoButtonColor = true
    btn.Parent = parent
    local c = Instance.new("UICorner",btn)
    c.CornerRadius = UDim.new(0,12)
    btn.MouseButton1Click:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(90,90,90)
        task.wait(0.08)
        btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
        callback()
    end)
    return btn
end

-- Botões
createButton(Frame,"▶ Gravar",20,60,startRecording)
createButton(Frame,"■ Parar",120,60,stopRecording)
createButton(Frame,"▶ Reproduzir",220,60,function() playRecording(recordedFrames,false) end)
createButton(Frame,"⟳ Loop Inf.",320,60,function() playRecording(recordedFrames,true) end)

-- Input salvar
local SaveBox = Instance.new("TextBox",Frame)
SaveBox.Size = UDim2.new(0.45,0,0,36)
SaveBox.Position = UDim2.new(0.05,0,0,200)
SaveBox.PlaceholderText = "Nome da gravação..."
SaveBox.TextColor3 = Color3.fromRGB(255,255,255)
SaveBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
local cornerInput = Instance.new("UICorner",SaveBox)
cornerInput.CornerRadius = UDim.new(0,12)

-- Lista de gravações
local Scroll = Instance.new("ScrollingFrame",Frame)
Scroll.Size = UDim2.new(0.9,0,0,80)
Scroll.Position = UDim2.new(0.05,0,0,240)
Scroll.BackgroundColor3 = Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness = 8
local UIList = Instance.new("UIListLayout",Scroll)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Padding = UDim.new(0,6)

-- Refresh lista
function refreshList()
    Scroll:ClearAllChildren()
    UIList.Parent = Scroll
    for name,frames in pairs(savedRecordings) do
        local item = Instance.new("Frame",Scroll)
        item.Size = UDim2.new(1,0,0,36)
        item.BackgroundTransparency = 1
        local label = Instance.new("TextLabel",item)
        label.Size = UDim2.new(0.5,0,1,0)
        label.Text = name
        label.TextColor3 = Color3.fromRGB(255,255,255)
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.GothamBold
        label.TextSize = 16
        -- Play normal
        local playBtn = Instance.new("TextButton",item)
        playBtn.Size = UDim2.new(0.15,0,1,0)
        playBtn.Position = UDim2.new(0.5,0,0,0)
        playBtn.Text = "▶"
        playBtn.BackgroundColor3 = Color3.fromRGB(70,130,70)
        local c1 = Instance.new("UICorner",playBtn); c1.CornerRadius = UDim.new(0,8)
        playBtn.MouseButton1Click:Connect(function() playRecording(frames,false) end)
        -- Loop
        local loopBtn = Instance.new("TextButton",item)
        loopBtn.Size = UDim2.new(0.15,0,1,0)
        loopBtn.Position = UDim2.new(0.65,0,0,0)
        loopBtn.Text = "⟳"
        loopBtn.BackgroundColor3 = Color3.fromRGB(180,90,90)
        local c2 = Instance.new("UICorner",loopBtn); c2.CornerRadius = UDim.new(0,8)
        loopBtn.MouseButton1Click:Connect(function() playRecording(frames,true) end)
        -- Delete
        local delBtn = Instance.new("TextButton",item)
        delBtn.Size = UDim2.new(0.15,0,1,0)
        delBtn.Position = UDim2.new(0.8,0,0,0)
        delBtn.Text = "❌"
        delBtn.BackgroundColor3 = Color3.fromRGB(150,50,50)
        local c3 = Instance.new("UICorner",delBtn); c3.CornerRadius = UDim.new(0,8)
        delBtn.MouseButton1Click:Connect(function()
            savedRecordings[name] = nil
            refreshList()
        end)
    end
end

-- Botão salvar
createButton(Frame,"💾 Salvar",380,200,function()
    local name = SaveBox.Text
    if name~="" and #recordedFrames>0 then
        savedRecordings[name] = table.clone(recordedFrames)
        SaveBox.Text = ""
        refreshList()
        showNotification("💾 Gravação salva: "..name)
    end
end)
