--==================================================
-- KS PING LAB
-- MENU INDEPENDENTE
-- CLIQUE = ABRIR/FECHAR
-- ARRASTAR = MOVER KS
--==================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local selectedPlayer = nil
local enabled = false
local delayTime = 0.125

--==================================================
-- GUI
--==================================================

local gui = Instance.new("ScreenGui")
gui.Name = "KSPingLab"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = player:WaitForChild("PlayerGui")

--==================================================
-- BOTÃO KS
--==================================================

local ks = Instance.new("TextButton")
ks.Name = "KS_Ping_Button"
ks.Size = UDim2.new(0, 60, 0, 60)
ks.Position = UDim2.new(0, 20, 0.5, -30)

ks.BackgroundColor3 = Color3.fromRGB(105, 45, 180)
ks.Text = "KS"
ks.TextColor3 = Color3.fromRGB(255, 255, 255)
ks.TextSize = 21
ks.Font = Enum.Font.GothamBold
ks.AutoButtonColor = false
ks.Parent = gui

local ksCorner = Instance.new("UICorner")
ksCorner.CornerRadius = UDim.new(1, 0)
ksCorner.Parent = ks

local ksStroke = Instance.new("UIStroke")
ksStroke.Color = Color3.fromRGB(195, 130, 255)
ksStroke.Thickness = 2
ksStroke.Parent = ks

--==================================================
-- MENU
--==================================================

local menu = Instance.new("Frame")
menu.Name = "PingMenu"
menu.Size = UDim2.new(0, 330, 0, 380)
menu.BackgroundColor3 = Color3.fromRGB(22, 12, 35)
menu.Visible = false
menu.Parent = gui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 16)
menuCorner.Parent = menu

local menuStroke = Instance.new("UIStroke")
menuStroke.Color = Color3.fromRGB(150, 75, 230)
menuStroke.Thickness = 2
menuStroke.Parent = menu

--==================================================
-- TÍTULO
--==================================================

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -30, 0, 32)
title.Position = UDim2.new(0, 15, 0, 8)
title.BackgroundTransparency = 1
title.Text = "KS PING LAB"
title.TextColor3 = Color3.fromRGB(220, 175, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = menu

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -30, 0, 20)
subtitle.Position = UDim2.new(0, 15, 0, 40)
subtitle.BackgroundTransparency = 1
subtitle.Text = "Kalebzin duels"
subtitle.TextColor3 = Color3.fromRGB(165, 125, 190)
subtitle.TextSize = 11
subtitle.Font = Enum.Font.Gotham
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = menu

local status = Instance.new("TextLabel")
status.Size = UDim2.new(0, 100, 0, 25)
status.Position = UDim2.new(1, -115, 0, 12)
status.BackgroundTransparency = 1
status.Text = "OFF"
status.TextColor3 = Color3.fromRGB(175, 150, 190)
status.TextSize = 11
status.Font = Enum.Font.GothamBold
status.TextXAlignment = Enum.TextXAlignment.Right
status.Parent = menu

--==================================================
-- TARGET
--==================================================

local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(1, -30, 0, 32)
targetLabel.Position = UDim2.new(0, 15, 0, 72)
targetLabel.BackgroundColor3 = Color3.fromRGB(37, 20, 49)
targetLabel.Text = "Target: nenhum"
targetLabel.TextColor3 = Color3.fromRGB(235, 225, 240)
targetLabel.TextSize = 12
targetLabel.Font = Enum.Font.GothamBold
targetLabel.Parent = menu

local targetCorner = Instance.new("UICorner")
targetCorner.CornerRadius = UDim.new(0, 9)
targetCorner.Parent = targetLabel

--==================================================
-- LISTA DE JOGADORES
--==================================================

local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -30, 0, 90)
list.Position = UDim2.new(0, 15, 0, 110)
list.BackgroundColor3 = Color3.fromRGB(27, 15, 37)
list.BorderSizePixel = 0
list.ScrollBarThickness = 4
list.Parent = menu

local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 8)
listCorner.Parent = list

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 4)
layout.Parent = list

