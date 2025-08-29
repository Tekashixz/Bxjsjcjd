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
local menuVisible = true
local recordStartTime = 0

-- Timer
local function formatTime(seconds)
    local mins = math.floor(seconds/60)
    local secs = math.floor(seconds%60)
    return string.format("%02d:%02d", mins, secs)
end

-- Feedback visual
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
    local tween = TweenService:Create(notif,TweenInfo.new(0.5,Enum.EasingStyle.Quad),{Position=UDim2.new(0.5,-110,0.05,0)})
    tween:Play()
    task.delay(1.5,function()
        local tweenOut = TweenService:Create(notif,TweenInfo.new(0.5,Enum.EasingStyle.Quad),{Position=UDim2.new(0.5,-110,0.01,0);Transparency=1})
        tweenOut:Play()
        tweenOut.Completed:Wait()
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

-- Reprodução respeitando o tempo
local function playRecording(frames,loop)
    if not frames or #frames==0 then return end
    showNotification("▶ Reproduzindo gravação",Color3.fromRGB(50,220,50))
    task.spawn(function()
        repeat
            local startTime = tick()
            for i,frame in ipairs(frames) do
                local waitTime = 0
                if i>1 then waitTime = frame.Time - frames[i-1].Time end
                local elapsed = tick()-startTime
                local delta = waitTime - elapsed
                if delta>0 then task.wait(delta) end
                if frame.Type=="Camera" then
                    Camera.CFrame = frame.CFrame
                elseif frame.Type=="Click" then
                    -- Se a posição clicada for sobre GUI do jogo
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
        showNotification("✅ Reprodução concluída",Color3.fromRGB(50,220,50))
    end)
end

-- Captura câmera
RunService.RenderStepped:Connect(function()
    if isRecording then
        table.insert(recordedFrames,{Type="Camera",CFrame=Camera.CFrame,Time=tick()})
    end
end)

-- Captura cliques/toques e elementos de GUI
UserInputService.InputBegan:Connect(function(input,gpe)
    if isRecording and not gpe then
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            -- Detecta se clicou em algum GuiObject
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

--// GUI Premium Panorâmico
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal mais largo e menos alto
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0,480,0,360) -- mais largo e menos alto
Frame.Position = UDim2.new(0.5,-240,0.5,-180)
Frame.BackgroundColor3 = Color3.fromRGB(28,28,28)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
local UICorner = Instance.new("UICorner",Frame)
UICorner.CornerRadius = UDim.new(0,18)
local UIStroke = Instance.new("UIStroke",Frame)
UIStroke.Thickness=2
UIStroke.Color=Color3.fromRGB(90,90,90)

-- Timer Label
local TimerLabel = Instance.new("TextLabel")
TimerLabel.Size=UDim2.new(0.25,0,0,36)
TimerLabel.Position=UDim2.new(0.05,0,0,20)
TimerLabel.BackgroundTransparency=1
TimerLabel.Text="00:00"
TimerLabel.TextColor3=Color3.fromRGB(255,255,255)
TimerLabel.TextSize=20
TimerLabel.Font=Enum.Font.GothamBold
TimerLabel.TextXAlignment=Enum.TextXAlignment.Left
TimerLabel.Parent=Frame

-- Atualizar timer
spawn(function()
    while true do
        if isRecording then
            local elapsed = tick()-recordStartTime
            TimerLabel.Text=formatTime(elapsed)
        end
        task.wait(0.2)
    end
end)

-- Função criar botão premium
local function createButton(parent,text,posX,posY,callback)
    local btn=Instance.new("TextButton")
    btn.Size=UDim2.new(0.2,0,0,40)
    btn.Position=UDim2.new(0,posX,0,posY)
    btn.BackgroundColor3=Color3.fromRGB(60,60,60)
    btn.Text=text
    btn.TextColor3=Color3.fromRGB(255,255,255)
    btn.Font=Enum.Font.GothamBold
    btn.TextSize=18
    btn.AutoButtonColor=true
    btn.Parent=parent
    local corner=Instance.new("UICorner",btn)
    corner.CornerRadius=UDim.new(0,12)
    btn.MouseButton1Click:Connect(function()
        btn.BackgroundColor3=Color3.fromRGB(90,90,90)
        task.wait(0.08)
        btn.BackgroundColor3=Color3.fromRGB(60,60,60)
        callback()
    end)
    return btn
end

-- Botões principais panorâmicos
createButton(Frame,"▶ Gravar",20,70,function() startRecording() end)
createButton(Frame,"■ Parar",130,70,function() stopRecording() end)
createButton(Frame,"▶ Reproduzir",240,70,function() playRecording(recordedFrames,false) end)
createButton(Frame,"⟳ Loop Inf.",350,70,function() playRecording(recordedFrames,true) end)

-- Input salvar
local SaveBox = Instance.new("TextBox")
SaveBox.Parent = Frame
SaveBox.Size=UDim2.new(0.45,0,0,36)
SaveBox.Position=UDim2.new(0.05,0,0,280)
SaveBox.PlaceholderText="Nome da gravação..."
SaveBox.TextColor3=Color3.fromRGB(255,255,255)
SaveBox.BackgroundColor3=Color3.fromRGB(50,50,50)
local cornerInput=Instance.new("UICorner",SaveBox)
cornerInput.CornerRadius=UDim.new(0,12)

-- Lista de gravações
local Scroll = Instance.new("ScrollingFrame")
Scroll.Parent = Frame
Scroll.Size = UDim2.new(0.9,0,0,60)
Scroll.Position = UDim2.new(0.05,0,0,320)
Scroll.BackgroundColor3 = Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness = 8
local UIList = Instance.new("UIListLayout",Scroll)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Padding = UDim.new(0,6)

-- Atualizar lista
function refreshList()
    Scroll:ClearAllChildren()
    UIList.Parent=Scroll
    local y=0
    for name,frames in pairs(savedRecordings) do
        local item=Instance.new("Frame")
        item.Size=UDim2.new(1,0,0,36)
        item.BackgroundTransparency=1
        item.Parent=Scroll

        local label=Instance.new("TextLabel")
        label.Size=UDim2.new(0.5,0,1,0)
        label.Position=UDim2.new(0,0,0,0)
        label.Text=name
        label.TextColor3=Color3.fromRGB(255,255,255)
        label.BackgroundTransparency=1
        label.Font=Enum.Font.GothamBold
        label.TextSize=16
        label.Parent=item

        local playBtn=Instance.new("TextButton")
        play
        
