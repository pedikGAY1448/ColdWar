-- ==========================================
-- NAIM V3 - UNIFIED SCRIPT WITH RAPID FIRE, NO RECOIL & INF AMMO
-- + ANTI-AIM INTEGRATION + SHOT SYNC + BULLET TRACERS
-- ==========================================
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local GuiService = game:GetService("GuiService")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer
local ParentGui = (gethui and gethui()) or LocalPlayer:WaitForChild("PlayerGui")

local GUI_NAME = "Xv7mKq9L2p_v3"
local CROSSHAIR_NAME = "Rt4fG8bN3q_v3"
local GLOW_NAME = "Z9wP2xK5mC_v3"
local FRIEND_HAT_NAME = "FriendHatAdornment"
local FRIEND_ESP_NAME = "FriendESPBillboard"

if ParentGui:FindFirstChild(GUI_NAME) then
	ParentGui[GUI_NAME]:Destroy()
end

for _, obj in ipairs(Workspace:GetChildren()) do
	if obj.Name == CROSSHAIR_NAME then
		obj:Destroy()
	end
end

-- ==========================================
-- СИСТЕМА ЗВУКОВ
-- ==========================================
local SOUNDS = {
	OpenClose = "rbxassetid://6895079853",
	Tab = "rbxassetid://6895490212",
	Button = "rbxassetid://6895058622",
	Toggle = "rbxassetid://9114221580",
	Dropdown = "rbxassetid://6895058622"
}

local function PlaySound(soundId, baseVol)
	task.spawn(function()
		local sound = Instance.new("Sound")
		sound.SoundId = soundId
		sound.Volume = (baseVol or 1) * 3.0
		sound.PlayOnRemove = true
		sound.Parent = SoundService
		sound:Destroy()
	end)
end

-- ==========================================
-- ШРИФТЫ И ЦВЕТОВАЯ ПАЛИТРА THEMES
-- ==========================================
local FONTS = {
	Title = Enum.Font.GothamBold,
	Body = Enum.Font.GothamBold
}

local THEMES = {
	Red = { Accent = Color3.fromRGB(245, 35, 75), Border = Color3.fromRGB(220, 35, 70), DarkAccent = Color3.fromRGB(55, 14, 24) },
	Cyan = { Accent = Color3.fromRGB(0, 210, 255), Border = Color3.fromRGB(0, 180, 230), DarkAccent = Color3.fromRGB(10, 45, 60) },
	Birch = { Accent = Color3.fromRGB(150, 220, 170), Border = Color3.fromRGB(110, 190, 130), DarkAccent = Color3.fromRGB(25, 50, 32) },
	Blue = { Accent = Color3.fromRGB(40, 120, 255), Border = Color3.fromRGB(30, 95, 220), DarkAccent = Color3.fromRGB(12, 28, 60) },
	Yellow = { Accent = Color3.fromRGB(255, 195, 0), Border = Color3.fromRGB(225, 170, 0), DarkAccent = Color3.fromRGB(55, 42, 10) },
	Purple = { Accent = Color3.fromRGB(170, 60, 240), Border = Color3.fromRGB(140, 40, 210), DarkAccent = Color3.fromRGB(42, 15, 60) },
	Lime = { Accent = Color3.fromRGB(60, 220, 80), Border = Color3.fromRGB(45, 190, 65), DarkAccent = Color3.fromRGB(15, 50, 22) },
	Black = { Accent = Color3.fromRGB(85, 85, 95), Border = Color3.fromRGB(115, 115, 125), DarkAccent = Color3.fromRGB(28, 28, 34) },
	White = { Accent = Color3.fromRGB(240, 240, 245), Border = Color3.fromRGB(200, 200, 210), DarkAccent = Color3.fromRGB(55, 55, 60) }
}

local CurrentThemeName = "Birch"
local CurrentTheme = THEMES.Birch

local COLORS = {
	Background = Color3.fromRGB(11, 10, 14),
	CardBg = Color3.fromRGB(16, 15, 20),
	SidebarBg = Color3.fromRGB(14, 13, 17),
	AccentRed = CurrentTheme.Accent,
	AccentDarkRed = CurrentTheme.DarkAccent,
	BorderRed = CurrentTheme.Border,
	BorderDim = Color3.fromRGB(32, 30, 40),
	ToggleOff = Color3.fromRGB(24, 25, 32),
	TextWhite = Color3.fromRGB(245, 245, 250),
	TextDim = Color3.fromRGB(160, 160, 175),
	TextRed = CurrentTheme.Accent,
	VisibleGreen = Color3.fromRGB(0, 255, 0),
	BehindWallYellow = Color3.fromRGB(255, 255, 0),
	FriendGreen = Color3.fromRGB(0, 255, 120),
	FriendESPColor = Color3.fromRGB(50, 255, 50),
	FriendESPGlowColor = Color3.fromRGB(0, 255, 0)
}

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = GUI_NAME
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

local DropdownOverlay = Instance.new("Frame")
DropdownOverlay.Name = "DropdownOverlay"
DropdownOverlay.Size = UDim2.new(1, 0, 1, 0)
DropdownOverlay.BackgroundTransparency = 1
DropdownOverlay.ZIndex = 100
DropdownOverlay.Parent = ScreenGui

local ActiveDropdown = nil
local function CloseActiveDropdown()
	if ActiveDropdown then
		ActiveDropdown.Close()
		ActiveDropdown = nil
	end
end

local Connections = {}
local Unloaded = false

local DynamicThemeElements = {
	Borders = {},
	Accents = {},
	DarkAccents = {},
	Toggles = {},
	ScrollBars = {}
}

local CameraCache = Workspace.CurrentCamera
Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
	CameraCache = Workspace.CurrentCamera
end)

-- ==========================================
-- ИНИЦИАЛИЗАЦИЯ ЭФФЕКТОВ ГРАФИКИ (LIGHTING)
-- ==========================================
local GraphicsEffects = {
	ColorCorrection = Lighting:FindFirstChild("NAIM_ColorCorrection") or Instance.new("ColorCorrectionEffect"),
	Bloom = Lighting:FindFirstChild("NAIM_Bloom") or Instance.new("BloomEffect"),
	SunRays = Lighting:FindFirstChild("NAIM_SunRays") or Instance.new("SunRaysEffect"),
	Blur = Lighting:FindFirstChild("NAIM_Blur") or Instance.new("BlurEffect")
}

GraphicsEffects.ColorCorrection.Name = "NAIM_ColorCorrection"
GraphicsEffects.ColorCorrection.Parent = Lighting

GraphicsEffects.Bloom.Name = "NAIM_Bloom"
GraphicsEffects.Bloom.Enabled = false
GraphicsEffects.Bloom.Parent = Lighting

GraphicsEffects.SunRays.Name = "NAIM_SunRays"
GraphicsEffects.SunRays.Enabled = false
GraphicsEffects.SunRays.Parent = Lighting

GraphicsEffects.Blur.Name = "NAIM_Blur"
GraphicsEffects.Blur.Size = 0
GraphicsEffects.Blur.Enabled = false
GraphicsEffects.Blur.Parent = Lighting

-- ==========================================
-- DRAWING FOV CIRCLE SETUP
-- ==========================================
local FOVCircle = nil
local DrawFOVState = false

if pcall(function() return Drawing.new end) then
	FOVCircle = Drawing.new("Circle")
	FOVCircle.Thickness = 1
	FOVCircle.NumSides = 60
	FOVCircle.Filled = false
	FOVCircle.Transparency = 1
	FOVCircle.Visible = false
end

-- ==========================================
-- ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
-- ==========================================
local globalRaycastParams = RaycastParams.new()
globalRaycastParams.FilterType = Enum.RaycastFilterType.Exclude

local function IsAlly(player)
	if not player or player == LocalPlayer then return true end
	if LocalPlayer.Team and player.Team then return LocalPlayer.Team == player.Team end
	if LocalPlayer.TeamColor and player.TeamColor then return LocalPlayer.TeamColor == player.TeamColor end
	return false
end

local function IsPlayerAlive(player)
	if not player then return false end
	local character = player.Character
	if not character then return false end
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not humanoid then return false end
	return humanoid.Health > 0
end

local VisibilityCache = {}

local function CheckPointVisible(origin, targetPos, ignoreList)
	globalRaycastParams.FilterDescendantsInstances = ignoreList
	local direction = targetPos - origin
	local result = Workspace:Raycast(origin, direction, globalRaycastParams)
	return result == nil
end

local function IsVisible(targetChar)
	local now = os.clock()
	if VisibilityCache[targetChar] and (now - VisibilityCache[targetChar].Time < 0.05) then
		return VisibilityCache[targetChar].Result
	end

	if not targetChar or not CameraCache then return false end
	local origin = CameraCache.CFrame.Position

	local ignoreList = {}
	if LocalPlayer.Character then table.insert(ignoreList, LocalPlayer.Character) end
	table.insert(ignoreList, targetChar)

	local partsToCheck = {
		targetChar:FindFirstChild("Head"),
		targetChar:FindFirstChild("UpperTorso") or targetChar:FindFirstChild("Torso"),
		targetChar:FindFirstChild("HumanoidRootPart")
	}

	local isAnyVisible = false
	for _, part in ipairs(partsToCheck) do
		if part and part:IsA("BasePart") then
			if CheckPointVisible(origin, part.Position, ignoreList) then
				isAnyVisible = true
				break
			end
		end
	end

	VisibilityCache[targetChar] = { Result = isAnyVisible, Time = now }
	return isAnyVisible
end

local function GetTargetPartInstance(character, targetName)
	if not character then return nil end
	if targetName == "Head" then
		return character:FindFirstChild("Head")
	elseif targetName == "Torso" then
		return character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso") or character:FindFirstChild("Torso")
	end
	return character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
end

-- ==========================================
-- КАСТОМНЫЙ ПРИЦЕЛ И DISTANCE DISPLAY
-- ==========================================
local CrosshairState = false
local CrosshairOffsetX = 0
local CrosshairOffsetY = 0
local CrosshairR = 255
local CrosshairG = 255
local CrosshairB = 255
local CrosshairThickness = 1
local CrosshairLength = 10
local CrosshairGap = 2
local CrosshairOutline = false
local ShowDistanceState = false

local CrosshairContainer = Instance.new("Frame")
CrosshairContainer.Name = CROSSHAIR_NAME
CrosshairContainer.Size = UDim2.new(0, 0, 0, 0)
CrosshairContainer.Position = UDim2.new(0.5, 0, 0.5, 0)
CrosshairContainer.AnchorPoint = Vector2.new(0.5, 0.5)
CrosshairContainer.BackgroundTransparency = 1
CrosshairContainer.Visible = false
CrosshairContainer.Parent = ScreenGui

local DistanceLabel = Instance.new("TextLabel")
DistanceLabel.Name = "DistanceLabel"
DistanceLabel.Size = UDim2.new(0, 80, 0, 14)
DistanceLabel.AnchorPoint = Vector2.new(0.5, 0)
DistanceLabel.BackgroundTransparency = 1
DistanceLabel.Text = ""
DistanceLabel.TextColor3 = Color3.fromRGB(245, 245, 250)
DistanceLabel.TextSize = 10
DistanceLabel.Font = FONTS.Body
DistanceLabel.Visible = false
DistanceLabel.Parent = CrosshairContainer

local DistanceStroke = Instance.new("UIStroke")
DistanceStroke.Color = Color3.new(0, 0, 0)
DistanceStroke.Thickness = 1
DistanceStroke.Parent = DistanceLabel

local function CreateCrosshairLine()
	local line = Instance.new("Frame")
	line.BackgroundColor3 = Color3.fromRGB(CrosshairR, CrosshairG, CrosshairB)
	line.BorderSizePixel = 0
	line.Parent = CrosshairContainer
	
	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.new(0, 0, 0)
	stroke.Thickness = 1
	stroke.Transparency = 1
	stroke.Parent = line
	
	return line
end

local TopLine = CreateCrosshairLine()
local BottomLine = CreateCrosshairLine()
local LeftLine = CreateCrosshairLine()
local RightLine = CreateCrosshairLine()

