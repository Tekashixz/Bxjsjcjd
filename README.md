local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis
local isRecording = false
local recordedFrames = {}
local menuVisible = true

-- Funções principais
local function startRecording()
	isRecording = true
	recordedFrames = {}
end

local function stopRecording()
	isRecording = false
end

local function playRecording(loop)
	if #recordedFrames == 0 then return end

	task.spawn(function()
		repeat
			for i = 1, #recordedFrames do
				local frame = recordedFrames[i]
				local waitTime = 0
				if i > 1 then
					waitTime = recordedFrames[i].Time - recordedFrames[i-1].Time
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

--// GUI Mobile
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Container principal
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 160, 0, 220)
Frame.Position = UDim2.new(1, -170, 1, -240)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
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
Title.BackgroundTransparency = 1
Title.Text = "📱 Recorder"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 20
Title.Font = Enum.Font.GothamBold

-- Botão minimizar (fica dentro do menu)
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

-- Função helper p/ botões grandes
local function createButton(text, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 50)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.TextSize = 20
	btn.Font = Enum.Font.GothamBold
	btn.Text = text
	btn.AutoButtonColor = true

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = btn

	btn.MouseButton1Click:Connect(callback)
	btn.Parent = Frame
end

-- Botões
createButton("▶ Gravar", 50, function()
	startRecording()
end)

createButton("■ Parar", 110, function()
	stopRecording()
end)

createButton("⟳ Loop", 170, function()
	playRecording(true)
end)

-- Botão flutuante quando minimizado
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
