# Silent-aim-mobile
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- ==========================================
-- CONFIGURAÇÕES E ESTADO
-- ==========================================
local SilentAimEnabled = false
local TeamCheckEnabled = true
local WallCheckEnabled = true
local AimFOV = 150 -- Valor inicial do FOV

-- ==========================================
-- SUPORTE A ARRASTAR (MOUSE & TOUCH)
-- ==========================================
local function MakeDraggable(Frame, Handle)
	local Dragging = false
	local DragInput, DragStart, StartPosition

	local function Update(Input)
		local Delta = Input.Position - DragStart
		Frame.Position = UDim2.new(
			StartPosition.X.Scale, 
			StartPosition.X.Offset + Delta.X, 
			StartPosition.Y.Scale, 
			StartPosition.Y.Offset + Delta.Y
		)
	end

	Handle.InputBegan:Connect(function(Input)
		if Input.UserInputType == Enum.UserInputType.MouseButton1 or Input.UserInputType == Enum.UserInputType.Touch then
			Dragging = true
			DragStart = Input.Position
			StartPosition = Frame.Position

			Input.Changed:Connect(function()
				if Input.UserInputState == Enum.UserInputState.End then
					Dragging = false
				end
			end)
		end
	end)

	Handle.InputChanged:Connect(function(Input)
		if Input.UserInputType == Enum.UserInputType.MouseMovement or Input.UserInputType == Enum.UserInputType.Touch then
			DragInput = Input
		end
	end)

	UserInputService.InputChanged:Connect(function(Input)
		if Input == DragInput and Dragging then
			Update(Input)
		end
	end)
end

-- ==========================================
-- CRIAÇÃO DA UI (CLEAN & MODERN)
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SilentAimSystem"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- 1. BOTÃO FLUTUANTE (ABRIR/FECHAR)
local FloatButton = Instance.new("TextButton")
FloatButton.Name = "FloatButton"
FloatButton.Size = UDim2.new(0, 50, 0, 50)
FloatButton.Position = UDim2.new(0.05, 0, 0.2, 0)
FloatButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
FloatButton.Text = "AIM"
FloatButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatButton.Font = Enum.Font.GothamBold
FloatButton.TextSize = 14
FloatButton.BorderSizePixel = 0
FloatButton.Parent = ScreenGui

local FloatCorner = Instance.new("UICorner")
FloatCorner.CornerRadius = UDim.new(0, 25) -- Círculo perfeito
FloatCorner.Parent = FloatButton

local FloatStroke = Instance.new("UIStroke")
FloatStroke.Color = Color3.fromRGB(70, 70, 70)
FloatStroke.Thickness = 1.5
FloatStroke.Parent = FloatButton

MakeDraggable(FloatButton, FloatButton)

-- 2. PAINEL PRINCIPAL (MENU)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 270)
MainFrame.Position = UDim2.new(0.5, -125, 0.4, -135)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false -- Inicia oculto
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(50, 50, 50)
MainStroke.Thickness = 1.5
MainStroke.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "SILENT SYSTEM"
Title.TextColor3 = Color3.fromRGB(220, 220, 220)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.Parent = MainFrame

MakeDraggable(MainFrame, Title) -- Arrastável pelo título

-- Função auxiliar para criar botões Toggle modernos
local function CreateToggle(name, text, defaultVal, yPos, callback)
	local Btn = Instance.new("TextButton")
	Btn.Name = name
	Btn.Size = UDim2.new(0.9, 0, 0, 35)
	Btn.Position = UDim2.new(0.05, 0, 0, yPos)
	Btn.BackgroundColor3 = defaultVal and Color3.fromRGB(40, 60, 40) or Color3.fromRGB(30, 30, 30)
	Btn.Text = text .. (defaultVal and ": ON" or ": OFF")
	Btn.TextColor3 = defaultVal and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
	Btn.Font = Enum.Font.GothamMedium
	Btn.TextSize = 12
	Btn.BorderSizePixel = 0
	Btn.Parent = MainFrame

	local Corner = Instance.new("UICorner")
	Corner.CornerRadius = UDim.new(0, 6)
	Corner.Parent = Btn

	local enabled = defaultVal
	Btn.MouseButton1Click:Connect(function()
		enabled = not enabled
		Btn.Text = text .. (enabled and ": ON" or ": OFF")
		Btn.TextColor3 = enabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
		Btn.BackgroundColor3 = enabled and Color3.fromRGB(40, 60, 40) or Color3.fromRGB(30, 30, 30)
		callback(enabled)
	end)
end

-- Toggles
CreateToggle("AimToggle", "Silent Aim", SilentAimEnabled, 45, function(val) SilentAimEnabled = val end)
CreateToggle("TeamToggle", "Team Check", TeamCheckEnabled, 85, function(val) TeamCheckEnabled = val end)
CreateToggle("WallToggle", "Wall Check", WallCheckEnabled, 125, function(val) WallCheckEnabled = val end)