local function UpdateCrosshair()
	CrosshairContainer.Position = UDim2.new(0.5, CrosshairOffsetX, 0.5, CrosshairOffsetY)
	CrosshairContainer.Visible = CrosshairState or ShowDistanceState
	
	local col = Color3.fromRGB(CrosshairR, CrosshairG, CrosshairB)
	local outlineTrans = CrosshairOutline and 0 or 1
	
	TopLine.Size = UDim2.new(0, CrosshairThickness, 0, CrosshairLength)
	TopLine.Position = UDim2.new(0.5, -CrosshairThickness/2, 0, -CrosshairGap - CrosshairLength)
	TopLine.BackgroundColor3 = col
	TopLine.Visible = CrosshairState and CrosshairLength > 0
	TopLine.UIStroke.Transparency = outlineTrans
	
	BottomLine.Size = UDim2.new(0, CrosshairThickness, 0, CrosshairLength)
	BottomLine.Position = UDim2.new(0.5, -CrosshairThickness/2, 0, CrosshairGap)
	BottomLine.BackgroundColor3 = col
	BottomLine.Visible = CrosshairState and CrosshairLength > 0
	BottomLine.UIStroke.Transparency = outlineTrans
	
	LeftLine.Size = UDim2.new(0, CrosshairLength, 0, CrosshairThickness)
	LeftLine.Position = UDim2.new(0, -CrosshairGap - CrosshairLength, 0.5, -CrosshairThickness/2)
	LeftLine.BackgroundColor3 = col
	LeftLine.Visible = CrosshairState and CrosshairLength > 0
	LeftLine.UIStroke.Transparency = outlineTrans
	
	RightLine.Size = UDim2.new(0, CrosshairLength, 0, CrosshairThickness)
	RightLine.Position = UDim2.new(0, CrosshairGap, 0.5, -CrosshairThickness/2)
	RightLine.BackgroundColor3 = col
	RightLine.Visible = CrosshairState and CrosshairLength > 0
	RightLine.UIStroke.Transparency = outlineTrans

	DistanceLabel.Position = UDim2.new(0.5, 0, 0, CrosshairGap + CrosshairLength + 4)
	DistanceLabel.Visible = ShowDistanceState
end

-- ==========================================
-- ЛОГИКА FULLBRIGHT
-- ==========================================
local FullBrightState = false
local isUpdatingLighting = false

local OldLighting = {
	Brightness = Lighting.Brightness,
	ClockTime = Lighting.ClockTime,
	FogEnd = Lighting.FogEnd,
	GlobalShadows = Lighting.GlobalShadows,
	OutdoorAmbient = Lighting.OutdoorAmbient,
	Ambient = Lighting.Ambient
}

local function ApplyFullBright()
	if not FullBrightState or isUpdatingLighting then return end
	isUpdatingLighting = true
	Lighting.Brightness = 2
	Lighting.ClockTime = 14
	Lighting.FogEnd = 100000
	Lighting.GlobalShadows = false
	Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
	Lighting.Ambient = Color3.fromRGB(128, 128, 128)
	isUpdatingLighting = false
end

local function RestoreLighting()
	isUpdatingLighting = true
	Lighting.Brightness = OldLighting.Brightness
	Lighting.ClockTime = OldLighting.ClockTime
	Lighting.FogEnd = OldLighting.FogEnd
	Lighting.GlobalShadows = OldLighting.GlobalShadows
	Lighting.OutdoorAmbient = OldLighting.OutdoorAmbient
	Lighting.Ambient = OldLighting.Ambient
	isUpdatingLighting = false
end

-- ==========================================
-- ЛОГИКА ОПТИМИЗАЦИИ
-- ==========================================
local PotatoState = false
local OriginalMaterials = {}

local function SetPotatoGraphics(state)
	PotatoState = state
	if state then
		for _, obj in ipairs(Workspace:GetDescendants()) do
			if obj:IsA("BasePart") and not obj:IsA("Terrain") then
				if not OriginalMaterials[obj] then
					OriginalMaterials[obj] = {Material = obj.Material, MaterialVariant = obj.MaterialVariant}
				end
				obj.Material = Enum.Material.SmoothPlastic
			end
		end
	else
		for obj, props in pairs(OriginalMaterials) do
			if obj and obj.Parent and obj:IsA("BasePart") then
				obj.Material = props.Material
				pcall(function() obj.MaterialVariant = props.MaterialVariant end)
			end
		end
		table.clear(OriginalMaterials)
	end
end

local NoFogState = false
local OldFogEnd = Lighting.FogEnd
local OldAtmospheres = {}

local function ToggleNoFog(state)
	NoFogState = state
	if state then
		Lighting.FogEnd = 100000000
		for _, child in ipairs(Lighting:GetChildren()) do
			if child:IsA("Atmosphere") then
				if OldAtmospheres[child] == nil then
					OldAtmospheres[child] = child.Density
				end
				child.Density = 0
			end
		end
	else
		Lighting.FogEnd = OldFogEnd or 10000
		for atmosphere, density in pairs(OldAtmospheres) do
			if atmosphere and atmosphere.Parent then
				atmosphere.Density = density
			end
		end
		table.clear(OldAtmospheres)
	end
end

local RemoveMuzzleState = false
local DisabledMuzzles = {}

local function ProcessMuzzleObject(obj)
	if not RemoveMuzzleState then return end
	if obj:IsA("ParticleEmitter") or obj:IsA("Beam") or obj:IsA("PointLight") or obj:IsA("SpotLight") then
		local lowerName = obj.Name:lower()
		if lowerName:find("muzzle") or lowerName:find("flash") or lowerName:find("fire") or lowerName:find("smoke") or lowerName:find("spark") then
			if DisabledMuzzles[obj] == nil then
				DisabledMuzzles[obj] = obj.Enabled
			end
			obj.Enabled = false
		end
	end
end

local function ToggleRemoveMuzzle(state)
	RemoveMuzzleState = state
	if state then
		for _, obj in ipairs(Workspace:GetDescendants()) do
			ProcessMuzzleObject(obj)
		end
	else
		for obj, origState in pairs(DisabledMuzzles) do
			if obj and obj.Parent then
				obj.Enabled = origState
			end
		end
		table.clear(DisabledMuzzles)
	end
end

-- ==========================================
-- FRIEND HAT И FRIEND ESP
-- ==========================================
local FriendHatState = false
local FriendsCache = {}
local FriendAdornments = {}

local function IsFriendAsync(player)
	if player == LocalPlayer then return false end
	if FriendsCache[player.UserId] ~= nil then
		return FriendsCache[player.UserId]
	end
	
	local isFriend = false
	pcall(function()
		isFriend = LocalPlayer:IsFriendsWith(player.UserId)
	end)
	FriendsCache[player.UserId] = isFriend
	return isFriend
end

local function RemoveFriendHat(player)
	if FriendAdornments[player] then
		FriendAdornments[player]:Destroy()
		FriendAdornments[player] = nil
	end
end

local function ApplyFriendHat(player)
	if not FriendHatState then
		RemoveFriendHat(player)
		return
	end

	task.spawn(function()
		if not IsFriendAsync(player) then return end

		local char = player.Character
		if not char then RemoveFriendHat(player) return end

		local head = char:FindFirstChild("Head")
		if not head then RemoveFriendHat(player) return end

		local adornment = FriendAdornments[player]
		if not adornment or adornment.Adornee ~= head then
			RemoveFriendHat(player)

			adornment = Instance.new("CylinderHandleAdornment")
			adornment.Name = FRIEND_HAT_NAME
			adornment.Height = 0.15
			adornment.Radius = 1.1
			adornment.InnerRadius = 0.8
			adornment.Color3 = COLORS.FriendGreen
			adornment.AlwaysOnTop = true
			adornment.Transparency = 0.1
			adornment.CFrame = CFrame.new(0, 1.3, 0) * CFrame.Angles(math.rad(90), 0, 0)
			adornment.Adornee = head
			adornment.Parent = head
			FriendAdornments[player] = adornment
		end
	end)
end

local function ToggleFriendHat(state)
	FriendHatState = state
	if state then
		for _, player in ipairs(Players:GetPlayers()) do
			ApplyFriendHat(player)
		end
	else
		for player, _ in pairs(FriendAdornments) do
			RemoveFriendHat(player)
		end
	end
end

local FriendESPState = false
local FriendESPStorage = {}

local function RemoveFriendESP(player)
	if FriendESPStorage[player] then
		if FriendESPStorage[player].Billboard then FriendESPStorage[player].Billboard:Destroy() end
		if FriendESPStorage[player].Highlight then FriendESPStorage[player].Highlight:Destroy() end
		FriendESPStorage[player] = nil
	end
end

local function ApplyFriendESP(player)
	if not FriendESPState then
		RemoveFriendESP(player)
		return
	end

	task.spawn(function()
		if not IsFriendAsync(player) then RemoveFriendESP(player) return end

		local char = player.Character
		if not char or not IsPlayerAlive(player) then RemoveFriendESP(player) return end

		local head = char:FindFirstChild("Head")
		if not head then RemoveFriendESP(player) return end

		local data = FriendESPStorage[player]
		if not data or data.Character ~= char then
			RemoveFriendESP(player)
			
			data = { Character = char }
			
			local billboard = Instance.new("BillboardGui")
			billboard.Name = FRIEND_ESP_NAME
			billboard.Size = UDim2.new(0, 200, 0, 30)
			billboard.StudsOffset = Vector3.new(0, 2.5, 0)
			billboard.AlwaysOnTop = true
			billboard.MaxDistance = 500
			billboard.Adornee = head
			billboard.Parent = head
			
			local nameLabel = Instance.new("TextLabel")
			nameLabel.Size = UDim2.new(1, 0, 1, 0)
			nameLabel.BackgroundTransparency = 1
			nameLabel.Text = player.DisplayName or player.Name
			nameLabel.TextColor3 = COLORS.FriendESPColor
			nameLabel.TextSize = 14
			nameLabel.Font = Enum.Font.GothamBold
			nameLabel.TextStrokeTransparency = 0
			nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
			nameLabel.Parent = billboard
			
			data.Billboard = billboard
			
			local highlight = Instance.new("Highlight")
			highlight.Name = "FriendESPGlow"
			highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
			highlight.FillTransparency = 1
			highlight.OutlineTransparency = 0
			highlight.OutlineColor = COLORS.FriendESPGlowColor
			highlight.Parent = char
			
			data.Highlight = highlight
			FriendESPStorage[player] = data
		end
	end)
end

local function ToggleFriendESP(state)
	FriendESPState = state
	if state then
		for _, player in ipairs(Players:GetPlayers()) do ApplyFriendESP(player) end
	else
		for player, _ in pairs(FriendESPStorage) do RemoveFriendESP(player) end
	end
end

-- ==========================================
-- СИСТЕМА ТРАССЕРОВ ПУЛЬ
-- ==========================================
local BulletTracersState = true
local BulletTracersLifeTime = 5        -- секунд жизни
local BulletTracersThickness = 0.08    -- толщина цилиндра
local ActiveTracers = {}

local function GetTracerColor()
	return COLORS.AccentRed
end

local function CreateBulletTracer(startPos, endPos)
	if not BulletTracersState then return end
	if not startPos or not endPos then return end
	
	local distance = (endPos - startPos).Magnitude
	if distance < 0.1 then return end
	
	local tracerColor = GetTracerColor()
	
	local part = Instance.new("Part")
	part.Name = "NAIM_BulletTracer"
	part.Anchored = true
	part.CanCollide = false
	part.CanQuery = false
	part.CanTouch = false
	part.CastShadow = false
	part.Material = Enum.Material.Neon
	part.Color = tracerColor
	part.Transparency = 0.15
	part.Size = Vector3.new(BulletTracersThickness, BulletTracersThickness, distance)
	part.CFrame = CFrame.lookAt(startPos, endPos) * CFrame.new(0, 0, -distance / 2)
	part.Parent = Workspace
	
	local tracerData = {
		Part = part,
		StartTime = os.clock(),
		LifeTime = BulletTracersLifeTime,
		BaseTransparency = 0.15
	}
	table.insert(ActiveTracers, tracerData)
end

-- Цикл обновления прозрачности трассеров
table.insert(Connections, RunService.RenderStepped:Connect(function()
	local now = os.clock()
	for i = #ActiveTracers, 1, -1 do
		local data = ActiveTracers[i]
		if not data.Part or not data.Part.Parent then
			table.remove(ActiveTracers, i)
		else
			local elapsed = now - data.StartTime
			local ratio = elapsed / data.LifeTime
			if ratio >= 1 then
				data.Part:Destroy()
				table.remove(ActiveTracers, i)
			else
				-- Плавное затухание: от BaseTransparency до 1
				local transparency = data.BaseTransparency + (1 - data.BaseTransparency) * ratio
				data.Part.Transparency = math.clamp(transparency, 0, 1)
			end
		end
	end
end))

-- ==========================================
-- ЛОГИКА SPEED+ (ФИЗИЧЕСКИЙ ТОЛЧОК VELOCITY)
-- ==========================================
local SpeedPlusState = false
local SpeedPlusForce = 0.5

