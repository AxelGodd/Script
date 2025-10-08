-- PlayerFinderClient.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local REQUEST_TOP = ReplicatedStorage:WaitForChild("RequestTopPlayers")
local REQUEST_TELE = ReplicatedStorage:WaitForChild("RequestTeleportTo")

local localPlayer = Players.LocalPlayer

-- Crear GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlayerFinderGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Botón para mostrar/ocultar
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 140, 0, 40)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
toggleButton.TextColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18
toggleButton.Text = "Mostrar FFA"
toggleButton.Name = "ToggleFinder"
toggleButton.Parent = screenGui

-- Contenedor principal del panel
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 360, 0, 450)
frame.Position = UDim2.new(0, 20, 0, 70)
frame.BackgroundColor3 = Color3.fromRGB(255, 215, 0) -- Amarillo
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 36)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "FInd For Axxelll"
title.TextColor3 = Color3.fromRGB(0, 0, 0)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.Parent = frame

-- Imagen de noob (arriba a la derecha)
local noobImage = Instance.new("ImageLabel")
noobImage.Size = UDim2.new(0, 36, 0, 36)
noobImage.Position = UDim2.new(1, -40, 0, 2)
noobImage.BackgroundTransparency = 1
noobImage.Image = "rbxassetid://11177224439"
noobImage.ScaleType = Enum.ScaleType.Fit
noobImage.Parent = frame

-- Botón refrescar
local refreshBtn = Instance.new("TextButton")
refreshBtn.Size = UDim2.new(0, 100, 0, 28)
refreshBtn.Position = UDim2.new(1, -110, 0, 4)
refreshBtn.BackgroundColor3 = Color3.fromRGB(240, 200, 0)
refreshBtn.Text = "Refrescar"
refreshBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
refreshBtn.Font = Enum.Font.SourceSans
refreshBtn.TextSize = 16
refreshBtn.Parent = frame

-- Scrolling lista
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -10, 1, -46)
scroll.Position = UDim2.new(0, 5, 0, 40)
scroll.CanvasSize = UDim2.new(0, 0, 1, 0)
scroll.BackgroundTransparency = 1
scroll.ScrollBarThickness = 6
scroll.Parent = frame

local uiList = Instance.new("UIListLayout")
uiList.Padding = UDim.new(0, 6)
uiList.Parent = scroll

-- Crear entrada individual
local function makeEntry(data)
	local entry = Instance.new("Frame")
	entry.Size = UDim2.new(1, -10, 0, 50)
	entry.BackgroundColor3 = Color3.fromRGB(255, 239, 100)
	entry.BorderSizePixel = 0
	entry.Parent = scroll

	local nameLbl = Instance.new("TextLabel")
	nameLbl.Size = UDim2.new(0.6, 0, 1, 0)
	nameLbl.Position = UDim2.new(0, 8, 0, 0)
	nameLbl.BackgroundTransparency = 1
	nameLbl.TextXAlignment = Enum.TextXAlignment.Left
	nameLbl.Text = (data.DisplayName ~= "" and data.DisplayName or data.Name) .. " (" .. data.Name .. ")"
	nameLbl.TextColor3 = Color3.fromRGB(0, 0, 0)
	nameLbl.Font = Enum.Font.SourceSans
	nameLbl.TextSize = 16
	nameLbl.Parent = entry

	local brLbl = Instance.new("TextLabel")
	brLbl.Size = UDim2.new(0.2, 0, 1, 0)
	brLbl.Position = UDim2.new(0.6, 0, 0, 0)
	brLbl.BackgroundTransparency = 1
	brLbl.Text = tostring(data.Brainrots)
	brLbl.TextColor3 = Color3.fromRGB(50, 50, 50)
	brLbl.Font = Enum.Font.SourceSans
	brLbl.TextSize = 16
	brLbl.Parent = entry

	local teleBtn = Instance.new("TextButton")
	teleBtn.Size = UDim2.new(0.2, -10, 0.7, 0)
	teleBtn.Position = UDim2.new(0.8, -10, 0.15, 0)
	teleBtn.Text = "Ir"
	teleBtn.BackgroundColor3 = Color3.fromRGB(255, 230, 0)
	teleBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
	teleBtn.Font = Enum.Font.SourceSansBold
	teleBtn.TextSize = 14
	teleBtn.Parent = entry

	local debounce = false
	teleBtn.MouseButton1Click:Connect(function()
		if debounce then return end
		debounce = true
		REQUEST_TELE:FireServer(data.UserId)
		teleBtn.Text = "..."
		task.wait(0.5)
		teleBtn.Text = "Ir"
		debounce = false
	end)
end

-- Llenar lista
local function populateList(list)
	for _, child in pairs(scroll:GetChildren()) do
		if child:IsA("Frame") then child:Destroy() end
	end
	for _, data in ipairs(list) do
		makeEntry(data)
	end
	scroll.CanvasSize = UDim2.new(0, 0, 0, uiList.AbsoluteContentSize.Y + 10)
end

-- Recibir datos del servidor
REQUEST_TOP.OnClientEvent:Connect(function(topList)
	populateList(topList)
end)

-- Refrescar lista
refreshBtn.MouseButton1Click:Connect(function()
	REQUEST_TOP:FireServer(20)
end)

-- Mostrar/Ocultar panel
toggleButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
	toggleButton.Text = frame.Visible and "Ocultar FFA" or "Mostrar FFA"
	if frame.Visible then
		REQUEST_TOP:FireServer(20)
	end
end)