local function refreshPlayers()

	for _, object in ipairs(list:GetChildren()) do
		if object:IsA("TextButton") then
			object:Destroy()
		end
	end

	local count = 0

	for _, target in ipairs(Players:GetPlayers()) do

		if target ~= player then

			local button = Instance.new("TextButton")
			button.Size = UDim2.new(1, -8, 0, 28)
			button.BackgroundColor3 = Color3.fromRGB(55, 28, 75)

			button.Text =
				target.DisplayName ..
				"  @" ..
				target.Name

			button.TextColor3 = Color3.fromRGB(240, 225, 248)
			button.TextSize = 10
			button.Font = Enum.Font.GothamBold
			button.AutoButtonColor = false
			button.Parent = list

			local c = Instance.new("UICorner")
			c.CornerRadius = UDim.new(0, 7)
			c.Parent = button

			button.MouseButton1Click:Connect(function()

				selectedPlayer = target

				targetLabel.Text =
					"Target: " ..
					target.DisplayName

			end)

			count += 1
		end
	end

	list.CanvasSize =
		UDim2.new(
			0,
			0,
			0,
			count * 32
		)
end

Players.PlayerAdded:Connect(refreshPlayers)

Players.PlayerRemoving:Connect(function(target)

	if selectedPlayer == target then

		selectedPlayer = nil
		targetLabel.Text = "Target: nenhum"

		if enabled then
			enabled = false
			status.Text = "OFF"
			enableButton.Text = "ENABLE"
			enableButton.BackgroundColor3 =
				Color3.fromRGB(95, 42, 145)
		end
	end

	refreshPlayers()
end)

refreshPlayers()

--==================================================
-- MODOS
--==================================================

local function createMode(text, x, value)

	local button = Instance.new("TextButton")

	button.Size =
		UDim2.new(0, 72, 0, 34)

	button.Position =
		UDim2.new(0, x, 0, 210)

	button.BackgroundColor3 =
		Color3.fromRGB(55, 27, 72)

	button.Text = text
	button.TextColor3 =
		Color3.fromRGB(240, 225, 250)

	button.TextSize = 11
	button.Font = Enum.Font.GothamBold
	button.AutoButtonColor = false
	button.Parent = menu

	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 8)
	c.Parent = button

	button.MouseButton1Click:Connect(function()

		delayTime = value
		updateDelay()

	end)

end

createMode("LOW", 15, 0.125)
createMode("MID", 92, 0.250)
createMode("HIGH", 169, 0.500)
createMode("ULTRA", 246, 1.000)

--==================================================
-- DELAY
--==================================================

local delayLabel = Instance.new("TextLabel")
delayLabel.Size = UDim2.new(1, -30, 0, 32)
delayLabel.Position = UDim2.new(0, 15, 0, 252)
delayLabel.BackgroundColor3 = Color3.fromRGB(37, 20, 49)
delayLabel.Text = "Delay: 125 ms"
delayLabel.TextColor3 = Color3.fromRGB(235, 225, 240)
delayLabel.TextSize = 12
delayLabel.Font = Enum.Font.GothamBold
delayLabel.Parent = menu

local delayCorner = Instance.new("UICorner")
delayCorner.CornerRadius = UDim.new(0, 9)
delayCorner.Parent = delayLabel

function updateDelay()

	delayLabel.Text =
		"Delay: " ..
		math.floor(delayTime * 1000) ..
		" ms"

end

--==================================================
-- MENOS
--==================================================

local minus = Instance.new("TextButton")
minus.Size = UDim2.new(0, 60, 0, 34)
minus.Position = UDim2.new(0, 15, 0, 295)
minus.BackgroundColor3 = Color3.fromRGB(55, 27, 72)
minus.Text = "−"
minus.TextColor3 = Color3.new(1, 1, 1)
minus.TextSize = 18
minus.Font = Enum.Font.GothamBold
minus.Parent = menu

local minusCorner = Instance.new("UICorner")
minusCorner.CornerRadius = UDim.new(0, 8)
minusCorner.Parent = minus

minus.MouseButton1Click:Connect(function()

	delayTime =
		math.max(
			delayTime - 0.025,
			0.05
		)

	updateDelay()

end)

--==================================================
-- MAIS
--==================================================

local plus = Instance.new("TextButton")
plus.Size = UDim2.new(0, 60, 0, 34)
plus.Position = UDim2.new(0, 80, 0, 295)
plus.BackgroundColor3 = Color3.fromRGB(55, 27, 72)
plus.Text = "+"
plus.TextColor3 = Color3.new(1, 1, 1)
plus.TextSize = 18
plus.Font = Enum.Font.GothamBold
plus.Parent = menu

