--===================================================================================--
--                SISTEMA DE ASSISTÊNCIA E INTERFACE GRÁFICA AVANÇADA               --
--===================================================================================--

--// Serviços do Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

--// Referências Globais
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Configurações Globais (Estado Inicial)
local Config = {
    AimbotEnabled = false,
    TeamCheck = true,
    WallCheck = true,
    TargetNPCs = true,
    FovRadius = 150,
    FovVisible = true,
    ChamsEnabled = false,
    Smoothness = 0.15, -- Suavização do movimento da câmera (0.1 = Rápido, 0.5 = Lento)
}

--// Instanciando Desenhos Bidimensionais (FOV)
local FovCircle = Drawing.new("Circle")
FovCircle.Thickness = 1.5
FovCircle.Color = Color3.fromRGB(0, 255, 127)
FovCircle.Filled = false
FovCircle.Transparency = 0.8
FovCircle.NumSides = 64
FovCircle.Visible = Config.FovVisible
FovCircle.Radius = Config.FovRadius

--===================================================================================--
--                             Criação da Interface Gráfica                           --
--===================================================================================--

-- Criando ScreenGui protegida (tenta colocar no CoreGui se disponível, caso contrário vai no PlayerGui)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AmbienteDeTestes_Gui"
ScreenGui.ResetOnSpawn = false
pcall(function()
    ScreenGui.Parent = CoreGui
end)
if not ScreenGui.Parent then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Botão Flutuante Arrastável para Celular / PC
local FloatingButton = Instance.new("TextButton")
FloatingButton.Name = "FloatingButton"
FloatingButton.Size = UDim2.new(0, 60, 0, 60)
FloatingButton.Position = UDim2.new(0.1, 0, 0.4, 0)
FloatingButton.BackgroundColor3 = Color3.fromRGB(31, 31, 46)
FloatingButton.Text = "MENU"
FloatingButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatingButton.Font = Enum.Font.GothamBold
FloatingButton.TextSize = 14
FloatingButton.BorderSizePixel = 0
FloatingButton.Parent = ScreenGui

local UICornerFloat = Instance.new("UICorner")
UICornerFloat.CornerRadius = UDim.new(1, 0) -- Redondo
UICornerFloat.Parent = FloatingButton

local UIStrokeFloat = Instance.new("UIStroke")
UIStrokeFloat.Color = Color3.fromRGB(0, 180, 216)
UIStrokeFloat.Width = 2
UIStrokeFloat.Parent = FloatingButton

-- Mecânica de Arrastar o Botão Flutuante (Segura o Touch ou Clique)
local dragging, dragInput, dragStart, startPos
local function updateDrag(input)
    local delta = input.Position - dragStart
    FloatingButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

FloatingButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = FloatingButton.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

FloatingButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateDrag(input)
    end
end)

-- Painel Principal do Menu
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 320, 0, 400)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 37)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

local UICornerMain = Instance.new("UICorner")
UICornerMain.CornerRadius = UDim.new(0, 10)
UICornerMain.Parent = MainFrame

local UIStrokeMain = Instance.new("UIStroke")
UIStrokeMain.Color = Color3.fromRGB(45, 45, 68)
UIStrokeMain.Width = 1
UIStrokeMain.Parent = MainFrame

-- Título do Menu
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 46)
Title.Text = "Menu de Testes Avançado"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.BorderSizePixel = 0
Title.Parent = MainFrame

local UICornerTitle = Instance.new("UICorner")
UICornerTitle.CornerRadius = UDim.new(0, 10)
UICornerTitle.Parent = Title

-- Container dos Itens do Menu (Lista)
local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -20, 1, -65)
Container.Position = UDim2.new(0, 10, 0, 55)
Container.BackgroundTransparency = 1
Container.BorderSizePixel = 0
Container.ScrollBarThickness = 4
Container.CanvasSize = UDim2.new(0, 0, 0, 420)
Container.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.Parent = Container

-- Alternar visibilidade do Menu ao clicar no Botão Flutuante
FloatingButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

--===================================================================================--
--                         Funções Utilitárias para Componentes                      --
--===================================================================================--

