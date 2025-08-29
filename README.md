local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")

-- Variáveis
local isRecording = false
local recordedFrames = {}
local savedReplays = {} -- { [nome] = {frames} }
local menuVisible = true

-- Funções principais
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
			for i = 1, #frames do
				local frame = frames[i]
				local waitTime = 0
				if i > 1 then
					waitTime = frames[i].Time - frames[i-1].Time
				end
				task.wait(waitTime)
				Camera.CFrame = frame.CFrame
			end
		until not loop
	end)
end

-- Loop de gravação
RunService.RenderStepped:Connect(function()
	if isRecording then
		table.insert(recordedFrames, {
			CFrame = Camera.CFrame,
			Time = tick()
		})
	end
end)

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Painel
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 320, 0, 400)
Frame.Position = UDim2.new(1, -340, 1, -420)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = Frame

-- Sombra
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.Parent = Frame
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0, -15, 0, -15)
Shadow.Size = UDim2.new(1, 30, 1, 30)
Shadow.Image = "rbxassetid://1316045217"
Shadow.ImageTransparency = 0.4
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(10, 10, 118, 118)
Shadow.ZIndex = -1

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1, -40, 0, 40)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🎥 Recorder Avançado"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 20
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão minimizar
local MinBtn = Instance.new("TextButton")
MinBtn.Parent = Frame
MinBtn.Size = UDim2.new(0, 30, 0, 30)
MinBtn.Position = UDim2.new(1, -35, 0, 5)
MinBtn.BackgroundColor3 = Color3.fromRGB(55,55,55)
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.new(1,1,1)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 20
local cornerMin = Instance.new("UICorner")
cornerMin.CornerRadius = UDim.new(0, 8)
cornerMin.Parent = MinBtn

-- Botão flutuante (quando menu escondido)
local FloatBtn = Instance.new("TextButton")
FloatBtn.Parent = ScreenGui
FloatBtn.Size = UDim2.new(0, 50, 0, 50)
FloatBtn.Position = UDim2.new(1, -60, 1, -120)
FloatBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
FloatBtn.Text = "⚙"
FloatBtn.TextColor3 = Color3.new(1,1,1)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 24
FloatBtn.Visible = false
local cornerFloat = Instance.new("UICorner")
cornerFloat.CornerRadius = UDim.new(1, 0)
cornerFloat.Parent = FloatBtn

-- Função helper para botões
local function createButton(text, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 40)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(55, 55, 60)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.TextSize = 18
	btn.Font = Enum.Font.GothamBold
	btn.Text = text
	btn.AutoButtonColor = true
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = btn
	btn.MouseButton1Click:Connect(callback)
	btn.Parent = Frame
	return btn
end

-- Botões principais
local recordBtn = createButton("▶ Gravar", 50, function()
	startRecording()
end)

local stopBtn = createButton("■ Parar", 100, function()
	stopRecording()
end)

-- Caixa de texto para nome
local NameBox = Instance.new("TextBox")
NameBox.Size = UDim2.new(1, -20, 0, 40)
NameBox.Position = UDim2.new(0, 10, 0, 150)
NameBox.PlaceholderText = "Nome do Replay..."
NameBox.Text = ""
NameBox.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
NameBox.TextColor3 = Color3.new(1,1,1)
NameBox.Font = Enum.Font.Gotham
NameBox.TextSize = 18
local cornerBox = Instance.new("UICorner")
cornerBox.CornerRadius = UDim.new(0, 8)
cornerBox.Parent = NameBox
NameBox.Parent = Frame

-- Botão salvar replay
local saveBtn = createButton("💾 Salvar Replay", 200, function()
	local name = NameBox.Text
	if name ~= "" and #recordedFrames > 0 then
		savedReplays[name] = table.clone(recordedFrames)
		NameBox.Text = ""
	end
end)

-- Lista de replays
local ReplayList = Instance.new("ScrollingFrame")
ReplayList.Size = UDim2.new(1, -20, 0, 150)
ReplayList.Position = UDim2.new(0, 10, 0, 250)
ReplayList.BackgroundColor3 = Color3.fromRGB(45,45,50)
ReplayList.ScrollBarThickness = 6
ReplayList.CanvasSize = UDim2.new(0,0,0,0)
ReplayList.Parent = Frame
local cornerList = Instance.new("UICorner")
cornerList.CornerRadius = UDim.new(0, 8)
cornerList.Parent = ReplayList

-- Atualizar lista de replays
local function refreshReplayList()
	ReplayList:ClearAllChildren()
	local y = 0
	for name, frames in pairs(savedReplays) do
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(1, -10, 0, 40)
		btn.Position = UDim2.new(0, 5, 0, y)
		btn.BackgroundColor3 = Color3.fromRGB(65, 65, 70)
		btn.TextColor3 = Color3.new(1,1,1)
		btn.TextSize = 16
		btn.Font = Enum.Font.GothamBold
		btn.Text = "▶ "..name
		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 6)
		corner.Parent = btn
		btn.MouseButton1Click:Connect(function()
			playRecording(frames, false)
		end)
		btn.Parent = ReplayList
		y = y + 45
	end
	ReplayList.CanvasSize = UDim2.new(0,0,0,y)
end

-- Atualiza lista sempre que salvar
saveBtn.MouseButton1Click:Connect(refreshReplayList)

-- Minimizar / Restaurar
MinBtn.MouseButton1Click:Connect(function()
	menuVisible = false
	Frame.Visible = false
	FloatBtn.Visible = true
end)

FloatBtn.MouseButton1Click:Connect(function()
	menuVisible = true
	Frame.Visible = true
	FloatBtn.Visible = false
end)