table.insert(Connections, RunService.RenderStepped:Connect(function()
	if SpeedPlusState then
		local character = LocalPlayer.Character
		if character and IsPlayerAlive(LocalPlayer) then
			local hrp = character:FindFirstChild("HumanoidRootPart")
			if hrp then
				local lookVector = hrp.CFrame.LookVector
				local currentVel = hrp.AssemblyLinearVelocity
				
				if UserInputService:IsKeyDown(Enum.KeyCode.W) then
					hrp.AssemblyLinearVelocity = Vector3.new(
						currentVel.X + (lookVector.X * SpeedPlusForce),
						currentVel.Y,
						currentVel.Z + (lookVector.Z * SpeedPlusForce)
					)
				elseif UserInputService:IsKeyDown(Enum.KeyCode.S) then
					hrp.AssemblyLinearVelocity = Vector3.new(
						currentVel.X - (lookVector.X * SpeedPlusForce),
						currentVel.Y,
						currentVel.Z - (lookVector.Z * SpeedPlusForce)
					)
				end
			end
		end
	end
end))

-- ==========================================
-- ЛОГИКА ANTI-AIM (ИНТЕГРИРОВАНО ИЗ SKEETGUI)
-- + SHOT SYNC (поворот на цель при выстреле)
-- ==========================================
local AA_Enabled = false
local AA_Pitch = 89
local AA_YawOffset = 180
local AA_Spin = false
local AA_SpinSpeed = 15
local AA_Jitter = false
local AA_JitterAngle = 45
local AA_JitterDelay = 0.1
local AA_Snapback = true
local AA_ShotSync = true
local AA_ShotSyncDuration = 0.1

local originalC0s = {}
local currentSpin = 0
local jitterState = false
local lastJitterTime = os.clock()
local snapbackFrames = 0
local aaLastShotTime = 0

local aaShotSyncTarget = nil
local aaShotSyncEndTime = 0

local function CleanupAA(hrp, humanoid)
	if humanoid then humanoid.AutoRotate = true end
	if hrp and hrp:FindFirstChild("AA_Attachment") then hrp.AA_Attachment:Destroy() end
	if hrp and hrp:FindFirstChild("AA_Align") then hrp.AA_Align:Destroy() end
end

local function ActivateAAShotSync(targetPart)
	if not AA_Enabled or not AA_ShotSync then return end
	if not targetPart then return end
	aaShotSyncTarget = targetPart
	aaShotSyncEndTime = os.clock() + AA_ShotSyncDuration
end

-- Snapback-отслеживание ЛКМ
table.insert(Connections, UserInputService.InputBegan:Connect(function(input, gpe)
	if not gpe and input.UserInputType == Enum.UserInputType.MouseButton1 then
		aaLastShotTime = os.clock()
		if AA_Enabled and AA_Snapback and not AA_ShotSync then
			snapbackFrames = 5
		end
	end
end))

-- Основной цикл AA
table.insert(Connections, RunService.RenderStepped:Connect(function(deltaTime)
	local character = LocalPlayer.Character
	if not character then return end
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not humanoid or not hrp then return end

	local shotSyncActive = AA_ShotSync 
		and aaShotSyncTarget 
		and os.clock() < aaShotSyncEndTime
		and aaShotSyncTarget.Parent 
		and aaShotSyncTarget:IsDescendantOf(Workspace)

	if not shotSyncActive then
		aaShotSyncTarget = nil
	end

	-- Snapback после выстрела (только если ShotSync выключен)
	if snapbackFrames > 0 and not shotSyncActive then
		local backYaw = math.rad(AA_YawOffset)
		hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, backYaw, 0)
		snapbackFrames = snapbackFrames - 1
	end

	local waist = character:FindFirstChild("UpperTorso") and character.UpperTorso:FindFirstChild("Waist")
	local neck = character:FindFirstChild("Head") and character.Head:FindFirstChild("Neck")

	if waist and not originalC0s[waist] then originalC0s[waist] = waist.C0 end
	if neck and not originalC0s[neck] then originalC0s[neck] = neck.C0 end

	if not AA_Enabled then
		CleanupAA(hrp, humanoid)
		if waist and originalC0s[waist] then waist.C0 = originalC0s[waist] end
		if neck and originalC0s[neck] then neck.C0 = originalC0s[neck] end
		return
	end

	humanoid.AutoRotate = false

	-- SHOT SYNC: поворот строго на цель
	if shotSyncActive then
		local targetPos = aaShotSyncTarget.Position
		local lookDir = Vector3.new(
			targetPos.X - hrp.Position.X,
			0,
			targetPos.Z - hrp.Position.Z
		)

		if lookDir.Magnitude > 0.001 then
			lookDir = lookDir.Unit
			local attachment = hrp:FindFirstChild("AA_Attachment") or Instance.new("Attachment", hrp)
			attachment.Name = "AA_Attachment"

			local align = hrp:FindFirstChild("AA_Align") or Instance.new("AlignOrientation", hrp)
			align.Name = "AA_Align"
			align.Mode = Enum.OrientationAlignmentMode.OneAttachment
			align.Attachment0 = attachment
			align.MaxTorque = 100000000
			align.MaxAngularVelocity = 1000000
			align.Responsiveness = 1000

			local baseLookCFrame = CFrame.lookAt(Vector3.zero, lookDir)
			align.CFrame = baseLookCFrame
		end

		if waist and originalC0s[waist] then waist.C0 = originalC0s[waist] end
		if neck and originalC0s[neck] then neck.C0 = originalC0s[neck] end
		return
	end

	-- ОБЫЧНЫЙ AA (spin/jitter/offset)
	local finalYaw = AA_YawOffset
	if AA_Spin then
		currentSpin = (currentSpin + AA_SpinSpeed) % 360
		finalYaw = finalYaw + currentSpin
	end
	if AA_Jitter then
		if os.clock() - lastJitterTime > AA_JitterDelay then
			jitterState = not jitterState
			lastJitterTime = os.clock()
		end
		if jitterState then
			finalYaw = finalYaw + AA_JitterAngle
		else
			finalYaw = finalYaw - AA_JitterAngle
		end
	end

	local lookPos = hrp.Position + CameraCache.CFrame.LookVector
	local lookDir = (lookPos - hrp.Position)
	lookDir = Vector3.new(lookDir.X, 0, lookDir.Z)

	if lookDir.Magnitude > 0.001 then
		lookDir = lookDir.Unit
		local attachment = hrp:FindFirstChild("AA_Attachment") or Instance.new("Attachment", hrp)
		attachment.Name = "AA_Attachment"

		local align = hrp:FindFirstChild("AA_Align") or Instance.new("AlignOrientation", hrp)
		align.Name = "AA_Align"
		align.Mode = Enum.OrientationAlignmentMode.OneAttachment
		align.Attachment0 = attachment
		align.MaxTorque = 100000000
		align.MaxAngularVelocity = 1000000
		align.Responsiveness = 200

		local baseLookCFrame = CFrame.lookAt(Vector3.zero, lookDir)
		align.CFrame = baseLookCFrame * CFrame.Angles(0, math.rad(finalYaw), 0)
	end

	local pitchRad = math.rad(AA_Pitch)
	if waist and neck then
		waist.C0 = originalC0s[waist] * CFrame.Angles(pitchRad * 0.5, 0, 0)
		neck.C0 = originalC0s[neck] * CFrame.Angles(pitchRad * 0.5, 0, 0)
	elseif neck then
		neck.C0 = originalC0s[neck] * CFrame.Angles(-pitchRad, 0, 0)
	end
end))

-- ==========================================
-- ЛОГИКА AIMBOT И SILENT AIM
-- ==========================================
local AimbotState = false
local AimbotTargetPart = "Head"
local AimbotSpeed = 30
local AimbotVisibleCheck = true
local AimbotKeyName = "Mouse 2 (RMB)"
local AimbotFOV = 65

local AimToCrosshair = false
local AimbotPrediction = true
local AimbotPredictionAmount = 0.007

local DistanceCompensateState = false
local RandomHitpartState = false
local currentRandomPart = nil
local lastTargetPlayer = nil

local SilentAimState = false
local SilentAimFOV = 50
local SilentAimTargetPart = "Head"
local SilentAimTeamCheck = true
local SilentAimWallCheck = true
local SilentAimMaxDistance = 1000

local function GetRandomWeightedPartName()
	local rng = math.random(1, 100)
	if rng <= 65 then return "Head"
	elseif rng <= 95 then return "Torso"
	else return "Head" end
end

local function GetActiveTargetPart()
	if RandomHitpartState then
		if not currentRandomPart then currentRandomPart = GetRandomWeightedPartName() end
		return currentRandomPart
	end
	return AimbotTargetPart
end

local function CalculateDistanceOffset(targetPos, camPos)
	if not DistanceCompensateState then return Vector3.zero end
	local distance = (targetPos - camPos).Magnitude
	if distance <= 20 then return Vector3.zero end
	local deltaDist = distance - 20
	local dropOffset = (deltaDist ^ 1.45) * 0.000175
	return Vector3.new(0, dropOffset, 0)
end

local function IsAimKeyPressed()
	local keyEnum = (AimbotKeyName == "Mouse 1 (LMB)") and Enum.UserInputType.MouseButton1 or Enum.UserInputType.MouseButton2
	return UserInputService:IsMouseButtonPressed(keyEnum)
end

local function GetAimScreenCenter()
	if not CameraCache then return UserInputService:GetMouseLocation() end
	if AimToCrosshair then
		local viewport = CameraCache.ViewportSize
		local inset = GuiService:GetGuiInset()
		return Vector2.new(viewport.X / 2 + CrosshairOffsetX, viewport.Y / 2 + CrosshairOffsetY + inset.Y)
	else
		return UserInputService:GetMouseLocation()
	end
end

local function GetPredictedPosition(part)
	if not part then return nil end
	local pos = part.Position
	if AimbotPrediction then
		local char = part.Parent
		local root = char and (char:FindFirstChild("HumanoidRootPart") or part)
		if root and root:IsA("BasePart") then
			local velocity = root.AssemblyLinearVelocity or root.Velocity
			pos = pos + (velocity * AimbotPredictionAmount)
		end
	end
	return pos
end

local function GetTargetCFrame(camPos, targetPos)
	local baseCFrame = CFrame.lookAt(camPos, targetPos)
	if not CameraCache or not AimToCrosshair or (CrosshairOffsetX == 0 and CrosshairOffsetY == 0) then
		return baseCFrame
	end
	
	local viewport = CameraCache.ViewportSize
	local center = Vector2.new(viewport.X / 2, viewport.Y / 2)
	local crosshairPos = Vector2.new(center.X + CrosshairOffsetX, center.Y + CrosshairOffsetY)
	
	local centerRay = CameraCache:ViewportPointToRay(center.X, center.Y)
	local crosshairRay = CameraCache:ViewportPointToRay(crosshairPos.X, crosshairPos.Y)
	
	local offsetRotation = CFrame.lookAt(Vector3.zero, centerRay.Direction):ToObjectSpace(CFrame.lookAt(Vector3.zero, crosshairRay.Direction))
	return baseCFrame * offsetRotation:Inverse()
end

local function GetClosestPlayer(targetPartName)
	local closestPlayer = nil
	local shortestDistance = AimbotFOV
	if not CameraCache then return nil end

	local mousePos = GetAimScreenCenter()

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and not IsAlly(player) and IsPlayerAlive(player) then
			local char = player.Character
			if char then
				local humanoid = char:FindFirstChildOfClass("Humanoid")
				local part = GetTargetPartInstance(char, targetPartName or GetActiveTargetPart())
				
				if humanoid and humanoid.Health > 0 and part then
					local predPos = GetPredictedPosition(part)
					local screenPos, onScreen = CameraCache:WorldToScreenPoint(predPos)
					if onScreen then
						local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
						if dist < shortestDistance then
							if not AimbotVisibleCheck or IsVisible(char) then
								shortestDistance = dist
								closestPlayer = player
							end
						end
					end
				end
			end
		end
	end
	return closestPlayer
end

