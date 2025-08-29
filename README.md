local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis
local isRecording = false
local recordedFrames = {}
local savedRecordings = {}
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
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BackgroundTransparency = 0.1
Frame.BorderSizePixel = 0
Frame.Visible = true
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = Frame

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Text = "🎥 Recorder Pro"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold

local cornerTitle = Instance.new("UICorner")
cornerTitle.CornerRadius = UDim.new(0, 12)
cornerTitle.Parent = Title

-- Botão minimizar
local MinBtn = Instance.new("TextButton")
MinBtn.Parent = Title
MinBtn.Size = UDim2.new(0, 30, 0, 30)
MinBtn.Position = UDim2.new(1, -35, 0.5, -15)
MinBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
MinBtn.Text = "-"
MinBtn.TextColor3 = Color3.new(1,1,1)
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 20

local cornerMin = Instance.new("UICorner")
cornerMin.CornerRadius = UDim.new(0, 8)
cornerMin.Parent = MinBtn

-- Botão flutuante
local FloatBtn = Instance.new("TextButton")
FloatBtn.Parent = ScreenGui
FloatBtn.Size = UDim2.new(0, 60, 0, 60)
FloatBtn.Position = UDim2.new(1, -70, 1, -120)
FloatBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
FloatBtn.Text = "⚙"
FloatBtn.TextColor3 = Color3.new(1,1,1)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 28
FloatBtn.Visible = false

local cornerFloat = Instance.new("UICorner")
cornerFloat.CornerRadius = UDim.new(1, 0)
cornerFloat.Parent = FloatBtn

-- Área de botões
local ButtonFrame = Instance.new("Frame")
ButtonFrame.Parent = Frame
ButtonFrame.Size = UDim2.new(1, -20, 0, 140)
ButtonFrame.Position = UDim2.new(0, 10, 0, 50)
ButtonFrame.BackgroundTransparency = 1

-- Função helper p/ botões
local function createButton(text, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, 40)
	btn.Position = UDim2.new(0, 0, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.TextSize = 20
	btn.Font = Enum.Font.GothamBold
	btn.Text = text

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = btn

	btn.MouseButton1Click:Connect(callback)
	btn.Parent = ButtonFrame
end

-- Botões principais
createButton("▶ Gravar", 0, function()
	startRecording()
end)

createButton("■ Parar", 50, function()
	stopRecording()
end)

createButton("▶ Play", 100, function()
	playRecording(recordedFrames, false)
end)

createButton("⟳ Loop", 150, function()
	playRecording(recordedFrames, true)
end)

-- Input para salvar
local SaveBox = Instance.new("TextBox")
SaveBox.Parent = Frame
SaveBox.Size = UDim2.new(1, -20, 0, 40)
SaveBox.Position = UDim2.new(0, 10, 0, 200)
SaveBox.PlaceholderText = "Nome da gravação..."
SaveBox.Text = ""
SaveBox.Font = Enum.Font.Gotham
SaveBox.TextSize = 18
SaveBox.TextColor3 = Color3.new(1,1,1)
SaveBox.BackgroundColor3 = Color3.fromRGB(40,40,40)

local cornerInput = Instance.new("UICorner")
cornerInput.CornerRadius = UDim.new(0, 8)
cornerInput.Parent = SaveBox

-- Botão salvar
local SaveBtn = Instance.new("TextButton")
SaveBtn.Parent = Frame
SaveBtn.Size = UDim2.new(1, -20, 0, 40)
SaveBtn.Position = UDim2.new(0, 10, 0, 250)
SaveBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
SaveBtn.Text = "💾 Salvar"
SaveBtn.TextColor3 = Color3.new(1,1,1)
SaveBtn.Font = Enum.Font.GothamBold
SaveBtn.TextSize = 20

local cornerSave = Instance.new("UICorner")
cornerSave.CornerRadius = UDim.new(0, 8)
cornerSave.Parent = SaveBtn

-- Lista de gravações
local Scroll = Instance.new("ScrollingFrame")
Scroll.Parent = Frame
Scroll.Size = UDim2.new(1, -20, 0, 100)
Scroll.Position = UDim2.new(0, 10, 0, 300)
Scroll.CanvasSize = UDim2.new(0,0,0,0)
Scroll.BackgroundColor3 = Color3.fromRGB(35,35,35)
Scroll.ScrollBarThickness = 6

local cornerScroll = Instance.new("UICorner")
cornerScroll.CornerRadius = UDim.new(0, 8)
cornerScroll.Parent = Scroll

-- Atualizar lista
local function refreshList()
	Scroll:ClearAllChildren()
	local y = 0
	for name, frames in pairs(savedRecordings) do
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(1, -10, 0, 35)
		btn.Position = UDim2.new(0, 5, 0, y)
		btn.BackgroundColor3 = Color3.fromRGB(55,55,55)
		btn.Text = "▶ " .. name
		btn.TextColor3 = Color3.new(1,1,1)
		btn.Font = Enum.Font.GothamBold
		btn.TextSize = 18

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 8)
		corner.Parent = btn

		btn.MouseButton1Click:Connect(function()
			playRecording(frames, false)
		end)

		btn.Parent = Scroll
		y = y + 40
	end
	Scroll.CanvasSize = UDim2.new(0,0,0,y)
end

-- Salvar gravação
SaveBtn.MouseButton1Click:Connect(function()
	if SaveBox.Text ~= "" and #recordedFrames > 0 then
		savedRecordings[SaveBox.Text] = recordedFrames
		SaveBox.Text = ""
		refreshList()
	end
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
end)