-- 3. FOV BAR (SLIDER DE RAIO)
local SliderLabel = Instance.new("TextLabel")
SliderLabel.Size = UDim2.new(0.9, 0, 0, 20)
SliderLabel.Position = UDim2.new(0.05, 0, 0, 175)
SliderLabel.BackgroundTransparency = 1
SliderLabel.Text = "FOV Radius: " .. AimFOV
SliderLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
SliderLabel.Font = Enum.Font.GothamMedium
SliderLabel.TextSize = 11
SliderLabel.TextXAlignment = Enum.TextXAlignment.Left
SliderLabel.Parent = MainFrame

local SliderBar = Instance.new("Frame")
SliderBar.Size = UDim2.new(0.9, 0, 0, 8)
SliderBar.Position = UDim2.new(0.05, 0, 0, 200)
SliderBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
SliderBar.BorderSizePixel = 0
SliderBar.Parent = MainFrame

local SliderBarCorner = Instance.new("UICorner")
SliderBarCorner.CornerRadius = UDim.new(0, 4)
SliderBarCorner.Parent = SliderBar

local SliderFill = Instance.new("Frame")
SliderFill.Size = UDim2.new(AimFOV / 300, 0, 1, 0) -- Limite máx de 300
SliderFill.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
SliderFill.BorderSizePixel = 0
SliderFill.Parent = SliderBar

local SliderFillCorner = Instance.new("UICorner")
SliderFillCorner.CornerRadius = UDim.new(0, 4)
SliderFillCorner.Parent = SliderFill

-- Lógica do Slider (Suporta Clique e Arrastar com Dedo/Mouse)
local function UpdateSlider(input)
	local percentage = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
	AimFOV = math.floor(percentage * 300) -- Alcance de 0 a 300 pixels
	if AimFOV < 10 then AimFOV = 10 end -- Mínimo
	SliderFill.Size = UDim2.new(percentage, 0, 1, 0)
	SliderLabel.Text = "FOV Radius: " .. AimFOV
end

local Sliding = false
SliderBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		Sliding = true
		UpdateSlider(input)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if Sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		UpdateSlider(input)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		Sliding = false
	end
end)

-- Fechar/Abrir via Botão Flutuante
FloatButton.MouseButton1Click:Connect(function()
	MainFrame.Visible = not MainFrame.Visible
end)

-- 4. CÍRCULO VISUAL DE FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 64
FOVCircle.Filled = false

-- ==========================================
-- CORE: LÓGICA DE DETECÇÃO (TEAM + WALL CHECK)
-- ==========================================

-- Verifica se há barreira visual entre a câmera e o alvo
local function IsVisible(TargetPart, Character)
	local RaycastParams = RaycastParams.new()
	RaycastParams.FilterType = Enum.RaycastFilterType.Exclude
	RaycastParams.FilterDescendantsInstances = {LocalPlayer.Character, Character}
	RaycastParams.IgnoreWater = true

	local Origin = Camera.CFrame.Position
	local Direction = TargetPart.Position - Origin
	local RayResult = workspace:Raycast(Origin, Direction, RaycastParams)

	return RayResult == nil -- Retorna true se não colidiu com paredes
end

local function GetClosestPlayer()
	local Target = nil
	local ShortestDistance = math.huge
	local MousePos = UserInputService:GetMouseLocation() -- Melhor suporte multi-plataforma do que Mouse.X/Y

	for _, Player in ipairs(Players:GetPlayers()) do
		if Player ~= LocalPlayer and Player.Character then
			local Character = Player.Character
			local Humanoid = Character:FindFirstChildOfClass("Humanoid")
			local Head = Character:FindFirstChild("Head")

			if Humanoid and Humanoid.Health > 0 and Head then
				-- 1. Team Check
				if TeamCheckEnabled and Player.Team == LocalPlayer.Team then
					continue
				end

				-- Projeta a cabeça do alvo no espaço 2D da tela
				local ScreenPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)

				if OnScreen then
					local Distance = (Vector2.new(ScreenPos.X, ScreenPos.Y) - MousePos).Magnitude

					-- Se estiver dentro do limite do Slider (AimFOV)
					if Distance < ShortestDistance and Distance <= AimFOV then
						-- 2. Wall Check
						if WallCheckEnabled and not IsVisible(Head, Character) then
							continue
						end

						ShortestDistance = Distance
						Target = Head
					end
				end
			end
		end
	end
	return Target
end

-- Loop de Atualização do Desenho do Círculo
RunService.RenderStepped:Connect(function()
	if SilentAimEnabled then
		local MousePos = UserInputService:GetMouseLocation()
		FOVCircle.Radius = AimFOV
		FOVCircle.Position = MousePos
		FOVCircle.Visible = true
	else
		FOVCircle.Visible = false
	end
end)