table.insert(Connections, RunService.RenderStepped:Connect(function(deltaTime)
	if FOVCircle then
		if DrawFOVState and not Unloaded then
			local center = GetAimScreenCenter()
			FOVCircle.Position = center
			FOVCircle.Radius = AimbotFOV
			FOVCircle.Color = COLORS.AccentRed
			FOVCircle.Visible = true
		else
			FOVCircle.Visible = false
		end
	end

	if AimbotState and IsAimKeyPressed() then
		local activePart = GetActiveTargetPart()
		local target = GetClosestPlayer(activePart)
		
		if target then
			if target ~= lastTargetPlayer then
				lastTargetPlayer = target
				if RandomHitpartState then
					currentRandomPart = GetRandomWeightedPartName()
					activePart = currentRandomPart
				end
			end

			if target.Character and IsPlayerAlive(target) then
				local part = GetTargetPartInstance(target.Character, activePart)
				if part and CameraCache then
					local targetPos = GetPredictedPosition(part)
					targetPos = targetPos + CalculateDistanceOffset(targetPos, CameraCache.CFrame.Position)
					local targetCFrame = GetTargetCFrame(CameraCache.CFrame.Position, targetPos)
					
					if AimbotSpeed >= 100 then
						CameraCache.CFrame = targetCFrame
					else
						local alpha = math.clamp((AimbotSpeed / 100) * deltaTime * 60, 0.01, 1)
						CameraCache.CFrame = CameraCache.CFrame:Lerp(targetCFrame, alpha)
					end
				end
			end
		else
			lastTargetPlayer = nil
			currentRandomPart = nil
		end
	else
		lastTargetPlayer = nil
		currentRandomPart = nil
	end
end))

-- ==========================================
-- ЛОГИКА TEAM GLOW ESP
-- ==========================================
local GlowState = false
local GlowStorage = {}
local LastSeenTimes = {}

local function GetTeamColor(player)
	if player.Team and player.Team.TeamColor then
		return player.Team.TeamColor.Color
	elseif player.TeamColor then
		return player.TeamColor.Color
	end
	return COLORS.AccentRed
end

local function RemoveGlow(player)
	if GlowStorage[player] then
		GlowStorage[player]:Destroy()
		GlowStorage[player] = nil
	end
	LastSeenTimes[player] = nil
end

local function ApplyGlow(player)
	if not GlowState or player == LocalPlayer or IsAlly(player) or not IsPlayerAlive(player) then
		RemoveGlow(player)
		return
	end

	local char = player.Character
	if not char then RemoveGlow(player) return end

	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if not humanoid or humanoid.Health <= 0 then RemoveGlow(player) return end

	local highlight = GlowStorage[player]
	if not highlight or highlight.Parent ~= char then
		RemoveGlow(player)

		highlight = Instance.new("Highlight")
		highlight.Name = GLOW_NAME
		highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		highlight.FillTransparency = 1
		highlight.OutlineTransparency = 0
		highlight.Parent = char
		GlowStorage[player] = highlight
	end

	local visible = IsVisible(char)
	local currentTime = os.clock()
	local targetColor

	if visible then
		LastSeenTimes[player] = currentTime
		targetColor = COLORS.VisibleGreen
	else
		local lastSeen = LastSeenTimes[player]
		if lastSeen and (currentTime - lastSeen <= 15) then
			targetColor = COLORS.BehindWallYellow
		else
			targetColor = GetTeamColor(player)
		end
	end

	if highlight.OutlineColor ~= targetColor then
		highlight.OutlineColor = targetColor
	end
end

task.spawn(function()
	while not Unloaded do
		if GlowState then
			for _, player in ipairs(Players:GetPlayers()) do
				if player ~= LocalPlayer and not IsAlly(player) then
					pcall(ApplyGlow, player)
				else
					RemoveGlow(player)
				end
			end
		end
		task.wait(0.05)
	end
end)

local function ToggleTeamGlow(state)
	GlowState = state
	if not GlowState then
		for player, _ in pairs(GlowStorage) do RemoveGlow(player) end
		table.clear(LastSeenTimes)
	end
end

-- ==========================================
-- RAPID FIRE, NO RECOIL, INF AMMO & SILENT AIM HOOKS
-- ==========================================
local ClientFireModule = nil
local originalFireVolley = nil

local RapidFireState = false
local RapidFireDelay = 0.03
local RapidFireHold = true
local rapidFireHolding = false
local rapidFireLastArgs = nil
local rapidFireLastShotTime = 0

local NoRecoilState = false
local ZeroSpreadState = true
local ZeroRecoilState = true
local ZeroFirstShotState = true
local ZeroVisualState = true

local InfAmmoState = false

table.insert(Connections, UserInputService.InputBegan:Connect(function(input, gpe)
	if not gpe and input.UserInputType == Enum.UserInputType.MouseButton1 then
		rapidFireHolding = true
		rapidFireLastShotTime = 0
	end
end))

table.insert(Connections, UserInputService.InputEnded:Connect(function(input, gpe)
	if not gpe and input.UserInputType == Enum.UserInputType.MouseButton1 then
		rapidFireHolding = false
		rapidFireLastArgs = nil
	end
end))

table.insert(Connections, RunService.RenderStepped:Connect(function()
	if RapidFireState and rapidFireHolding and rapidFireLastArgs and originalFireVolley then
		local currentTime = tick()
		if currentTime - rapidFireLastShotTime >= RapidFireDelay then
			pcall(function()
				originalFireVolley(unpack(rapidFireLastArgs))
			end)
			rapidFireLastShotTime = currentTime
		end
	end
end))

table.insert(Connections, RunService.RenderStepped:Connect(function()
	if InfAmmoState then
		local character = LocalPlayer.Character
		if character and IsPlayerAlive(LocalPlayer) then
			local tool = character:FindFirstChildOfClass("Tool")
			if tool then
				for _, v in ipairs(tool:GetDescendants()) do
					if v:IsA("IntValue") or v:IsA("NumberValue") then
						local lname = v.Name:lower()
						if lname:find("ammo") or lname:find("clip") or lname:find("bullet") or lname:find("mag") then
							v.Value = 9999
						end
					end
				end
			end
		end
	end
end))

task.spawn(function()
	while not Unloaded do
		task.wait(0.5)
		if (NoRecoilState or InfAmmoState) and LocalPlayer.Character then
			if NoRecoilState then
				for _, item in pairs(Workspace:GetChildren()) do
					if item.Name:find("Camera") or item.Name:find("Viewmodel") then
						for _, v in pairs(item:GetDescendants()) do
							if v:IsA("Vector3Value") and (v.Name:find("Recoil") or v.Name:find("Spring")) then
								v.Value = Vector3.new(0, 0, 0)
							end
						end
					end
				end
			end
			
			local playerScripts = LocalPlayer:FindFirstChild("PlayerScripts")
			if playerScripts then
				for _, module in pairs(playerScripts:GetDescendants()) do
					if module:IsA("ModuleScript") and (module.Name:find("Weapon") or module.Name:find("Gun") or module.Name:find("Stats") or module.Name:find("Ballistics")) then
						local success, result = pcall(require, module)
						if success and type(result) == "table" then
							if NoRecoilState then
								if ZeroSpreadState and result.Spread then result.Spread = 0 end
								if ZeroRecoilState and result.Recoil then result.Recoil = 0 end
								if ZeroFirstShotState and result.FirstShotRecoil then result.FirstShotRecoil = 0 end
								if ZeroVisualState and result.VisualRecoil then result.VisualRecoil = 0 end
							end
							if InfAmmoState then
								if result.Ammo then result.Ammo = 9999 end
								if result.MagSize then result.MagSize = 9999 end
								if result.MaxAmmo then result.MaxAmmo = 9999 end
								if result.Capacity then result.Capacity = 9999 end
								if result.StoredAmmo then result.StoredAmmo = 9999 end
							end
						end
					end
				end
			end
		end
	end
end)

local function IsSilentAimTeammate(vPlayer)
	if not SilentAimTeamCheck then return false end
	if vPlayer.Team and LocalPlayer.Team and vPlayer.Team == LocalPlayer.Team then return true end
	return false
end

local function IsSilentAimVisible(targetPart)
	if not SilentAimWallCheck then return true end
	local origin = CameraCache.CFrame.Position
	local direction = targetPart.Position - origin
	local raycastParams = RaycastParams.new()
	raycastParams.FilterType = Enum.RaycastFilterType.Exclude
	raycastParams.IgnoreWater = true
	local ignoreList = {}
	if LocalPlayer.Character then table.insert(ignoreList, LocalPlayer.Character) end
	if targetPart.Parent then table.insert(ignoreList, targetPart.Parent) end
	raycastParams.FilterDescendantsInstances = ignoreList
	return not Workspace:Raycast(origin, direction, raycastParams)
end

local function GetSilentAimTarget()
	local closest = nil
	local shortestDist = SilentAimFOV
	local screenSize = CameraCache.ViewportSize
	local screenCenter = Vector2.new(screenSize.X / 2, screenSize.Y / 2)

	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and not IsSilentAimTeammate(p) and IsPlayerAlive(p) and p.Character then
			local targetPart = p.Character:FindFirstChild(SilentAimTargetPart) or p.Character:FindFirstChild("Head")
			local hum = p.Character:FindFirstChildOfClass("Humanoid")
			local hrp = p.Character:FindFirstChild("HumanoidRootPart")
			local myHrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

			if targetPart and hum and hum.Health > 0 and hrp and myHrp then
				local worldDist = (myHrp.Position - hrp.Position).Magnitude
				if worldDist <= SilentAimMaxDistance then
					local screenPos, onScreen = CameraCache:WorldToViewportPoint(targetPart.Position)
					if onScreen and screenPos.Z > 0 and IsSilentAimVisible(targetPart) then
						local distFromCenter = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
						if distFromCenter < shortestDist then
							shortestDist = distFromCenter
							closest = targetPart
						end
					end
				end
			end
		end
	end
	return closest
end

-- ==========================================
-- TRACER SPAWN HELPER (для fireVolley)
-- ==========================================
local function SpawnTracersForVolley(origin, directions, weaponName)
	if not BulletTracersState then return end
	if not origin or not directions then return end
	
	local maxRange = 500
	for i = 1, #directions do
		local dir = directions[i]
		if typeof(dir) == "Vector3" then
			local endPos = origin + dir.Unit * maxRange
			-- Реальный raycast чтобы трассер не уходил сквозь стены далеко
			local rayParams = RaycastParams.new()
			rayParams.FilterType = Enum.RaycastFilterType.Exclude
			local ignoreList = {}
			if LocalPlayer.Character then table.insert(ignoreList, LocalPlayer.Character) end
			rayParams.FilterDescendantsInstances = ignoreList
			
			local result = Workspace:Raycast(origin, dir.Unit * maxRange, rayParams)
			if result then
				endPos = result.Position
			end
			
			CreateBulletTracer(origin, endPos)
		end
	end
end

local function SetupFireVolleyInterceptor()
	task.spawn(function()
		if Unloaded then return end
		local playerScripts = LocalPlayer:WaitForChild("PlayerScripts", 10)
		if not playerScripts then return end
		local ballistics = playerScripts:WaitForChild("BallisticsClient", 10)
		if not ballistics then return end
		local clientFire = ballistics:WaitForChild("ClientFire", 10)
		if not clientFire then return end
		
		ClientFireModule = require(clientFire)
		if not originalFireVolley then
			originalFireVolley = ClientFireModule.fireVolley
			ClientFireModule.fireVolley = function(weaponName, muzzleIndex, bulletIndex, origin, directions)
				-- 1. Сохранение аргументов для Rapid Fire
				if RapidFireState then
					rapidFireLastArgs = {weaponName, muzzleIndex, bulletIndex, origin, directions}
					rapidFireLastShotTime = tick()
				end

				-- 2. No Recoil (Zero Spread) направление пуль
				if NoRecoilState and ZeroSpreadState and CameraCache then
					local exactDirection = CameraCache.CFrame.LookVector
					for i = 1, #directions do
						directions[i] = exactDirection
					end
				end

				-- 3. Silent Aim перехват направлений + АКТИВАЦИЯ SHOT SYNC AA
				if SilentAimState and CameraCache then
					local targetPart = GetSilentAimTarget()
					if targetPart then
						local newDirection = (targetPart.Position - origin).Unit
						for i = 1, #directions do
							directions[i] = newDirection
						end
						
						-- Активируем Anti-Aim shot-sync на 0.1 сек
						ActivateAAShotSync(targetPart)
					end
				end

				-- 4. Спавн трассеров (используем финальные directions)
				SpawnTracersForVolley(origin, directions, weaponName)

				return originalFireVolley(weaponName, muzzleIndex, bulletIndex, origin, directions)
			end
		end
	end)
end

SetupFireVolleyInterceptor()

