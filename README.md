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

-- Feedback visual
local function showNotification(text,color)
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0,200,0,40)
    notif.Position = UDim2.new(0.5,-100,0.1,0)
    notif.BackgroundColor3 = color or Color3.fromRGB(70,70,70)
    notif.TextColor3 = Color3.new(1,1,1)
    notif.Text = text
    notif.TextSize = 18
    notif.Font = Enum.Font.GothamBold
    notif.BackgroundTransparency = 0.1
    notif.Parent = LocalPlayer:WaitForChild("PlayerGui")
    local uicorner = Instance.new("UICorner",notif)
    uicorner.CornerRadius = UDim.new(0,12)
    local tween = TweenService:Create(notif,TweenInfo.new(0.5,Enum.EasingStyle.Quad),{Position=UDim2.new(0.5,-100,0.05,0)})
    tween:Play()
    task.delay(1.5,function()
        local tweenOut = TweenService:Create(notif,TweenInfo.new(0.5,Enum.EasingStyle.Quad),{Position=UDim2.new(0.5,-100,0.01,0);Transparency=1})
        tweenOut:Play()
        tweenOut.Completed:Wait()
        notif:Destroy()
    end)
end

-- Gravação
local function startRecording()
    isRecording = true
    recordedFrames = {}
    showNotification("🎬 Gravação iniciada",Color3.fromRGB(70,130,180))
end

local function stopRecording()
    isRecording = false
    showNotification("⏹ Gravação parada",Color3.fromRGB(220,50,50))
end

local function playRecording(frames,loop)
    if not frames or #frames == 0 then return end
    showNotification("▶ Reproduzindo gravação",Color3.fromRGB(50,220,50))
    task.spawn(function()
        repeat
            for i,frame in ipairs(frames) do
                local waitTime = 0
                if i>1 then waitTime=frame.Time-frames[i-1].Time end
                task.wait(waitTime)
                if frame.Type=="Camera" then
                    Camera.CFrame = frame.CFrame
                elseif frame.Type=="Click" then
                    Vim:SendMouseButtonEvent(frame.Position.X,frame.Position.Y,0,true,game,1)
                    task.wait(0.05)
                    Vim:SendMouseButtonEvent(frame.Position.X,frame.Position.Y,0,false,game,1)
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

-- Captura cliques/toques
UserInputService.InputBegan:Connect(function(input,gpe)
    if isRecording and not gpe then
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            table.insert(recordedFrames,{Type="Click",Position=input.Position,Time=tick()})
        end
    end
end)

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size=UDim2.new(0,360,0,520)
Frame.Position=UDim2.new(0.5,-180,0.5,-260)
Frame.BackgroundColor3=Color3.fromRGB(30,30,30)
Frame.BorderSizePixel=0
Frame.Parent=ScreenGui
local UICorner = Instance.new("UICorner",Frame)
UICorner.CornerRadius=UDim.new(0,16)
local UIStroke = Instance.new("UIStroke",Frame)
UIStroke.Thickness=2
UIStroke.Color=Color3.fromRGB(80,80,80)

-- Drag da GUI
local dragging=false
local dragInput,mousePos,framePos
local function update(input)
    local delta = input.Position - mousePos
    Frame.Position = UDim2.new(0,framePos.X+delta.X,0,framePos.Y+delta.Y)
end
Frame.InputBegan:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 then
        dragging=true
        mousePos=input.Position
        framePos=Frame.Position
        input.Changed:Connect(function()
            if input.UserInputState==Enum.UserInputState.End then dragging=false end
        end)
    end
end)
Frame.InputChanged:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseMovement then
        dragInput=input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input==dragInput and dragging then update(input) end
end)

-- Título
local Title=Instance.new("TextLabel")
Title.Parent=Frame
Title.Size=UDim2.new(1,0,0,50)
Title.Position=UDim2.new(0,0,0,0)
Title.BackgroundColor3=Color3.fromRGB(45,45,45)
Title.Text="🎥 Recorder Pro Ultimate"
Title.TextColor3=Color3.fromRGB(255,255,255)
Title.TextSize=24
Title.Font=Enum.Font.GothamBold
local cornerTitle=Instance.new("UICorner",Title)
cornerTitle.CornerRadius=UDim.new(0,16)

-- Botão minimizar
local MinBtn=Instance.new("TextButton")
MinBtn.Parent=Title
MinBtn.Size=UDim2.new(0,36,0,36)
MinBtn.Position=UDim2.new(1,-42,0.5,-18)
MinBtn.BackgroundColor3=Color3.fromRGB(60,60,60)
MinBtn.Text="-"
MinBtn.TextColor3=Color3.fromRGB(255,255,255)
MinBtn.Font=Enum.Font.GothamBold
MinBtn.TextSize=24
local cornerMin=Instance.new("UICorner",MinBtn)
cornerMin.CornerRadius=UDim.new(0,10)

-- Botão flutuante
local FloatBtn=Instance.new("TextButton")
FloatBtn.Parent=ScreenGui
FloatBtn.Size=UDim2.new(0,64,0,64)
FloatBtn.Position=UDim2.new(1,-80,1,-140)
FloatBtn.BackgroundColor3=Color3.fromRGB(45,45,45)
FloatBtn.Text="⚙"
FloatBtn.TextColor3=Color3.fromRGB(255,255,255)
FloatBtn.Font=Enum.Font.GothamBold
FloatBtn.TextSize=32
FloatBtn.Visible=false
local cornerFloat=Instance.new("UICorner",FloatBtn)
cornerFloat.CornerRadius=UDim.new(1,0)

