#-- MURDER MYSTERY DEBUG - ALL IN ONE
-- Painel: Beiçola do Luiz e Guilherme
-- Server + Client (auto-injetado)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- =======================
-- REMOTE EVENT
-- =======================
local RolesEvent = ReplicatedStorage:FindFirstChild("MM_RolesEvent")
if not RolesEvent then
	RolesEvent = Instance.new("RemoteEvent")
	RolesEvent.Name = "MM_RolesEvent"
	RolesEvent.Parent = ReplicatedStorage
end

-- =======================
-- SISTEMA DE ROLES
-- =======================
local Roles = {}

local function assignRoles()
	local plrs = Players:GetPlayers()
	if #plrs < 3 then return end

	Roles = {}
	for _,p in ipairs(plrs) do
		Roles[p.UserId] = "Innocent"
	end

	local murderer = plrs[math.random(#plrs)]
	Roles[murderer.UserId] = "Murderer"

	local sheriff
	repeat sheriff = plrs[math.random(#plrs)] until sheriff ~= murderer
	Roles[sheriff.UserId] = "Sheriff"

	RolesEvent:FireAllClients(Roles)
end

-- =======================
-- CLIENT (PAINEL MOBILE)
-- =======================
local CLIENT_SOURCE = [[
local Players = game:GetService("Players")
local RS = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local RolesEvent = RS:WaitForChild("MM_RolesEvent")

local rolesCache = {}

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "MM_DebugGUI"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- BOLINHA
local ball = Instance.new("TextButton", gui)
ball.Size = UDim2.new(0,50,0,50)
ball.Position = UDim2.new(1,-65,0.5,-25)
ball.Text = "☰"
ball.Font = Enum.Font.GothamBold
ball.TextSize = 22
ball.BackgroundColor3 = Color3.fromRGB(200,0,0)
ball.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", ball).CornerRadius = UDim.new(1,0)

-- PAINEL
local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0,320,0,360)
panel.Position = UDim2.new(0,15,1,-375)
panel.BackgroundColor3 = Color3.fromRGB(18,18,18)
panel.Visible = true
Instance.new("UICorner", panel).CornerRadius = UDim.new(0,14)

local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1,0,0,45)
title.BackgroundTransparency = 1
title.Text = "Beiçola do Luiz e Guilherme"
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextColor3 = Color3.new(1,1,1)

-- ABRIR / FECHAR
ball.MouseButton1Click:Connect(function()
	panel.Visible = not panel.Visible
end)

-- TAGS
local function clearTags()
	for _,plr in pairs(Players:GetPlayers()) do
		if plr.Character then
			local t = plr.Character:FindFirstChild("RoleTag")
			if t then t:Destroy() end
		end
	end
end

local function applyTags()
	clearTags()
	for _,plr in pairs(Players:GetPlayers()) do
		if plr.Character and plr.Character:FindFirstChild("Head") then
			local role = rolesCache[plr.UserId]
			if role then
				local bb = Instance.new("BillboardGui")
				bb.Name = "RoleTag"
				bb.Size = UDim2.new(0,140,0,30)
				bb.StudsOffset = Vector3.new(0,2.5,0)
				bb.AlwaysOnTop = true
				bb.Parent = plr.Character

				local tx = Instance.new("TextLabel", bb)
				tx.Size = UDim2.new(1,0,1,0)
				tx.BackgroundTransparency = 1
				tx.Font = Enum.Font.GothamBold
				tx.TextScaled = true
				tx.Text = plr.Name

				if role == "Murderer" then
					tx.TextColor3 = Color3.fromRGB(255,0,0)
				elseif role == "Sheriff" then
					tx.TextColor3 = Color3.fromRGB(0,140,255)
				else
					tx.TextColor3 = Color3.fromRGB(0,255,0)
				end
			end
		end
	end
end

RolesEvent.OnClientEvent:Connect(function(r)
	rolesCache = r
	applyTags()
end)
]]

-- =======================
-- INJETAR CLIENT
-- =======================
local function injectClient(player)
	local pg = player:WaitForChild("PlayerGui")

	if pg:FindFirstChild("MM_Client") then return end

	local ls = Instance.new("LocalScript")
	ls.Name = "MM_Client"
	ls.Source = CLIENT_SOURCE
	ls.Parent = pg
end

Players.PlayerAdded:Connect(function(player)
	injectClient(player)
	task.wait(1)
	assignRoles()
end)

Players.PlayerRemoving:Connect(assignRoles)

-- primeira rodada
task.delay(2, assignRoles)