local plusCorner = Instance.new("UICorner")
plusCorner.CornerRadius = UDim.new(0, 8)
plusCorner.Parent = plus

plus.MouseButton1Click:Connect(function()

	delayTime =
		math.min(
			delayTime + 0.025,
			2
		)

	updateDelay()

end)

--==================================================
-- ENABLE
--==================================================

enableButton = Instance.new("TextButton")
enableButton.Size = UDim2.new(0, 190, 0, 34)
enableButton.Position = UDim2.new(0, 145, 0, 295)

enableButton.BackgroundColor3 =
	Color3.fromRGB(95, 42, 145)

enableButton.Text = "ENABLE"
enableButton.TextColor3 =
	Color3.fromRGB(255, 255, 255)

enableButton.TextSize = 12
enableButton.Font = Enum.Font.GothamBold
enableButton.AutoButtonColor = false
enableButton.Parent = menu

local enableCorner = Instance.new("UICorner")
enableCorner.CornerRadius = UDim.new(0, 8)
enableCorner.Parent = enableButton

enableButton.MouseButton1Click:Connect(function()

	if not selectedPlayer then

		status.Text = "ESCOLHA ALVO"
		status.TextColor3 =
			Color3.fromRGB(255, 170, 170)

		return
	end

	enabled = not enabled

	if enabled then

		status.Text = "ON"
		status.TextColor3 =
			Color3.fromRGB(210, 155, 255)

		enableButton.Text = "DISABLE"

		enableButton.BackgroundColor3 =
			Color3.fromRGB(125, 55, 190)

	else

		status.Text = "OFF"
		status.TextColor3 =
			Color3.fromRGB(175, 150, 190)

		enableButton.Text = "ENABLE"

		enableButton.BackgroundColor3 =
			Color3.fromRGB(95, 42, 145)

	end

end)

--==================================================
-- POSIÇÃO DO MENU
--==================================================

local function updateMenuPosition()

	local screen = camera.ViewportSize

	local x = ks.Position.X.Offset
	local y = ks.Position.Y.Offset + 70

	if y + menu.AbsoluteSize.Y > screen.Y then
		y =
			ks.Position.Y.Offset -
			menu.AbsoluteSize.Y -
			10
	end

	x = math.clamp(
		x,
		0,
		math.max(
			0,
			screen.X - menu.AbsoluteSize.X
		)
	)

	y = math.clamp(
		y,
		0,
		math.max(
			0,
			screen.Y - menu.AbsoluteSize.Y
		)
	)

	menu.Position =
		UDim2.new(
			0,
			x,
			0,
			y
		)
end

--==================================================
-- CLIQUE / ARRASTO
--==================================================

local dragging = false
local moved = false

local dragStart
local startPosition

local function beginDrag(input)

	dragging = true
	moved = false

	dragStart = input.Position
	startPosition = ks.Position

end

local function updateDrag(input)

	if not dragging then
		return
	end

	local delta =
		input.Position - dragStart

	if delta.Magnitude > 6 then
		moved = true
	end

	local screen =
		camera.ViewportSize

	local newX =
		startPosition.X.Offset +
		delta.X

	local newY =
		startPosition.Y.Offset +
		delta.Y

	newX =
		math.clamp(
			newX,
			0,
			math.max(
				0,
				screen.X - ks.AbsoluteSize.X
			)
		)

	newY =
		math.clamp(
			newY,
			0,
			math.max(
				0,
				screen.Y - ks.AbsoluteSize.Y
			)
		)

	ks.Position =
		UDim2.new(
			0,
			newX,
			0,
			newY
		)

	if menu.Visible then
		updateMenuPosition()
	end

end

ks.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		beginDrag(input)

	end
end)

UserInputService.InputChanged:Connect(function(input)

	if not dragging then
		return
	end

	if input.UserInputType ==
		Enum.UserInputType.MouseMovement
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		updateDrag(input)

	end
end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		-- Só abre/fecha se NÃO tiver arrastado
		if not moved then

			menu.Visible =
				not menu.Visible

			if menu.Visible then
				updateMenuPosition()
			end

		end

		dragging = false

	end
end)

--==================================================
-- START
--==================================================

updateDelay()

print("[KS Ping Lab] carregado")