-- Função criar botão premium
local function createButton(parent,text,posY,callback)
    local btn=Instance.new("TextButton")
    btn.Size=UDim2.new(0.9,0,0,44)
    btn.Position=UDim2.new(0.05,0,0,posY)
    btn.BackgroundColor3=Color3.fromRGB(60,60,60)
    btn.Text=text
    btn.TextColor3=Color3.fromRGB(255,255,255)
    btn.Font=Enum.Font.GothamBold
    btn.TextSize=20
    btn.AutoButtonColor=true
    btn.Parent=parent
    local corner=Instance.new("UICorner",btn)
    corner.CornerRadius=UDim.new(0,12)

    -- Animação clique
    btn.MouseButton1Click:Connect(function()
        btn.BackgroundColor3=Color3.fromRGB(90,90,90)
        task.wait(0.08)
        btn.BackgroundColor3=Color3.fromRGB(60,60,60)
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
local SaveBox=Instance.new("TextBox")
SaveBox.Parent=Frame
SaveBox.Size=UDim2.new(0.9,0,0,40)
SaveBox.Position=UDim2.new(0.05,0,0,300)
SaveBox.PlaceholderText="Nome da gravação..."
SaveBox.TextColor3=Color3.fromRGB(255,255,255)
SaveBox.BackgroundColor3=Color3.fromRGB(50,50,50)
local cornerInput=Instance.new("UICorner",SaveBox)
cornerInput.CornerRadius=UDim.new(0,12)

-- Botão salvar
createButton(Frame,"💾 Salvar",360,function()
    local name=SaveBox.Text
    if name~="" and #recordedFrames>0 then
        savedRecordings[name]=table.clone(recordedFrames)
        SaveBox.Text=""
        refreshList()
        showNotification("💾 Gravação salva: "..name)
    end
end)

-- Lista de gravações
local Scroll=Instance.new("ScrollingFrame")
Scroll.Parent=Frame
Scroll.Size=UDim2.new(0.9,0,0,140)
Scroll.Position=UDim2.new(0.05,0,0,420)
Scroll.BackgroundColor3=Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness=8
local UIList=Instance.new("UIListLayout",Scroll)
UIList.SortOrder=Enum.SortOrder.LayoutOrder
UIList.Padding=UDim.new(0,6)

-- Atualizar lista
function refreshList()
    Scroll:ClearAllChildren()
    UIList.Parent=Scroll
    local y=0
    for name,frames in pairs(savedRecordings) do
        local item=Instance.new("Frame")
        item.Size=UDim2.new(1,0,0,42)
        item.BackgroundTransparency=1
        item.Parent=Scroll

        local label=Instance.new("TextLabel")
        label.Size=UDim2.new(0.55,0,1,0)
        label.Position=UDim2.new(0,0,0,0)
        label.Text=name
        label.TextColor3=Color3.fromRGB(255,255,255)
        label.BackgroundTransparency=1
        label.Font=Enum.Font.GothamBold
        label.TextSize=16
        label.Parent=item

        -- Play
        local playBtn=Instance.new("TextButton")
        playBtn.Size=UDim2.new(0.13,0,1,0)
        playBtn.Position=UDim2.new(0.55,0,0,0)
        playBtn.Text="▶"
        playBtn.TextColor3=Color3.fromRGB(255,255,255)
        playBtn.BackgroundColor3=Color3.fromRGB(70,130,70)
        local c1=Instance.new("UICorner",playBtn)
        c1.CornerRadius=UDim.new(0,8)
        playBtn.Parent=item
        playBtn.MouseButton1Click:Connect(function() playRecording(frames,false) end)

        -- Loop individual
        local loopBtn=Instance.new("TextButton")
        loopBtn.Size=UDim2.new(0.13,0,1,0)
        loopBtn.Position=UDim2.new(0.68,0,0,0)
        loopBtn.Text="⟳"
        loopBtn.TextColor3=Color3.fromRGB(255,255,255)
        loopBtn.BackgroundColor3=Color3.fromRGB(180,90,90)
        local c2=Instance.new("UICorner",loopBtn)
        c2.CornerRadius=UDim.new(0,8)
        loopBtn.Parent=item
        loopBtn.MouseButton1Click:Connect(function() playRecording(frames,true) end)

        -- Deletar
        local delBtn=Instance.new("TextButton")
        delBtn.Size=UDim2.new(0.1,0,1,0)
        delBtn.Position=UDim2.new(0.81,0,0,0)
        delBtn.Text="❌"
        delBtn.TextColor3=Color3.fromRGB(255,255,255)
        delBtn.BackgroundColor3=Color3.fromRGB(150,50,50)
        local c3=Instance.new("UICorner",delBtn)
        c3.CornerRadius=UDim.new(0,8)
        delBtn.Parent=item
        delBtn.MouseButton1Click:Connect(function()
            savedRecordings[name]=nil
            refreshList()
        end)
        y=y+48
    end
    Scroll.CanvasSize=UDim2.new(0,0,y)
end

-- Minimizar / flutuante
MinBtn.MouseButton1Click:Connect(function()
    menuVisible=false
    Frame.Visible=false
    FloatBtn.Visible=true
end)
FloatBtn.MouseButton1Click:Connect(function()
    menuVisible=true
    Frame.Visible=true

   