-- Criador de Botão Liga/Desliga (Toggle)
local function CreateToggle(name, default, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, 0, 0, 40)
    Frame.BackgroundTransparency = 1
    Frame.Parent = Container

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.7, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = "  " .. name
    Label.TextColor3 = Color3.fromRGB(200, 200, 200)
    Label.Font = Enum.Font.GothamSemibold
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0.25, 0, 0.8, 0)
    Button.Position = UDim2.new(0.75, 0, 0.1, 0)
    Button.BorderSizePixel = 0
    Button.Text = ""
    Button.Parent = Frame

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Button

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = Color3.fromRGB(60, 60, 80)
    Stroke.Width = 1
    Stroke.Parent = Button

    local state = default
    local function updateVisual()
        if state then
            Button.BackgroundColor3 = Color3.fromRGB(0, 180, 216)
            Button.Text = "ATIVO"
            Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            Button.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
            Button.Text = "OFF"
            Button.TextColor3 = Color3.fromRGB(150, 150, 150)
        end
    end
    updateVisual()

    Button.MouseButton1Click:Connect(function()
        state = not state
        updateVisual()
        callback(state)
    end)
end

-- Criador de Controles Deslizantes (Sliders)
local function CreateSlider(name, min, max, default, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, 0, 0, 50)
    Frame.BackgroundTransparency = 1
    Frame.Parent = Container

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 0.5, 0)
    Label.BackgroundTransparency = 1
    Label.Text = "  " .. name
    Label.TextColor3 = Color3.fromRGB(200, 200, 200)
    Label.Font = Enum.Font.GothamSemibold
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local ValLabel = Instance.new("TextLabel")
    ValLabel.Size = UDim2.new(0.4, 0, 0.5, 0)
    ValLabel.Position = UDim2.new(0.6, 0, 0, 0)
    ValLabel.BackgroundTransparency = 1
    ValLabel.Text = tostring(default)
    ValLabel.TextColor3 = Color3.fromRGB(0, 180, 216)
    ValLabel.Font = Enum.Font.GothamBold
    ValLabel.TextSize = 13
    ValLabel.TextXAlignment = Enum.TextXAlignment.Right
    ValLabel.Parent = Frame

    local SliderBar = Instance.new("TextButton")
    SliderBar.Size = UDim2.new(1, -10, 0, 6)
    SliderBar.Position = UDim2.new(0, 5, 0.7, 0)
    SliderBar.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
    SliderBar.BorderSizePixel = 0
    SliderBar.Text = ""
    SliderBar.Parent = Frame

    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    Fill.BackgroundColor3 = Color3.fromRGB(0, 180, 216)
    Fill.BorderSizePixel = 0
    Fill.Parent = SliderBar

    local active = false
    local function snap(input)
        local ratio = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
        local value = math.round(min + (max - min) * ratio)
        Fill.Size = UDim2.new(ratio, 0, 1, 0)
        ValLabel.Text = tostring(value)
        callback(value)
    end

    SliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            active = true
            snap(input)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if active and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            snap(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            active = false
        end
    end)
end

--===================================================================================--
--                       Injeção de Opções na UI do Usuário                           --
--===================================================================================--

CreateToggle("Ativar Aimbot", Config.AimbotEnabled, function(state) Config.AimbotEnabled = state end)
CreateToggle("Ver Círculo de FOV", Config.FovVisible, function(state) Config.FovVisible = state end)
CreateSlider("Tamanho do FOV", 50, 400, Config.FovRadius, function(value) Config.FovRadius = value end)
CreateToggle("Verificação de Times (Team)", Config.TeamCheck, function(state) Config.TeamCheck = state end)
CreateToggle("Verificação de Parede (Wall)", Config.WallCheck, function(state) Config.WallCheck = state end)
CreateToggle("Detectar NPCs (Bots)", Config.TargetNPCs, function(state) Config.TargetNPCs = state end)
CreateToggle("Ativar Chams (Inimigos)", Config.ChamsEnabled, function(state) Config.ChamsEnabled = state end)

--===================================================================================--
--                         Lógica Principal de Assistência                           --
--===================================================================================--