-- ==========================================
-- СИСТЕМА КОНФИГОВ И ПРИМЕНЕНИЕ ТЕМ
-- ==========================================
local ConfigSystem = {
	Configs = {
		Legit = {
			name = "Legit Config",
			theme = "Birch",
			desc = "Березовая тема | RMB, Predict 7ms, FOV 65, Smooth 30, ESP, No Recoil",
			settings = {
				AimbotState = true,
				AimbotKeyName = "Mouse 2 (RMB)",
				AimbotPrediction = true,
				AimbotPredictionAmount = 0.007,
				DistanceCompensateState = true,
				RandomHitpartState = true,
				AimbotFOV = 65,
				AimbotTargetPart = "Head",
				AimbotSpeed = 30,
				GlowState = true,
				FriendHatState = true,
				FriendESPState = true,
				WorldSaturation = 3,
				RemoveMuzzleState = true,
				NoFogState = false,
				SilentAimState = false,
				RapidFireState = false,
				NoRecoilState = true,
				ZeroSpreadState = true,
				ZeroRecoilState = true,
				ZeroFirstShotState = true,
				ZeroVisualState = true,
				InfAmmoState = false,
				DrawFOVState = false,
				SpeedPlusState = false,
				SpeedPlusForce = 0.5,
				AA_Enabled = false,
				AA_Pitch = 89,
				AA_YawOffset = 180,
				AA_Spin = false,
				AA_SpinSpeed = 15,
				AA_Jitter = false,
				AA_JitterAngle = 45,
				AA_JitterDelay = 0.1,
				AA_Snapback = true,
				AA_ShotSync = true,
				AA_ShotSyncDuration = 0.1,
				BulletTracersState = true,
				BulletTracersLifeTime = 5
			}
		},
		Medium = {
			name = "Medium Config",
			theme = "Yellow",
			desc = "Желтая тема | LMB, Predict 7ms, Silent Aim 50px, No Recoil + Rapid Fire + Inf Ammo",
			settings = {
				AimbotState = true,
				AimbotKeyName = "Mouse 1 (LMB)",
				AimbotPrediction = true,
				AimbotPredictionAmount = 0.007,
				AimbotFOV = 65,
				AimbotTargetPart = "Head",
				AimbotSpeed = 45,
				SilentAimState = true,
				SilentAimFOV = 50,
				SilentAimTargetPart = "Head",
				SilentAimWallCheck = true,
				SilentAimTeamCheck = true,
				SilentAimMaxDistance = 1000,
				RapidFireState = true,
				RapidFireDelay = 0.03,
				NoRecoilState = true,
				ZeroSpreadState = true,
				ZeroRecoilState = true,
				ZeroFirstShotState = true,
				ZeroVisualState = true,
				InfAmmoState = true,
				GlowState = true,
				FriendHatState = true,
				FriendESPState = true,
				WorldSaturation = 3,
				RemoveMuzzleState = true,
				NoFogState = false,
				DistanceCompensateState = false,
				RandomHitpartState = false,
				DrawFOVState = false,
				SpeedPlusState = false,
				SpeedPlusForce = 0.5,
				AA_Enabled = false,
				AA_Pitch = 89,
				AA_YawOffset = 180,
				AA_Spin = false,
				AA_SpinSpeed = 15,
				AA_Jitter = false,
				AA_JitterAngle = 45,
				AA_JitterDelay = 0.1,
				AA_Snapback = true,
				AA_ShotSync = true,
				AA_ShotSyncDuration = 0.1,
				BulletTracersState = true,
				BulletTracersLifeTime = 5
			}
		},
		SemiRage = {
			name = "Semi-Rage Config",
			theme = "Blue",
			desc = "Синяя тема | FOV 100, Silent 110px, Rapid Fire, No Recoil, Inf Ammo, No Fog",
			settings = {
				AimbotState = true,
				AimbotKeyName = "Mouse 1 (LMB)",
				AimbotPrediction = true,
				AimbotPredictionAmount = 0.005,
				AimbotFOV = 100,
				DrawFOVState = true,
				AimbotTargetPart = "Head",
				AimbotSpeed = 70,
				SilentAimState = true,
				SilentAimFOV = 110,
				SilentAimTargetPart = "Head",
				SilentAimWallCheck = true,
				SilentAimTeamCheck = true,
				SilentAimMaxDistance = 1000,
				RapidFireState = true,
				RapidFireDelay = 0.03,
				NoRecoilState = true,
				ZeroSpreadState = true,
				ZeroRecoilState = true,
				ZeroFirstShotState = true,
				ZeroVisualState = true,
				InfAmmoState = true,
				GlowState = true,
				FriendHatState = true,
				FriendESPState = true,
				WorldSaturation = 3,
				RemoveMuzzleState = true,
				NoFogState = true,
				DistanceCompensateState = false,
				RandomHitpartState = false,
				SpeedPlusState = false,
				SpeedPlusForce = 0.5,
				AA_Enabled = true,
				AA_Pitch = 89,
				AA_YawOffset = 180,
				AA_Spin = true,
				AA_SpinSpeed = 20,
				AA_Jitter = true,
				AA_JitterAngle = 45,
				AA_JitterDelay = 0.1,
				AA_Snapback = false,
				AA_ShotSync = true,
				AA_ShotSyncDuration = 0.1,
				BulletTracersState = true,
				BulletTracersLifeTime = 5
			}
		},
		Rage = {
			name = "Rage Config",
			theme = "White",
			desc = "Белая тема | Максимальный Rage, Rapid Fire 30мс, Полный No Recoil + Inf Ammo + AA",
			settings = {
				AimbotState = true,
				AimbotKeyName = "Mouse 1 (LMB)",
				AimbotPrediction = true,
				AimbotPredictionAmount = 0.005,
				AimbotFOV = 800,
				AimbotTargetPart = "Head",
				AimbotSpeed = 100,
				SilentAimState = true,
				SilentAimFOV = 800,
				SilentAimTargetPart = "Head",
				SilentAimWallCheck = true,
				SilentAimTeamCheck = true,
				SilentAimMaxDistance = 5000,
				RapidFireState = true,
				RapidFireDelay = 0.025,
				NoRecoilState = true,
				ZeroSpreadState = true,
				ZeroRecoilState = true,
				ZeroFirstShotState = true,
				ZeroVisualState = true,
				InfAmmoState = true,
				GlowState = true,
				FriendHatState = true,
				FriendESPState = true,
				WorldSaturation = 3,
				RemoveMuzzleState = true,
				NoFogState = true,
				DistanceCompensateState = false,
				RandomHitpartState = false,
				DrawFOVState = false,
				SpeedPlusState = false,
				SpeedPlusForce = 0.5,
				AA_Enabled = true,
				AA_Pitch = 89,
				AA_YawOffset = 180,
				AA_Spin = true,
				AA_SpinSpeed = 30,
				AA_Jitter = true,
				AA_JitterAngle = 60,
				AA_JitterDelay = 0.08,
				AA_Snapback = false,
				AA_ShotSync = true,
				AA_ShotSyncDuration = 0.1,
				BulletTracersState = true,
				BulletTracersLifeTime = 5
			}
		}
	}
}

local ApplyTheme

local function ApplyConfig(configKey)
	local config = ConfigSystem.Configs[configKey]
	if not config then return end
	
	local s = config.settings
	
	if config.theme and ApplyTheme then
		ApplyTheme(config.theme)
	end

	AimbotState = s.AimbotState or false
	AimbotKeyName = s.AimbotKeyName or "Mouse 1 (LMB)"
	AimbotPrediction = s.AimbotPrediction or false
	AimbotPredictionAmount = s.AimbotPredictionAmount or 0.007
	DistanceCompensateState = s.DistanceCompensateState or false
	RandomHitpartState = s.RandomHitpartState or false
	AimbotFOV = s.AimbotFOV or 65
	AimbotTargetPart = s.AimbotTargetPart or "Head"
	AimbotSpeed = s.AimbotSpeed or 30
	DrawFOVState = s.DrawFOVState or false
	
	SilentAimState = s.SilentAimState or false
	SilentAimFOV = s.SilentAimFOV or 50
	SilentAimTargetPart = s.SilentAimTargetPart or "Head"
	SilentAimWallCheck = (s.SilentAimWallCheck ~= nil) and s.SilentAimWallCheck or true
	SilentAimTeamCheck = (s.SilentAimTeamCheck ~= nil) and s.SilentAimTeamCheck or true
	SilentAimMaxDistance = s.SilentAimMaxDistance or 1000

	RapidFireState = s.RapidFireState or false
	RapidFireDelay = s.RapidFireDelay or 0.03

	NoRecoilState = s.NoRecoilState or false
	ZeroSpreadState = (s.ZeroSpreadState ~= nil) and s.ZeroSpreadState or true
	ZeroRecoilState = (s.ZeroRecoilState ~= nil) and s.ZeroRecoilState or true
	ZeroFirstShotState = (s.ZeroFirstShotState ~= nil) and s.ZeroFirstShotState or true
	ZeroVisualState = (s.ZeroVisualState ~= nil) and s.ZeroVisualState or true
	
	InfAmmoState = s.InfAmmoState or false
	
	SpeedPlusState = s.SpeedPlusState or false
	SpeedPlusForce = s.SpeedPlusForce or 0.5

	AA_Enabled = s.AA_Enabled or false
	AA_Pitch = s.AA_Pitch or 89
	AA_YawOffset = s.AA_YawOffset or 180
	AA_Spin = s.AA_Spin or false
	AA_SpinSpeed = s.AA_SpinSpeed or 15
	AA_Jitter = s.AA_Jitter or false
	AA_JitterAngle = s.AA_JitterAngle or 45
	AA_JitterDelay = s.AA_JitterDelay or 0.1
	AA_Snapback = (s.AA_Snapback ~= nil) and s.AA_Snapback or true
	AA_ShotSync = (s.AA_ShotSync ~= nil) and s.AA_ShotSync or true
	AA_ShotSyncDuration = s.AA_ShotSyncDuration or 0.1

	BulletTracersState = (s.BulletTracersState ~= nil) and s.BulletTracersState or true
	BulletTracersLifeTime = s.BulletTracersLifeTime or 5

	ToggleTeamGlow(s.GlowState or false)
	ToggleFriendHat(s.FriendHatState or false)
	ToggleFriendESP(s.FriendESPState or false)
	
	GraphicsEffects.ColorCorrection.Saturation = s.WorldSaturation or 0
	ToggleRemoveMuzzle(s.RemoveMuzzleState or false)
	ToggleNoFog(s.NoFogState or false)
	
	PlaySound(SOUNDS.Button, 1.0)
end

-- ==========================================
-- ВЫГРУЗКА СКРИПТА (UNLOAD)
-- ==========================================
local function Unload()
	Unloaded = true
	CrosshairState = false
	ShowDistanceState = false
	SpeedPlusState = false
	RapidFireState = false
	NoRecoilState = false
	InfAmmoState = false
	AA_Enabled = false
	CloseActiveDropdown()
	UpdateCrosshair()

	if FOVCircle then pcall(function() FOVCircle:Remove() end) end
	if PotatoState then SetPotatoGraphics(false) end
	if NoFogState then ToggleNoFog(false) end
	if RemoveMuzzleState then ToggleRemoveMuzzle(false) end
	if FriendHatState then ToggleFriendHat(false) end
	if FriendESPState then ToggleFriendESP(false) end

	local char = LocalPlayer.Character
	if char then
		local hrp = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hrp then CleanupAA(hrp, hum) end
		local waist = char:FindFirstChild("UpperTorso") and char.UpperTorso:FindFirstChild("Waist")
		local neck = char:FindFirstChild("Head") and char.Head:FindFirstChild("Neck")
		if waist and originalC0s[waist] then waist.C0 = originalC0s[waist] end
		if neck and originalC0s[neck] then neck.C0 = originalC0s[neck] end
	end

	-- Удаление всех трассеров
	for _, data in ipairs(ActiveTracers) do
		if data.Part then data.Part:Destroy() end
	end
	table.clear(ActiveTracers)

	if GraphicsEffects then
		for _, fx in pairs(GraphicsEffects) do if fx then fx:Destroy() end end
	end

	for _, conn in ipairs(Connections) do
		if conn and conn.Connected then conn:Disconnect() end
	end
	table.clear(Connections)

	if FullBrightState then
		FullBrightState = false
		RestoreLighting()
	end

	ToggleTeamGlow(false)

	if ClientFireModule and originalFireVolley then
		ClientFireModule.fireVolley = originalFireVolley
	end

	if CrosshairContainer then CrosshairContainer:Destroy() end
	if ScreenGui then ScreenGui:Destroy() end
end

-- ==========================================
-- ОСНОВНОЕ ОКНО GUI
-- ==========================================
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 560, 0, 390)
MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.BackgroundColor3 = COLORS.Background
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
local MainStroke = Instance.new("UIStroke")
MainStroke.Color = COLORS.BorderRed
MainStroke.Thickness = 1.2
MainStroke.Transparency = 0.2
MainStroke.Parent = MainFrame

