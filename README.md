local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis
local isRecording = false
local recordedFrames = {}
local recordings = {} -- Armazena gravações salvas {nome = {frames}}
local currentRecording = nil
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

-- Container principal
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 300, 0, 400)
Frame.Position = UDim2.new(0.5, -150, 0.5, -200)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Visible = true
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 14)
UICorner.Parent = Frame

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "🎥 Recorder Menu"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold

-- Botão minimizar
local MinBtn = Instance.new("TextButton")
MinBtn.Parent = Frame
MinBtn.Size = UDim2.new(0, 30, 0, 30)
MinBtn.Position = UDim2.new(1, -40, 0, 5)
MinBtn.BackgroundColor3 = Color3.fromRGB(55,55,55)
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.new(1,1,1)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 20

local cornerMin = Instance.new("UICorner")
cornerMin.CornerRadius = UDim.new(0, 8)
cornerMin.Parent = MinBtn

-- Botão flutuante quando minimizado
local FloatBtn = Instance.new("TextButton")
FloatBtn.Parent = ScreenGui
FloatBtn.Size = UDim2.new(0, 55, 0, 55)
FloatBtn.Position = UDim2.new(0, 20, 1, -80)
FloatBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
FloatBtn.Text = "⚙"
FloatBtn.TextColor3 = Color3.new(1,1,1)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 26
FloatBtn.Visible = false

local cornerFloat = Instance.new("UICorner")
cornerFloat.CornerRadius = UDim.new(1, 0)
cornerFloat.Parent = FloatBtn

-- Função helper para botões
local function createButton(text, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.45, -10, 0, 40)
	btn.Position = UDim2.new(0.05, 0, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
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
local recordBtn = createButton("▶ Gravar", 50, startRecording)
recordBtn.Size = UDim2.new(0.45, -10, 0, 40)

local stopBtn = createButton("⏹ Parar", 50, stopRecording)
stopBtn.Position = UDim2.new(0.5, 5, 0, 50)

local saveBtn = createButton("💾 Salvar", 100, function()
	if #recordedFrames == 0 then return end
	-- Caixa de texto para nome
	local input = Instance.new("TextBox")
	input.Parent = Frame
	input.Size = UDim2.new(0.9, 0, 0, 35)
	input.Position = UDim2.new(0.05, 0, 0, 150)
	input.PlaceholderText = "Nome da gravação..."
	input.Text = ""
	input.TextColor3 = Color3.new(1,1,1)
	input.BackgroundColor3 = Color3.fromRGB(40,40,40)
	input.Font = Enum.Font.GothamBold
	input.TextSize = 18

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = input

	input.FocusLost:Connect(function(enter)
		if enter and input.Text ~= "" then
			recordings[input.Text] = table.clone(recordedFrames)
			input:Destroy()
		else
			input:Destroy()
		end
	end)
end)
saveBtn.Size = UDim2.new(0.9, 0, 0, 40)
saveBtn.Position = UDim2.new(0.05, 0, 0, 100)

-- Lista de gravações
local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Parent = Frame
ScrollingFrame.Size = UDim2.new(0.9, 0, 0, 120)
ScrollingFrame.Position = UDim2.new(0.05, 0, 0, 190)
ScrollingFrame.BackgroundColor3 = Color3.fromRGB(35,35,35)
ScrollingFrame.BorderSizePixel = 0
ScrollingFrame.ScrollBarThickness = 6
ScrollingFrame.CanvasSize = UDim2.new(0,0,0,0)

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = ScrollingFrame
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0,5)

-- Atualizar lista
local function updateList()
	ScrollingFrame:ClearAllChildren()
	UIListLayout.Parent = ScrollingFrame
	for name, frames in pairs(recordings) do
		local item = Instance.new("TextButton")
		item.Size = UDim2.new(1, -5, 0, 30)
		item.BackgroundColor3 = Color3.fromRGB(55,55,55)
		item.Text = name
		item.TextColor3 = Color3.new(1,1,1)
		item.Font = Enum.Font.GothamBold
		item.TextSize = 18
		item.Parent = ScrollingFrame

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 6)
		corner.Parent = item

		item.MouseButton1Click:Connect(function()
			currentRecording = name
		end)
	end
	ScrollingFrame.CanvasSize = UDim2.new(0,0,0,UIListLayout.AbsoluteContentSize.Y)
end

-- Botões de play e loop
local playBtn = createButton("▶ Play", 320, function()
	if currentRecording and recordings[currentRecording] then
		playRecording(recordings[currentRecording], false)
	end
end)
playBtn.Size = UDim2.new(0.45, -10, 0, 40)

local loopBtn = createButton("🔄 Loop", 320, function()
	if currentRecording and recordings[currentRecording] then
		playRecording(recordings[currentRecording], true)
	end
end)
loopBtn.Position = UDim2.new(0.5, 5, 0, 320)

-- Atualizar lista quando salvar
local oldSave = saveBtn.MouseButton1Click
saveBtn.MouseButton1Click:Connect(function()
	updateList()
end)

-- Alternar menu
MinBtn.MouseButton1Click:Connect(function()
	menuVisible = false
	Frame.Visible = false
	FloatBtn.Visible = true
end)

FloatBtn.MouseButton1Click:Connect(function()
	menuVisible = true
	Frame.Visible = true
	FloatBtn.Visible = false
	updateList()
end)