-- Verifica se há objetos bloqueando a visão direta para o alvo (Raycast)
local function isPointVisible(targetPosition, character)
    local origin = Camera.CFrame.Position
    local direction = targetPosition - origin
    
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LocalPlayer.Character, character}
    params.IgnoreWater = true
    
    local result = workspace:Raycast(origin, direction, params)
    return result == nil -- Retorna verdadeiro se o feixe não colidir com nenhum obstáculo
end

-- Seleciona o melhor alvo baseado no FOV estático no centro da tela
local function GetClosestTarget()
    local closestTarget = nil
    local shortestDistance = math.huge
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    -- Função interna para avaliar um modelo (seja jogador ou NPC)
    local function evaluateCandidate(model, isNpc)
        if not model then return end
        
        -- Verifica se o candidato está vivo
        local humanoid = model:FindFirstChildOfClass("Humanoid")
        local rootPart = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Head")
        
        if humanoid and humanoid.Health > 0 and rootPart then
            -- Team Check
            if not isNpc and Config.TeamCheck then
                local playerInstance = Players:GetPlayerFromCharacter(model)
                if playerInstance and playerInstance.Team == LocalPlayer.Team then
                    return
                end
            end
            
            -- Projeta a posição 3D na Tela 2D
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            if onScreen then
                -- O FOV centralizado calcula a distância a partir do centro físico da tela
                local distanceToCenter = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                
                if distanceToCenter <= Config.FovRadius and distanceToCenter < shortestDistance then
                    -- Wall Check
                    if Config.WallCheck and not isPointVisible(rootPart.Position, model) then
                        return
                    end
                    
                    shortestDistance = distanceToCenter
                    closestTarget = rootPart
                end
            end
        end
    end

    -- Escaneia Jogadores Reais
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            evaluateCandidate(player.Character, false)
        end
    end

    -- Escaneia Bots e NPCs no Workspace
    if Config.TargetNPCs then
        for _, descendant in ipairs(workspace:GetDescendants()) do
            if descendant:IsA("Model") and not Players:GetPlayerFromCharacter(descendant) and descendant ~= LocalPlayer.Character then
                evaluateCandidate(descendant, true)
            end
        end
    end

    return closestTarget
end

-- Renderização Ativa (Executada a cada Frame)
RunService.RenderStepped:Connect(function()
    -- Atualiza dinamicamente a posição e tamanho do FOV Estático
    FovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FovCircle.Radius = Config.FovRadius
    FovCircle.Visible = Config.FovVisible

    -- Processa o travamento de mira (Aimbot)
    if Config.AimbotEnabled then
        local target = GetClosestTarget()
        if target then
            -- Mover a Câmera suavemente em direção à parte alvo (geralmente Head ou RootPart)
            local targetCFrame = CFrame.new(Camera.CFrame.Position, target.Position)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, Config.Smoothness)
        end
    end
end)

--===================================================================================--
--                    Chams (Visualizador de Inimigos por Paredes)                   --
--===================================================================================--

local highlightStorage = {}

local function applyHighlight(character, isEnemy)
    if not isEnemy then
        if highlightStorage[character] then
            highlightStorage[character]:Destroy()
            highlightStorage[character] = nil
        end
        return
    end

    if not highlightStorage[character] then
        local highlight = Instance.new("Highlight")
        highlight.Name = "Test_Highlight"
        highlight.FillColor = Color3.fromRGB(255, 30, 70)
        highlight.FillTransparency = 0.5
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.OutlineTransparency = 0.1
        highlight.Adornee = character
        highlight.Parent = ScreenGui
        highlightStorage[character] = highlight
    end
end

-- Ciclo contínuo de controle de efeitos visuais (Chams)
task.spawn(function()
    while task.wait(0.5) do
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local isEnemy = true
                if Config.TeamCheck and player.Team == LocalPlayer.Team then
                    isEnemy = false
                end
                
                if Config.ChamsEnabled and isEnemy then
                    applyHighlight(player.Character, true)
                else
                    applyHighlight(player.Character, false)
                end
            end
        end
        
        -- Remoção segura para jogadores que desconectaram
        for char, highlight in pairs(highlightStorage) do
            if not char or not char.Parent then
                highlight:Destroy()
                highlightStorage[char] = nil
            end
        end
    end
end)FloatButton.TextSize = 14
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