local UIScale = Instance.new("UIScale")
UIScale.Scale = 0
UIScale.Parent = MainFrame

local dragging = false
local dragStart, startPos

MainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
	end
end)

table.insert(Connections, UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		CloseActiveDropdown()
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end))

table.insert(Connections, UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end))

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -12)
Sidebar.Position = UDim2.new(0, 6, 0, 6)
Sidebar.BackgroundTransparency = 1
Sidebar.Parent = MainFrame

local LogoFrame = Instance.new("Frame")
LogoFrame.Size = UDim2.new(1, 0, 0, 36)
LogoFrame.BackgroundColor3 = COLORS.CardBg
LogoFrame.Parent = Sidebar
Instance.new("UICorner", LogoFrame).CornerRadius = UDim.new(0, 6)

local LogoStroke = Instance.new("UIStroke")
LogoStroke.Color = COLORS.BorderRed
LogoStroke.Thickness = 1
LogoStroke.Transparency = 0.4
LogoStroke.Parent = LogoFrame

local LogoText = Instance.new("TextLabel")
LogoText.Size = UDim2.new(1, 0, 1, 0)
LogoText.BackgroundTransparency = 1
LogoText.Text = "N A I M"
LogoText.TextColor3 = COLORS.AccentRed
LogoText.TextSize = 14
LogoText.Font = FONTS.Title
LogoText.Parent = LogoFrame

local VerticalDivider = Instance.new("Frame")
VerticalDivider.Size = UDim2.new(0, 1, 1, -20)
VerticalDivider.Position = UDim2.new(0, 142, 0, 10)
VerticalDivider.BackgroundColor3 = COLORS.BorderDim
VerticalDivider.BorderSizePixel = 0
VerticalDivider.Parent = MainFrame

local PlayerCard = Instance.new("Frame")
PlayerCard.Size = UDim2.new(1, 0, 0, 42)
PlayerCard.Position = UDim2.new(0, 0, 1, -42)
PlayerCard.BackgroundColor3 = COLORS.CardBg
PlayerCard.Parent = Sidebar
Instance.new("UICorner", PlayerCard).CornerRadius = UDim.new(0, 6)

local AvatarImg = Instance.new("ImageLabel")
AvatarImg.Size = UDim2.new(0, 28, 0, 28)
AvatarImg.Position = UDim2.new(0, 6, 0.5, -14)
AvatarImg.BackgroundColor3 = COLORS.ToggleOff
AvatarImg.Image = "rbxassetid://6031075929"
AvatarImg.Parent = PlayerCard
Instance.new("UICorner", AvatarImg).CornerRadius = UDim.new(1, 0)

task.spawn(function()
	pcall(function()
		AvatarImg.Image = Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
	end)
end)

local PlayerDisplayName = Instance.new("TextLabel")
PlayerDisplayName.Size = UDim2.new(1, -40, 0, 14)
PlayerDisplayName.Position = UDim2.new(0, 38, 0, 6)
PlayerDisplayName.BackgroundTransparency = 1
PlayerDisplayName.Text = LocalPlayer.DisplayName
PlayerDisplayName.TextColor3 = COLORS.TextWhite
PlayerDisplayName.TextXAlignment = Enum.TextXAlignment.Left
PlayerDisplayName.TextTruncate = Enum.TextTruncate.AtEnd
PlayerDisplayName.TextSize = 11
PlayerDisplayName.Font = FONTS.Title
PlayerDisplayName.Parent = PlayerCard

local PlayerUsername = Instance.new("TextLabel")
PlayerUsername.Size = UDim2.new(1, -40, 0, 12)
PlayerUsername.Position = UDim2.new(0, 38, 0, 22)
PlayerUsername.BackgroundTransparency = 1
PlayerUsername.Text = "@" .. LocalPlayer.Name
PlayerUsername.TextColor3 = COLORS.TextDim
PlayerUsername.TextXAlignment = Enum.TextXAlignment.Left
PlayerUsername.TextTruncate = Enum.TextTruncate.AtEnd
PlayerUsername.TextSize = 10
PlayerUsername.Font = FONTS.Body
PlayerUsername.Parent = PlayerCard

local TAB_NAMES = {"Aimbot", "Weapon Mods", "Anti-Aim", "Visuals", "Graphics", "Optimization", "Venchine mods", "Misc", "Settings"}
local TabHolder = Instance.new("Frame")
TabHolder.Size = UDim2.new(1, 0, 1, -94)
TabHolder.Position = UDim2.new(0, 0, 0, 42)
TabHolder.BackgroundTransparency = 1
TabHolder.Parent = Sidebar

local TabList = Instance.new("UIListLayout")
TabList.Padding = UDim.new(0, 4)
TabList.SortOrder = Enum.SortOrder.LayoutOrder
TabList.Parent = TabHolder

local HeaderFrame = Instance.new("Frame")
HeaderFrame.Size = UDim2.new(1, -152, 0, 32)
HeaderFrame.Position = UDim2.new(0, 148, 0, 6)
HeaderFrame.BackgroundColor3 = COLORS.CardBg
HeaderFrame.Parent = MainFrame
Instance.new("UICorner", HeaderFrame).CornerRadius = UDim.new(0, 6)

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -16, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "NAIM V3"
TitleLabel.TextColor3 = COLORS.TextWhite
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.TextSize = 12
TitleLabel.Font = FONTS.Title
TitleLabel.Parent = HeaderFrame

local ContentArea = Instance.new("Frame")
ContentArea.Size = UDim2.new(1, -152, 1, -48)
ContentArea.Position = UDim2.new(0, 148, 0, 42)
ContentArea.BackgroundTransparency = 1
ContentArea.ClipsDescendants = true
ContentArea.Parent = MainFrame

local TabsData = {}
local CurrentTabName = "Aimbot"

local TAB_INDICATOR_TWEEN = TweenInfo.new(0.22 / 0.6, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
local TAB_BUTTON_TWEEN = TweenInfo.new(0.18 / 0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

local function TweenTabIndicator(data, active)
	if data.IndicatorTween then
		data.IndicatorTween:Cancel()
		data.IndicatorTween = nil
	end

	local goal = active and { Size = UDim2.new(0, 3, 0.6, 0), BackgroundTransparency = 0, BackgroundColor3 = COLORS.AccentRed } 
	                    or { Size = UDim2.new(0, 3, 0, 0), BackgroundTransparency = 1 }

	local tween = TweenService:Create(data.Indicator, TAB_INDICATOR_TWEEN, goal)
	data.IndicatorTween = tween
	tween.Completed:Connect(function()
		if data.IndicatorTween == tween then data.IndicatorTween = nil end
	end)
	tween:Play()
end

local function SwitchTab(targetName)
	if not TabsData[targetName] then return end
	CloseActiveDropdown()
	if CurrentTabName ~= targetName then PlaySound(SOUNDS.Tab, 0.8) end
	CurrentTabName = targetName

	for name, data in pairs(TabsData) do
		local isActive = name == targetName
		TweenService:Create(data.Button, TAB_BUTTON_TWEEN, {
			BackgroundColor3 = isActive and COLORS.AccentDarkRed or COLORS.CardBg,
			TextColor3 = isActive and COLORS.TextWhite or COLORS.TextDim
		}):Play()

		TweenTabIndicator(data, isActive)
		data.Page.Visible = isActive
	end
end

ApplyTheme = function(themeName)
	local theme = THEMES[themeName]
	if not theme then return end

	CurrentThemeName = themeName
	CurrentTheme = theme

	COLORS.AccentRed = theme.Accent
	COLORS.BorderRed = theme.Border
	COLORS.AccentDarkRed = theme.DarkAccent
	COLORS.TextRed = theme.Accent

	MainStroke.Color = theme.Border
	LogoStroke.Color = theme.Border
	LogoText.TextColor3 = theme.Accent

	for _, stroke in ipairs(DynamicThemeElements.Borders) do stroke.Color = theme.Border end
	for _, elem in ipairs(DynamicThemeElements.Accents) do
		if elem:IsA("TextLabel") then elem.TextColor3 = theme.Accent
		elseif elem:IsA("Frame") or elem:IsA("TextButton") then elem.BackgroundColor3 = theme.Accent end
	end
	for _, elem in ipairs(DynamicThemeElements.DarkAccents) do elem.BackgroundColor3 = theme.DarkAccent end
	for _, toggleData in ipairs(DynamicThemeElements.Toggles) do
		toggleData.Switch.BackgroundColor3 = toggleData.GetState() and theme.Accent or COLORS.ToggleOff
	end
	for _, scrollBar in ipairs(DynamicThemeElements.ScrollBars) do
		if scrollBar and scrollBar.Parent then
			scrollBar.ScrollBarImageColor3 = theme.Border
		end
	end

	-- Обновляем цвет активных трассеров под новую тему
	for _, data in ipairs(ActiveTracers) do
		if data.Part and data.Part.Parent then
			data.Part.Color = theme.Accent
		end
	end

	SwitchTab(CurrentTabName)
end

local function CreateTab(name)
	local TabButton = Instance.new("TextButton")
	TabButton.Size = UDim2.new(1, 0, 0, 28)
	TabButton.BackgroundColor3 = COLORS.CardBg
	TabButton.Text = "     " .. name
	TabButton.TextColor3 = COLORS.TextDim
	TabButton.Font = FONTS.Title
	TabButton.TextSize = 11
	TabButton.TextXAlignment = Enum.TextXAlignment.Left
	TabButton.AutoButtonColor = false
	TabButton.Parent = TabHolder
	Instance.new("UICorner", TabButton).CornerRadius = UDim.new(0, 5)

	local ActiveIndicator = Instance.new("Frame")
	ActiveIndicator.Size = UDim2.new(0, 3, 0, 0)
	ActiveIndicator.Position = UDim2.new(0, 4, 0.5, 0)
	ActiveIndicator.AnchorPoint = Vector2.new(0, 0.5)
	ActiveIndicator.BackgroundColor3 = COLORS.AccentRed
	ActiveIndicator.BorderSizePixel = 0
	ActiveIndicator.BackgroundTransparency = 1
	ActiveIndicator.Parent = TabButton
	Instance.new("UICorner", ActiveIndicator).CornerRadius = UDim.new(1, 0)

	local TabPage = Instance.new("ScrollingFrame")
	TabPage.Size = UDim2.new(1, 0, 1, 0)
	TabPage.BackgroundTransparency = 1
	TabPage.BorderSizePixel = 0
	TabPage.ScrollBarThickness = 3
	TabPage.ScrollBarImageColor3 = COLORS.BorderRed
	TabPage.AutomaticCanvasSize = Enum.AutomaticSize.Y
	TabPage.CanvasSize = UDim2.new(0, 0, 0, 0)
	TabPage.Visible = false
	TabPage.ClipsDescendants = true
	TabPage.Parent = ContentArea

	table.insert(DynamicThemeElements.ScrollBars, TabPage)

	local PagePadding = Instance.new("UIPadding")
	PagePadding.PaddingTop = UDim.new(0, 2)
	PagePadding.PaddingBottom = UDim.new(0, 10)
	PagePadding.PaddingLeft = UDim.new(0, 2)
	PagePadding.PaddingRight = UDim.new(0, 8)
	PagePadding.Parent = TabPage

	local Layout = Instance.new("UIListLayout")
	Layout.Padding = UDim.new(0, 8)
	Layout.SortOrder = Enum.SortOrder.LayoutOrder
	Layout.Parent = TabPage

	TabPage:GetPropertyChangedSignal("CanvasPosition"):Connect(CloseActiveDropdown)

	TabsData[name] = {
		Button = TabButton,
		Indicator = ActiveIndicator,
		Page = TabPage,
		IndicatorTween = nil
	}
	TabButton.MouseButton1Click:Connect(function() SwitchTab(name) end)
	return TabPage
end

local Pages = {}
for _, tabName in ipairs(TAB_NAMES) do Pages[tabName] = CreateTab(tabName) end

-- ==========================================
-- КОМПОНЕНТЫ ИНТЕРФЕЙСА
-- ==========================================
local function CreateCard(parent, title)
	local Card = Instance.new("Frame")
	Card.Size = UDim2.new(1, 0, 0, 0)
	Card.AutomaticSize = Enum.AutomaticSize.Y
	Card.BackgroundColor3 = COLORS.CardBg
	Card.ClipsDescendants = true
	Card.Parent = parent

	Instance.new("UICorner", Card).CornerRadius = UDim.new(0, 6)
	local CardPadding = Instance.new("UIPadding")
	CardPadding.PaddingTop = UDim.new(0, 8)
	CardPadding.PaddingBottom = UDim.new(0, 10)
	CardPadding.PaddingLeft = UDim.new(0, 10)
	CardPadding.PaddingRight = UDim.new(0, 10)
	CardPadding.Parent = Card

	local Stroke = Instance.new("UIStroke")
	Stroke.Color = COLORS.BorderDim
	Stroke.Thickness = 1
	Stroke.Parent = Card

	local List = Instance.new("UIListLayout")
	List.Padding = UDim.new(0, 8)
	List.SortOrder = Enum.SortOrder.LayoutOrder
	List.Parent = Card

	local CardTitle = Instance.new("TextLabel")
	CardTitle.Size = UDim2.new(1, 0, 0, 16)
	CardTitle.BackgroundTransparency = 1
	CardTitle.Text = title
	CardTitle.TextColor3 = COLORS.AccentRed
	CardTitle.TextXAlignment = Enum.TextXAlignment.Left
	CardTitle.TextSize = 12
	CardTitle.Font = FONTS.Title
	CardTitle.Parent = Card

	table.insert(DynamicThemeElements.Accents, CardTitle)
	return Card
end

local function CreateToggle(parent, text, defaultState, callback)
	local Item = Instance.new("Frame")
	Item.Size = UDim2.new(1, 0, 0, 22)
	Item.BackgroundTransparency = 1
	Item.Parent = parent

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(1, -38, 1, 0)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.TextColor3 = defaultState and COLORS.TextWhite or COLORS.TextDim
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.TextSize = 11
	Label.Font = FONTS.Body
	Label.Parent = Item

	local Switch = Instance.new("TextButton")
	Switch.Size = UDim2.new(0, 30, 0, 14)
	Switch.Position = UDim2.new(1, -30, 0.5, -7)
	Switch.BackgroundColor3 = defaultState and COLORS.AccentRed or COLORS.ToggleOff
	Switch.Text = ""
	Switch.AutoButtonColor = false
	Switch.Parent = Item
	Instance.new("UICorner", Switch).CornerRadius = UDim.new(1, 0)

	local Thumb = Instance.new("Frame")
	Thumb.Size = UDim2.new(0, 10, 0, 10)
	Thumb.Position = defaultState and UDim2.new(1, -12, 0.5, -5) or UDim2.new(0, 2, 0.5, -5)
	Thumb.BackgroundColor3 = COLORS.TextWhite
	Thumb.Parent = Switch
	Instance.new("UICorner", Thumb).CornerRadius = UDim.new(1, 0)

	local state = defaultState
	Switch.MouseButton1Click:Connect(function()
		state = not state
		PlaySound(SOUNDS.Toggle, 0.9)
		TweenService:Create(Thumb, TweenInfo.new(0.12), {Position = state and UDim2.new(1, -12, 0.5, -5) or UDim2.new(0, 2, 0.5, -5)}):Play()
		TweenService:Create(Switch, TweenInfo.new(0.12), {BackgroundColor3 = state and COLORS.AccentRed or COLORS.ToggleOff}):Play()
		TweenService:Create(Label, TweenInfo.new(0.12), {TextColor3 = state and COLORS.TextWhite or COLORS.TextDim}):Play()
		if callback then callback(state) end
	end)

	table.insert(DynamicThemeElements.Toggles, {
		Switch = Switch,
		GetState = function() return state end
	})
end

local function CreateSlider(parent, text, min, max, default, callback, suffix, isFloat)
	suffix = suffix or "x"
	local Item = Instance.new("Frame")
	Item.Size = UDim2.new(1, 0, 0, 32)
	Item.BackgroundTransparency = 1
	Item.Parent = parent

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(1, -65, 0, 14)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.TextColor3 = COLORS.TextWhite
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.TextSize = 11
	Label.Font = FONTS.Body
	Label.Parent = Item

	local ValLabel = Instance.new("TextLabel")
	ValLabel.Size = UDim2.new(0, 60, 0, 14)
	ValLabel.Position = UDim2.new(1, -60, 0, 0)
	ValLabel.BackgroundTransparency = 1
	ValLabel.Text = isFloat and (string.format("%.2f", default) .. suffix) or (tostring(default) .. suffix)
	ValLabel.TextColor3 = COLORS.TextDim
	ValLabel.TextXAlignment = Enum.TextXAlignment.Right
	ValLabel.TextSize = 11
	ValLabel.Font = FONTS.Body
	ValLabel.Parent = Item

	local Track = Instance.new("TextButton")
	Track.Size = UDim2.new(1, 0, 0, 7)
	Track.Position = UDim2.new(0, 0, 0, 20)
	Track.BackgroundColor3 = COLORS.ToggleOff
	Track.Text = ""
	Track.AutoButtonColor = false
	Track.Parent = Item
	Instance.new("UICorner", Track).CornerRadius = UDim.new(1, 0)

	local Fill = Instance.new("Frame")
	Fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
	Fill.BackgroundColor3 = COLORS.AccentRed
	Fill.BorderSizePixel = 0
	Fill.Parent = Track
	Instance.new("UICorner", Fill).CornerRadius = UDim.new(1, 0)

	table.insert(DynamicThemeElements.Accents, Fill)

	local sliding = false
	local function UpdateSlider(input)
		local pos = math.clamp((input.Position.X - Track.AbsolutePosition.X) / Track.AbsoluteSize.X, 0, 1)
		local value
		if isFloat then
			value = math.floor((min + (max - min) * pos) * 1000 + 0.5) / 1000
			ValLabel.Text = string.format("%.3f", value) .. suffix
		else
			value = math.floor(min + (max - min) * pos)
			ValLabel.Text = tostring(value) .. suffix
		end
		Fill.Size = UDim2.new(pos, 0, 1, 0)
		if callback then callback(value) end
	end

	Track.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			sliding = true
			UpdateSlider(input)
		end
	end)

	table.insert(Connections, UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end
	end))

	table.insert(Connections, UserInputService.InputChanged:Connect(function(input)
		if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then UpdateSlider(input) end
	end))
end

local function CreateDropdown(parent, text, options, default, callback)
	local Item = Instance.new("Frame")
	Item.Size = UDim2.new(1, 0, 0, 24)
	Item.BackgroundTransparency = 1
	Item.Parent = parent

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(0.45, 0, 0, 24)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.TextColor3 = COLORS.TextWhite
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.TextSize = 11
	Label.Font = FONTS.Body
	Label.Parent = Item

	local DropBtn = Instance.new("TextButton")
	DropBtn.Size = UDim2.new(0.55, 0, 0, 24)
	DropBtn.Position = UDim2.new(0.45, 0, 0, 0)
	DropBtn.BackgroundColor3 = COLORS.ToggleOff
	DropBtn.Text = default .. "  ▼"
	DropBtn.TextColor3 = COLORS.TextWhite
	DropBtn.TextSize = 11
	DropBtn.Font = FONTS.Title
	DropBtn.AutoButtonColor = false
	DropBtn.Parent = Item
	Instance.new("UICorner", DropBtn).CornerRadius = UDim.new(0, 4)

	local itemHeight = 22
	local maxVisible = 4
	local containerHeight = math.min(#options, maxVisible) * itemHeight

	local OptionContainer = Instance.new("ScrollingFrame")
	OptionContainer.Size = UDim2.new(0, 120, 0, containerHeight)
	OptionContainer.BackgroundColor3 = COLORS.CardBg
	OptionContainer.BorderSizePixel = 0
	OptionContainer.Visible = false
	OptionContainer.ZIndex = 200
	OptionContainer.ScrollBarThickness = 3
	OptionContainer.ScrollBarImageColor3 = COLORS.BorderRed
	OptionContainer.CanvasSize = UDim2.new(0, 0, 0, #options * itemHeight)
	OptionContainer.ClipsDescendants = true
	OptionContainer.Parent = DropdownOverlay
	Instance.new("UICorner", OptionContainer).CornerRadius = UDim.new(0, 4)

	table.insert(DynamicThemeElements.ScrollBars, OptionContainer)

	local ContainerStroke = Instance.new("UIStroke")
	ContainerStroke.Color = COLORS.BorderRed
	ContainerStroke.Thickness = 1
	ContainerStroke.Parent = OptionContainer
	table.insert(DynamicThemeElements.Borders, ContainerStroke)

	local ListLayout = Instance.new("UIListLayout")
	ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
	ListLayout.Parent = OptionContainer

	local open = false
	local selected = default

	local function CloseThis()
		open = false
		OptionContainer.Visible = false
		DropBtn.Text = selected .. "  ▼"
		if ActiveDropdown and ActiveDropdown.Container == OptionContainer then
			ActiveDropdown = nil
		end
	end

	DropBtn.MouseButton1Click:Connect(function()
		PlaySound(SOUNDS.Dropdown, 0.7)
		if open then
			CloseThis()
		else
			CloseActiveDropdown()
			open = true
			local btnPos = DropBtn.AbsolutePosition
			local btnSize = DropBtn.AbsoluteSize
			OptionContainer.Position = UDim2.fromOffset(btnPos.X, btnPos.Y + btnSize.Y + 2)
			OptionContainer.Size = UDim2.fromOffset(btnSize.X, containerHeight)
			OptionContainer.Visible = true
			DropBtn.Text = selected .. "  ▲"
			ActiveDropdown = { Container = OptionContainer, Close = CloseThis }
		end
	end)

	for _, opt in ipairs(options) do
		local OptBtn = Instance.new("TextButton")
		OptBtn.Size = UDim2.new(1, 0, 0, itemHeight)
		OptBtn.BackgroundTransparency = 1
		OptBtn.Text = opt
		OptBtn.TextColor3 = (opt == default) and COLORS.AccentRed or COLORS.TextWhite
		OptBtn.TextSize = 10
		OptBtn.Font = FONTS.Body
		OptBtn.ZIndex = 201
		OptBtn.Parent = OptionContainer

		OptBtn.MouseButton1Click:Connect(function()
			selected = opt
			PlaySound(SOUNDS.Dropdown, 0.8)
			CloseThis()
			for _, child in ipairs(OptionContainer:GetChildren()) do
				if child:IsA("TextButton") then
					child.TextColor3 = (child.Text == selected) and COLORS.AccentRed or COLORS.TextWhite
				end
			end
			if callback then callback(selected) end
		end)
	end
end

local function CreateButton(parent, text, callback)
	local Item = Instance.new("Frame")
	Item.Size = UDim2.new(1, 0, 0, 26)
	Item.BackgroundTransparency = 1
	Item.Parent = parent

	local Button = Instance.new("TextButton")
	Button.Size = UDim2.new(1, 0, 1, 0)
	Button.BackgroundColor3 = COLORS.AccentDarkRed
	Button.Text = text
	Button.TextColor3 = COLORS.TextWhite
	Button.TextSize = 11
	Button.Font = FONTS.Title
	Button.AutoButtonColor = false
	Button.Parent = Item

	Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 5)

	local Stroke = Instance.new("UIStroke")
	Stroke.Color = COLORS.BorderRed
	Stroke.Thickness = 1
	Stroke.Transparency = 0.4
	Stroke.Parent = Button

	table.insert(DynamicThemeElements.DarkAccents, Button)
	table.insert(DynamicThemeElements.Borders, Stroke)

	Button.MouseButton1Click:Connect(function()
		PlaySound(SOUNDS.Button, 1.0)
		if callback then callback() end
	end)
end

local function CreateConfigCardButton(parent, configKey)
	local config = ConfigSystem.Configs[configKey]
	if not config then return end

	local themeObj = THEMES[config.theme] or THEMES.Birch

	local ButtonCard = Instance.new("Frame")
	ButtonCard.Size = UDim2.new(1, 0, 0, 42)
	ButtonCard.BackgroundColor3 = COLORS.SidebarBg
	ButtonCard.Parent = parent
	Instance.new("UICorner", ButtonCard).CornerRadius = UDim.new(0, 6)

	local CardStroke = Instance.new("UIStroke")
	CardStroke.Color = themeObj.Border
	CardStroke.Thickness = 1
	CardStroke.Transparency = 0.5
	CardStroke.Parent = ButtonCard

	local IndicatorBar = Instance.new("Frame")
	IndicatorBar.Size = UDim2.new(0, 4, 1, -8)
	IndicatorBar.Position = UDim2.new(0, 4, 0, 4)
	IndicatorBar.BackgroundColor3 = themeObj.Accent
	IndicatorBar.BorderSizePixel = 0
	IndicatorBar.Parent = ButtonCard
	Instance.new("UICorner", IndicatorBar).CornerRadius = UDim.new(1, 0)

	local Title = Instance.new("TextLabel")
	Title.Size = UDim2.new(1, -80, 0, 16)
	Title.Position = UDim2.new(0, 14, 0, 5)
	Title.BackgroundTransparency = 1
	Title.Text = config.name
	Title.TextColor3 = COLORS.TextWhite
	Title.TextXAlignment = Enum.TextXAlignment.Left
	Title.TextSize = 12
	Title.Font = FONTS.Title
	Title.Parent = ButtonCard

	local Desc = Instance.new("TextLabel")
	Desc.Size = UDim2.new(1, -80, 0, 14)
	Desc.Position = UDim2.new(0, 14, 0, 22)
	Desc.BackgroundTransparency = 1
	Desc.Text = config.desc
	Desc.TextColor3 = COLORS.TextDim
	Desc.TextXAlignment = Enum.TextXAlignment.Left
	Desc.TextTruncate = Enum.TextTruncate.AtEnd
	Desc.TextSize = 10
	Desc.Font = FONTS.Body
	Desc.Parent = ButtonCard

	local ApplyBtn = Instance.new("TextButton")
	ApplyBtn.Size = UDim2.new(0, 60, 0, 24)
	ApplyBtn.Position = UDim2.new(1, -66, 0.5, -12)
	ApplyBtn.BackgroundColor3 = themeObj.DarkAccent
	ApplyBtn.Text = "LOAD"
	ApplyBtn.TextColor3 = themeObj.Accent
	ApplyBtn.TextSize = 10
	ApplyBtn.Font = FONTS.Title
	ApplyBtn.AutoButtonColor = false
	ApplyBtn.Parent = ButtonCard
	Instance.new("UICorner", ApplyBtn).CornerRadius = UDim.new(0, 4)

	local BtnStroke = Instance.new("UIStroke")
	BtnStroke.Color = themeObj.Border
	BtnStroke.Thickness = 1
	BtnStroke.Transparency = 0.3
	BtnStroke.Parent = ApplyBtn

	ApplyBtn.MouseButton1Click:Connect(function()
		TweenService:Create(ApplyBtn, TweenInfo.new(0.08), {Size = UDim2.new(0, 56, 0, 22)}):Play()
		task.wait(0.08)
		TweenService:Create(ApplyBtn, TweenInfo.new(0.08), {Size = UDim2.new(0, 60, 0, 24)}):Play()
		ApplyConfig(configKey)
	end)
end

-- ==========================================
-- НАПОЛНЕНИЕ ВКЛАДОК
-- ==========================================

-- 1. Aimbot
local AimbotCard = CreateCard(Pages["Aimbot"], "Aimbot Settings")
CreateToggle(AimbotCard, "Enable Aimbot", AimbotState, function(s) AimbotState = s end)
CreateDropdown(AimbotCard, "Aim Key Trigger", {"Mouse 1 (LMB)", "Mouse 2 (RMB)"}, AimbotKeyName, function(s) AimbotKeyName = s end)
CreateToggle(AimbotCard, "Visible Only Check", AimbotVisibleCheck, function(s) AimbotVisibleCheck = s end)
CreateToggle(AimbotCard, "Enable Prediction", AimbotPrediction, function(s) AimbotPrediction = s end)
CreateSlider(AimbotCard, "Prediction Amount", 0, 50, math.floor(AimbotPredictionAmount * 1000), function(v) AimbotPredictionAmount = v / 1000 end, "ms")
CreateToggle(AimbotCard, "Distance Compensation", DistanceCompensateState, function(s) DistanceCompensateState = s end)
CreateToggle(AimbotCard, "Random Target Part", RandomHitpartState, function(s) RandomHitpartState = s end)
CreateToggle(AimbotCard, "Show FOV Circle", DrawFOVState, function(s) DrawFOVState = s end)
CreateSlider(AimbotCard, "Aimbot FOV Radius", 10, 800, AimbotFOV, function(v) AimbotFOV = v end, "px")
CreateDropdown(AimbotCard, "Target Part", {"Head", "Torso"}, AimbotTargetPart, function(s) AimbotTargetPart = s end)
CreateSlider(AimbotCard, "Aim Speed / Smoothness", 0, 100, AimbotSpeed, function(v) AimbotSpeed = v end)

local SilentAimCard = CreateCard(Pages["Aimbot"], "Silent Aim")
CreateToggle(SilentAimCard, "Enable Silent Aim", SilentAimState, function(s)
	SilentAimState = s
	if s and not ClientFireModule then SetupFireVolleyInterceptor() end
end)
CreateSlider(SilentAimCard, "Silent Aim FOV", 10, 800, SilentAimFOV, function(v) SilentAimFOV = v end, "px")
CreateDropdown(SilentAimCard, "Target Part", {"Head", "Torso"}, SilentAimTargetPart, function(s) SilentAimTargetPart = s end)
CreateToggle(SilentAimCard, "Team Check", SilentAimTeamCheck, function(s) SilentAimTeamCheck = s end)
CreateToggle(SilentAimCard, "Wall Check", SilentAimWallCheck, function(s) SilentAimWallCheck = s end)
CreateSlider(SilentAimCard, "Max Distance", 100, 5000, SilentAimMaxDistance, function(v) SilentAimMaxDistance = v end, "m")

-- 2. Weapon Mods
local RapidFireCard = CreateCard(Pages["Weapon Mods"], "Rapid Fire")
CreateToggle(RapidFireCard, "Enable Rapid Fire", RapidFireState, function(s) RapidFireState = s end)
CreateSlider(RapidFireCard, "Shot Delay", 0.01, 0.10, RapidFireDelay, function(v) RapidFireDelay = v end, "s", true)

local NoRecoilCard = CreateCard(Pages["Weapon Mods"], "No Recoil & Spread")
CreateToggle(NoRecoilCard, "Enable No Recoil", NoRecoilState, function(s) NoRecoilState = s end)
CreateToggle(NoRecoilCard, "Zero Spread", ZeroSpreadState, function(s) ZeroSpreadState = s end)
CreateToggle(NoRecoilCard, "Zero Recoil", ZeroRecoilState, function(s) ZeroRecoilState = s end)
CreateToggle(NoRecoilCard, "First Shot Recoil", ZeroFirstShotState, function(s) ZeroFirstShotState = s end)
CreateToggle(NoRecoilCard, "Visual Recoil", ZeroVisualState, function(s) ZeroVisualState = s end)

local InfAmmoCard = CreateCard(Pages["Weapon Mods"], "Infinite Ammo")
CreateToggle(InfAmmoCard, "Enable Inf Ammo", InfAmmoState, function(s) InfAmmoState = s end)

-- 3. Anti-Aim
local AACardMain = CreateCard(Pages["Anti-Aim"], "Anti-Aim Main")
CreateToggle(AACardMain, "Enable Anti-Aim", AA_Enabled, function(s) AA_Enabled = s end)
CreateSlider(AACardMain, "Pitch (Up / Down)", -89, 89, AA_Pitch, function(v) AA_Pitch = v end)
CreateSlider(AACardMain, "Yaw Offset", -180, 180, AA_YawOffset, function(v) AA_YawOffset = v end)
CreateToggle(AACardMain, "Snapback After Shot", AA_Snapback, function(s) AA_Snapback = s end)

local AACardShotSync = CreateCard(Pages["Anti-Aim"], "Shot Sync (Silent Aim)")
CreateToggle(AACardShotSync, "Enable Shot Sync", AA_ShotSync, function(s) AA_ShotSync = s end)
CreateSlider(AACardShotSync, "Sync Duration", 0.05, 0.5, AA_ShotSyncDuration, function(v) AA_ShotSyncDuration = v end, "s", true)

local AACardSpin = CreateCard(Pages["Anti-Aim"], "Spinbot")
CreateToggle(AACardSpin, "Enable Spinbot", AA_Spin, function(s) AA_Spin = s end)
CreateSlider(AACardSpin, "Spin Speed", 1, 50, AA_SpinSpeed, function(v) AA_SpinSpeed = v end)

local AACardJitter = CreateCard(Pages["Anti-Aim"], "Jitter")
CreateToggle(AACardJitter, "Enable Jitter", AA_Jitter, function(s) AA_Jitter = s end)
CreateSlider(AACardJitter, "Jitter Angle", 0, 180, AA_JitterAngle, function(v) AA_JitterAngle = v end)
CreateSlider(AACardJitter, "Jitter Delay", 10, 1000, math.floor(AA_JitterDelay * 1000), function(v) AA_JitterDelay = v / 1000 end, "ms")

-- 4. Visuals
local VisualsCard = CreateCard(Pages["Visuals"], "Player Visuals")
CreateToggle(VisualsCard, "Dynamic ESP Glow", GlowState, function(s) ToggleTeamGlow(s) end)
CreateToggle(VisualsCard, "Friend Hat (Halo)", FriendHatState, function(s) ToggleFriendHat(s) end)
CreateToggle(VisualsCard, "Friend ESP (Green Name)", FriendESPState, function(s) ToggleFriendESP(s) end)

local TracersCard = CreateCard(Pages["Visuals"], "Bullet Tracers")
CreateToggle(TracersCard, "Enable Bullet Tracers", BulletTracersState, function(s) BulletTracersState = s end)
CreateSlider(TracersCard, "Tracer LifeTime", 1, 10, BulletTracersLifeTime, function(v) BulletTracersLifeTime = v end, "s", true)

-- 5. Graphics
local WorldGraphicsCard = CreateCard(Pages["Graphics"], "World & Lighting Enhancements")
CreateSlider(WorldGraphicsCard, "World Saturation", 0, 50, 3, function(v) GraphicsEffects.ColorCorrection.Saturation = v end, "x")

-- 6. Optimization
local OptCard = CreateCard(Pages["Optimization"], "Performance & FPS Boost")
CreateToggle(OptCard, "Potato Graphics", PotatoState, function(s) SetPotatoGraphics(s) end)
CreateToggle(OptCard, "No Fog", NoFogState, function(s) ToggleNoFog(s) end)
CreateToggle(OptCard, "Remove Muzzle Flash", RemoveMuzzleState, function(s) ToggleRemoveMuzzle(s) end)

-- 7. Venchine mods
local VenchineCard = CreateCard(Pages["Venchine mods"], "Movement & Vehicle Mods")
CreateToggle(VenchineCard, "Enable Speed+", SpeedPlusState, function(s) SpeedPlusState = s end)
CreateSlider(VenchineCard, "Boost Force", 0.1, 2.0, SpeedPlusForce, function(v) SpeedPlusForce = v end, "", true)

-- 8. Misc
local LightingCard = CreateCard(Pages["Misc"], "Lighting Settings")
CreateToggle(LightingCard, "Enable FullBright", FullBrightState, function(s)
	FullBrightState = s
	if s then ApplyFullBright() else RestoreLighting() end
end)

local ScriptManagementCard = CreateCard(Pages["Misc"], "Script Management")
CreateButton(ScriptManagementCard, "Unload Script", function() Unload() end)

-- 9. Settings
local ConfigsCard = CreateCard(Pages["Settings"], "Preset Configurations")

CreateConfigCardButton(ConfigsCard, "Legit")
CreateConfigCardButton(ConfigsCard, "Medium")
CreateConfigCardButton(ConfigsCard, "SemiRage")
CreateConfigCardButton(ConfigsCard, "Rage")

local SettingsCard = CreateCard(Pages["Settings"], "UI Color Theme")
CreateDropdown(SettingsCard, "UI Color Theme", {"Birch", "Yellow", "Blue", "White", "Red", "Cyan", "Purple", "Lime", "Black"}, CurrentThemeName, function(selectedTheme)
	ApplyTheme(selectedTheme)
end)

-- ==========================================
-- УПРАВЛЕНИЕ ОКНОМ И СТАРТ
-- ==========================================
local guiVisible = false
local isTweening = false

local function ToggleGUI()
	if isTweening or Unloaded then return end
	isTweening = true
	PlaySound(SOUNDS.OpenClose, 1.0)
	CloseActiveDropdown()
	
	if guiVisible then
		local tween = TweenService:Create(UIScale, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Scale = 0})
		tween:Play()
		tween.Completed:Wait()
		MainFrame.Visible = false
		guiVisible = false
	else
		MainFrame.Visible = true
		local tween = TweenService:Create(UIScale, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1})
		tween:Play()
		tween.Completed:Wait()
		guiVisible = true
	end
	isTweening = false
end

table.insert(Connections, UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Enum.KeyCode.RightShift then
		ToggleGUI()
	end
end))

-- Запуск интерфейса
MainFrame.Visible = true
guiVisible = true
PlaySound(SOUNDS.OpenClose, 1.0)
TweenService:Create(UIScale, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
SwitchTab("Settings")

-- Применяем Legit-конфиг по умолчанию
ApplyConfig("Legit")
