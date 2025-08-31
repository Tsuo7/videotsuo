--[[
TSUO Nebula++ UI Library (ModuleScript)
Author: ChatGPT (for Kayo / TSUO)
License: You may use/modify inside your own Roblox experiences.

⚠️ IMPORTANT:
- Esta lib é focada em UI/UX e arquitetura. NÃO implementa aimbot/ESP/wallhack.
- Todos os recursos “HVH” devem ser alimentados **pelo seu servidor**. Aqui há apenas slots/PLACEHOLDERS.
- Visuals/Targeting expõem funções que recebem listas de alvos válidos (Instances) enviadas pelo servidor.

Entregáveis neste arquivo:
1) ModuleScript: NebulaUI (toda a lib)
2) Ao final do arquivo: exemplo de LocalScript (NebulaUI_Demo) — COPIE para outro script.

]]

-- =============================================================
-- =                        NebulaUI                          =
-- =============================================================
local NebulaUI = {}
NebulaUI.__index = NebulaUI

-- Services
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

local LOCAL_PLAYER = Players.LocalPlayer
local MOUSE = LOCAL_PLAYER and LOCAL_PLAYER:GetMouse() or nil

-- Utils -------------------------------------------------------
local function new(className, props, children)
	local inst = Instance.new(className)
	if props then
		for k, v in pairs(props) do
			inst[k] = v
		end
	end
	if children then
		for _, child in ipairs(children) do
			child.Parent = inst
		end
	end
	return inst
end

local function tween(o, info, goals)
	local ti = typeof(info) == "TweenInfo" and info or TweenInfo.new(info or 0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	return TweenService:Create(o, ti, goals)
end

local function clamp(n, a, b)
	return math.clamp(n, a, b)
end

-- Theme Engine -----------------------------------------------
local DEFAULT_THEME = {
	Name = "NebulaDark",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor = Color3.fromRGB(235, 235, 240),
	TextMuted = Color3.fromRGB(180, 180, 190),
	Bg = Color3.fromRGB(16, 16, 20),
	Bg2 = Color3.fromRGB(22, 22, 28),
	Panel = Color3.fromRGB(28, 28, 36),
	Stroke = Color3.fromRGB(60, 60, 72),
	Accent = Color3.fromRGB(126, 144, 255),
	Accent2 = Color3.fromRGB(88, 204, 237),
	Shadow = 0.25,
	Glass = 0.1, -- 0 (opaco) .. 0.6 (bem translúcido)
}

local THEME_PRESETS = {
	NebulaDark = DEFAULT_THEME,
	NebulaLight = {
		Name = "NebulaLight",
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor = Color3.fromRGB(28, 28, 36),
		TextMuted = Color3.fromRGB(90, 90, 100),
		Bg = Color3.fromRGB(242, 244, 248),
		Bg2 = Color3.fromRGB(230, 233, 238),
		Panel = Color3.fromRGB(255, 255, 255),
		Stroke = Color3.fromRGB(210, 214, 222),
		Accent = Color3.fromRGB(126, 144, 255),
		Accent2 = Color3.fromRGB(255, 104, 180),
		Shadow = 0.18,
		Glass = 0.05,
	},
	Matrix = {
		Name = "Matrix",
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor = Color3.fromRGB(210, 255, 210),
		TextMuted = Color3.fromRGB(140, 180, 140),
		Bg = Color3.fromRGB(6, 10, 8),
		Bg2 = Color3.fromRGB(10, 16, 12),
		Panel = Color3.fromRGB(14, 20, 16),
		Stroke = Color3.fromRGB(28, 44, 32),
		Accent = Color3.fromRGB(90, 255, 140),
		Accent2 = Color3.fromRGB(60, 180, 120),
		Shadow = 0.28,
		Glass = 0.08,
	},
	Sakura = {
		Name = "Sakura",
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor = Color3.fromRGB(40, 30, 36),
		TextMuted = Color3.fromRGB(110, 90, 100),
		Bg = Color3.fromRGB(250, 245, 248),
		Bg2 = Color3.fromRGB(244, 236, 242),
		Panel = Color3.fromRGB(255, 252, 254),
		Stroke = Color3.fromRGB(230, 216, 228),
		Accent = Color3.fromRGB(255, 120, 180),
		Accent2 = Color3.fromRGB(150, 110, 240),
		Shadow = 0.16,
		Glass = 0.04,
	},
	Aqua = {
		Name = "Aqua",
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor = Color3.fromRGB(235, 245, 255),
		TextMuted = Color3.fromRGB(170, 200, 220),
		Bg = Color3.fromRGB(10, 14, 24),
		Bg2 = Color3.fromRGB(14, 18, 28),
		Panel = Color3.fromRGB(18, 24, 38),
		Stroke = Color3.fromRGB(46, 64, 92),
		Accent = Color3.fromRGB(72, 160, 255),
		Accent2 = Color3.fromRGB(88, 204, 237),
		Shadow = 0.22,
		Glass = 0.10,
	},
}

local function deepCopy(tbl)
	local r = {}
	for k,v in pairs(tbl) do
		r[k] = (typeof(v) == "table") and deepCopy(v) or v
	end
	return r
end

-- Storage (sessão)
local SESSION = {
	config = {},
	connections = {},
}

-- Component Factory ------------------------------------------
local Component = {}
Component.__index = Component

function Component:Destroy()
	if self.Instance and self.Instance.Parent then self.Instance:Destroy() end
	for _, c in ipairs(self._conns or {}) do
		if c.Disconnect then c:Disconnect() end
	end
end

local function applyStroke(frame, color)
	local s = new("UIStroke", {Parent = frame, Color = color or Color3.fromRGB(60,60,72), Thickness = 1})
	return s
end

local function roundify(frame, r)
	new("UICorner", {Parent = frame, CornerRadius = UDim.new(0, r or 10)})
end

-- Main Window -------------------------------------------------
local Window = {}
Window.__index = Window

function NebulaUI.new(opts)
	opts = opts or {}
	local self = setmetatable({}, NebulaUI)
	self.Name = opts.Name or "TSUO Nebula++"
	self.IconAssetId = tostring(opts.IconAssetId or "rbxassetid://0")
	self.Theme = deepCopy(THEME_PRESETS[opts.ThemePreset or "NebulaDark"] or DEFAULT_THEME)
	self.Theme.Glass = clamp(opts.Glass or self.Theme.Glass, 0, 0.6)
	self.Root = new("ScreenGui", {
		Name = self.Name:gsub("%s","_") .. "_GUI",
		ResetOnSpawn = false,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
		IgnoreGuiInset = true,
	t})
	self.Root.Parent = LOCAL_PLAYER:WaitForChild("PlayerGui")

	-- Blur opcional (leve)
	local blur = Instance.new("BlurEffect")
	blur.Size = 0
	blur.Parent = workspace.CurrentCamera
	self._blur = blur

	-- Notifications stack
	local notifyRoot = new("Frame", {
		Name = "Notifications",
		Parent = self.Root,
		AnchorPoint = Vector2.new(1,1),
		Position = UDim2.fromScale(0.98, 0.98),
		Size = UDim2.fromOffset(300, 300),
		BackgroundTransparency = 1,
	})
	local nList = new("UIListLayout", {Parent = notifyRoot, Padding = UDim.new(0,8), HorizontalAlignment = Enum.HorizontalAlignment.Right, VerticalAlignment = Enum.VerticalAlignment.Bottom})
	self.NotifyRoot = notifyRoot

	-- Watermark
	local watermark = new("TextLabel", {
		Name = "Watermark",
		Parent = self.Root,
		Text = self.Name .. "  |  FPS: --  Ping: --",
		TextSize = self.Theme.TextSize,
		Font = self.Theme.Font,
		TextColor3 = self.Theme.TextMuted,
		BackgroundColor3 = self.Theme.Panel,
		BackgroundTransparency = 0.3 + self.Theme.Glass,
		Position = UDim2.fromOffset(16, 16),
		AutomaticSize = Enum.AutomaticSize.XY,
		BorderSizePixel = 0,
		ZIndex = 30,
		Padding = Vector2.new(10,4),
	})
	roundify(watermark, 8)
	applyStroke(watermark, self.Theme.Stroke)
	new("UIPadding", {Parent = watermark, PaddingLeft = UDim.new(0,8), PaddingRight = UDim.new(0,8), PaddingTop = UDim.new(0,4), PaddingBottom = UDim.new(0,4)})
	self._watermark = watermark

	-- Main Window
	local win = new("Frame", {
		Name = "Window",
		Parent = self.Root,
		Size = UDim2.fromOffset(640, 420),
		Position = UDim2.fromScale(0.5, 0.5),
		AnchorPoint = Vector2.new(0.5, 0.5),
		BackgroundColor3 = self.Theme.Panel,
		BackgroundTransparency = 0.15 + self.Theme.Glass,
	})
	roundify(win, 14)
	applyStroke(win, self.Theme.Stroke)
	self.Window = win

	-- Shadow (fake)
	local shadow = new("ImageLabel", {
		Name = "Shadow",
		Parent = win,
		ZIndex = 0,
		Image = "rbxassetid://5028857084",
		ImageColor3 = Color3.new(0,0,0),
		ImageTransparency = 1 - self.Theme.Shadow,
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(24,24,276,276),
		BackgroundTransparency = 1,
		AnchorPoint = Vector2.new(0.5,0.5),
		Position = UDim2.fromScale(0.5,0.5),
		Size = UDim2.fromScale(1.2, 1.2),
	})

	-- Titlebar
	local bar = new("Frame", {
		Name = "TitleBar",
		Parent = win,
		Size = UDim2.new(1,0,0,40),
		BackgroundColor3 = self.Theme.Bg2,
		BackgroundTransparency = 0.1 + self.Theme.Glass,
		BorderSizePixel = 0,
	})
	roundify(bar, 14)
	applyStroke(bar, self.Theme.Stroke)

	local icon = new("ImageLabel", {
		Name = "Icon",
		Parent = bar,
		Size = UDim2.fromOffset(24,24),
		Position = UDim2.fromOffset(12,8),
		BackgroundTransparency = 1,
		Image = self.IconAssetId,
	})

	local title = new("TextLabel", {
		Name = "Title",
		Parent = bar,
		Text = self.Name,
		Font = self.Theme.Font,
		TextSize = 16,
		TextColor3 = self.Theme.TextColor,
		BackgroundTransparency = 1,
		Position = UDim2.fromOffset(44, 0),
		Size = UDim2.new(1, -160, 1, 0),
		TextXAlignment = Enum.TextXAlignment.Left,
	})

	-- Controls (minimize, maximize, close)
	local btns = new("Frame", {Parent = bar, BackgroundTransparency = 1, AnchorPoint = Vector2.new(1,0), Position = UDim2.new(1,-8,0,4), Size = UDim2.fromOffset(120, 32)})
	local list = new("UIListLayout", {Parent = btns, FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0,6), HorizontalAlignment = Enum.HorizontalAlignment.Right, VerticalAlignment = Enum.VerticalAlignment.Center})

	local function makeBarButton(name, txt)
		local b = new("TextButton", {
			Name = name,
			Parent = btns,
			Text = txt,
			Font = self.Theme.Font,
			TextSize = 14,
			TextColor3 = self.Theme.TextColor,
			BackgroundColor3 = self.Theme.Panel,
			BackgroundTransparency = 0.15 + self.Theme.Glass,
			Size = UDim2.fromOffset(32, 32),
			AutoButtonColor = false,
			BorderSizePixel = 0,
		})
		roundify(b, 8)
		applyStroke(b, self.Theme.Stroke)
		b.MouseEnter:Connect(function() tween(b, 0.12, {BackgroundTransparency = 0.05 + self.Theme.Glass}):Play() end)
		b.MouseLeave:Connect(function() tween(b, 0.12, {BackgroundTransparency = 0.15 + self.Theme.Glass}):Play() end)
		return b
	end

	local minimize = makeBarButton("Minimize", "–")
	local maximize = makeBarButton("Maximize", "□")
	local close = makeBarButton("Close", "×")

	-- Dock icon (when minimized)
	local dock = new("ImageButton", {
		Name = "Dock",
		Parent = self.Root,
		BackgroundTransparency = 1,
		Visible = false,
		Image = self.IconAssetId,
		Size = UDim2.fromOffset(48, 48),
		Position = UDim2.new(0, 18, 1, -66),
		ZIndex = 50,
	})
	self._dock = dock

	local content = new("Frame", {
		Name = "Content",
		Parent = win,
		BackgroundTransparency = 1,
		Position = UDim2.fromOffset(12, 52),
		Size = UDim2.new(1, -24, 1, -64),
	})

	-- Tabs header
	local tabsHeader = new("Frame", {Parent = content, BackgroundTransparency = 1, Size = UDim2.new(1,0,0,36)})
	local thList = new("UIListLayout", {Parent = tabsHeader, FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0,8), HorizontalAlignment = Enum.HorizontalAlignment.Left, VerticalAlignment = Enum.VerticalAlignment.Center})

	-- Tab content area (scrolling)
	local tabArea = new("Frame", {Parent = content, BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 40), Size = UDim2.new(1,0,1,-40)})

	self._tabsHeader = tabsHeader
	self._tabArea = tabArea
	self._tabs = {}
	self._activeTab = nil

	-- Drag logic
	local dragging = false
	local dragStart, startPos
	bar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = win.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = (input.Position - dragStart)
			local newPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
			-- clamp to screen
			local vp = workspace.CurrentCamera.ViewportSize
			local w, h = win.AbsoluteSize.X, win.AbsoluteSize.Y
			newPos = UDim2.new(0, clamp(newPos.X.Offset, 8 - w, vp.X - 8), 0, clamp(newPos.Y.Offset, 8 - h, vp.Y - 8))
			win.Position = newPos
		end
	end)

	-- Buttons behav
	minimize.MouseButton1Click:Connect(function()
		win.Visible = false
		dock.Visible = true
		self:Notify("Minimizado", "Clique no ícone para restaurar", 2.5)
	end)
	maximize.MouseButton1Click:Connect(function()
		local full = win.Size == UDim2.fromScale(0.94, 0.86)
		if full then
			win.Size = UDim2.fromOffset(640, 420)
			win.Position = UDim2.fromScale(0.5, 0.5)
		else
			win.Size = UDim2.fromScale(0.94, 0.86)
			win.Position = UDim2.fromScale(0.5, 0.5)
		end
	end)
	close.MouseButton1Click:Connect(function()
		self.Root:Destroy()
		if self._blur then self._blur:Destroy() end
	end)
	dock.MouseButton1Click:Connect(function()
		win.Visible = true
		dock.Visible = false
	end)

	-- Splash
	self:_showSplash()
	self:Notify("Bem-vindo ao " .. self.Name, "discord.gg/tsuo", 4)

	-- FPS/Ping display (placeholder)
	local last = tick()
	RunService.RenderStepped:Connect(function()
		local now = tick()
		local fps = math.floor(1 / math.max(1/60, now - last))
		last = now
		self._watermark.Text = ("%s  |  FPS: %d  Ping: --"):format(self.Name, fps)
	end)

	return self
end

function NebulaUI:_applyThemeToDescendants()
	local t = self.Theme
	for _, inst in ipairs(self.Root:GetDescendants()) do
		if inst:IsA("TextLabel") or inst:IsA("TextButton") then
			inst.Font = t.Font
			inst.TextColor3 = inst.Name == "Muted" and t.TextMuted or t.TextColor
		end
		if inst:IsA("Frame") or inst:IsA("TextButton") then
			if inst:GetAttribute("isPanel") then
				inst.BackgroundColor3 = t.Panel
				inst.BackgroundTransparency = 0.15 + t.Glass
			end
		end
		if inst:IsA("UIStroke") then
			inst.Color = t.Stroke
		end
	end
end

function NebulaUI:SetThemeByName(name)
	local preset = THEME_PRESETS[name]
	if not preset then return false end
	self.Theme = deepCopy(preset)
	self:_applyThemeToDescendants()
	return true
end

function NebulaUI:RegisterTheme(name, themeTable)
	THEME_PRESETS[name] = themeTable
end

function NebulaUI:Notify(title, message, timeSec)
	timeSec = timeSec or 2
	local card = new("Frame", {Parent = self.NotifyRoot, BackgroundColor3 = self.Theme.Panel, BackgroundTransparency = 0.1 + self.Theme.Glass, Size = UDim2.fromOffset(260, 64)})
	roundify(card, 10)
	applyStroke(card, self.Theme.Stroke)
	new("UIPadding", {Parent = card, PaddingLeft = UDim.new(0,10), PaddingTop = UDim.new(0,8), PaddingRight = UDim.new(0,10), PaddingBottom = UDim.new(0,8)})
	local tl = new("TextLabel", {Parent = card, Text = title, Font = self.Theme.Font, TextSize = 14, TextColor3 = self.Theme.TextColor, BackgroundTransparency = 1, Position = UDim2.fromOffset(0,0), Size = UDim2.new(1,0,0,20), TextXAlignment = Enum.TextXAlignment.Left})
	local msg = new("TextLabel", {Parent = card, Text = message, Font = self.Theme.Font, TextSize = 13, TextColor3 = self.Theme.TextMuted, BackgroundTransparency = 1, Position = UDim2.fromOffset(0,22), Size = UDim2.new(1,0,1,-22), TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true})
	tween(card, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.fromOffset(280, 72)}):Play()
	task.delay(timeSec, function()
		if card and card.Parent then
			tween(card, 0.15, {BackgroundTransparency = 1}):Play()
			task.wait(0.15)
			card:Destroy()
		end
	end)
end

function NebulaUI:_showSplash()
	local overlay = new("Frame", {Parent = self.Root, BackgroundColor3 = self.Theme.Bg, BackgroundTransparency = 0.2 + self.Theme.Glass, Size = UDim2.fromScale(1,1), ZIndex = 100})
	local label = new("TextLabel", {Parent = overlay, BackgroundTransparency = 1, Text = "Carregando...", Font = self.Theme.Font, TextSize = 20, TextColor3 = self.Theme.TextColor, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.46)})
	local counter = new("TextLabel", {Parent = overlay, BackgroundTransparency = 1, Text = "5", Font = self.Theme.Font, TextSize = 36, TextColor3 = self.Theme.Accent, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.56)})
	for i=5,1,-1 do
		counter.Text = tostring(i)
		tween(counter, 0.3, {TextTransparency = 0}):Play()
		task.wait(0.7)
		counter.TextTransparency = 0.3
	end
	overlay:Destroy()
	if self._blur then tween(self._blur, 0.3, {Size = 4}):Play(); task.delay(0.3, function() tween(self._blur, 0.4, {Size = 0}):Play() end) end
end

-- Tabs --------------------------------------------------------
local Tab = {}
Tab.__index = Tab

function Window:AddTab(name, emojiOrIcon)
	local headerBtn = new("TextButton", {Parent = self._tabsHeader, Text = (emojiOrIcon and (tostring(emojiOrIcon) .. " ") or "") .. name, Font = self.Theme.Font, TextSize = 14, TextColor3 = self.Theme.TextColor, BackgroundColor3 = self.Theme.Panel, BackgroundTransparency = 0.15 + self.Theme.Glass, AutoButtonColor = false, Size = UDim2.fromOffset(120, 36), BorderSizePixel = 0})
	roundify(headerBtn, 10)
	applyStroke(headerBtn, self.Theme.Stroke)
	headerBtn.MouseEnter:Connect(function() tween(headerBtn, 0.1, {BackgroundTransparency = 0.06 + self.Theme.Glass}):Play() end)
	headerBtn.MouseLeave:Connect(function() tween(headerBtn, 0.1, {BackgroundTransparency = 0.15 + self.Theme.Glass}):Play() end)

	local scroll = new("ScrollingFrame", {Parent = self._tabArea, Active = true, ScrollingDirection = Enum.ScrollingDirection.Y, CanvasSize = UDim2.fromOffset(0,0), Size = UDim2.fromScale(1,1), BackgroundTransparency = 1, Visible = false, BorderSizePixel = 0})
	local list = new("UIListLayout", {Parent = scroll, Padding = UDim.new(0,8), HorizontalAlignment = Enum.HorizontalAlignment.Left, SortOrder = Enum.SortOrder.LayoutOrder})
	new("UIPadding", {Parent = scroll, PaddingLeft = UDim.new(0,8), PaddingRight = UDim.new(0,8), PaddingTop = UDim.new(0,8), PaddingBottom = UDim.new(0,8)})

	local tabObj = setmetatable({
		Name = name,
		Header = headerBtn,
		Scroll = scroll,
		Components = {},
	}, Tab)

	headerBtn.MouseButton1Click:Connect(function() self:SelectTab(tabObj) end)
	self._tabs[#self._tabs+1] = tabObj
	if not self._activeTab then self:SelectTab(tabObj) end
	return tabObj
end

function Window:SelectTab(tab)
	for _, t in ipairs(self._tabs) do
		t.Scroll.Visible = false
		t.Header.TextColor3 = self.Theme.TextMuted
	end
	tab.Scroll.Visible = true
	tab.Header.TextColor3 = self.Theme.TextColor
end

-- Sections & Components --------------------------------------
function Tab:AddSection(title)
	local section = new("Frame", {Parent = self.Scroll, BackgroundColor3 = self.Parent.Theme.Panel, BackgroundTransparency = 0.1 + self.Parent.Theme.Glass, Size = UDim2.new(1, -0, 0, 48)})
	section:SetAttribute("isPanel", true)
	roundify(section, 12)
	applyStroke(section, self.Parent.Theme.Stroke)
	new("UIPadding", {Parent = section, PaddingLeft = UDim.new(0,12), PaddingRight = UDim.new(0,12), PaddingTop = UDim.new(0,10), PaddingBottom = UDim.new(0,10)})
	local lbl = new("TextLabel", {Parent = section, Text = title, BackgroundTransparency = 1, Font = self.Parent.Theme.Font, TextSize = 15, TextColor3 = self.Parent.Theme.TextColor, Size = UDim2.new(1,0,1,0), TextXAlignment = Enum.TextXAlignment.Left})
	section.AutomaticSize = Enum.AutomaticSize.Y
	local inner = new("Frame", {Parent = section, BackgroundTransparency = 1, Position = UDim2.fromOffset(0,28), Size = UDim2.new(1,0,0,0), Name = "Inner"})
	local list = new("UIListLayout", {Parent = inner, Padding = UDim.new(0,8)})
	return inner
end

local function makeControlBase(parent, ui, labelText)
	local frame = new("Frame", {Parent = parent, BackgroundColor3 = ui.Theme.Bg2, BackgroundTransparency = 0.1 + ui.Theme.Glass, Size = UDim2.new(1,0,0,36)})
	frame:SetAttribute("isPanel", true)
	roundify(frame, 10)
	applyStroke(frame, ui.Theme.Stroke)
	local label = new("TextLabel", {Parent = frame, BackgroundTransparency = 1, Text = labelText, Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, Position = UDim2.fromOffset(12,0), Size = UDim2.new(1,-140,1,0), TextXAlignment = Enum.TextXAlignment.Left})
	return frame, label
end

function Tab:AddButton(text, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	frame.Size = UDim2.new(1,0,0,36)
	local btn = new("TextButton", {Parent = frame, Text = "Run", Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(80,28), AutoButtonColor = false, BorderSizePixel = 0})
	roundify(btn, 8)
	applyStroke(btn, ui.Theme.Stroke)
	btn.MouseButton1Click:Connect(function()
		if callback then
			local ok, err = pcall(callback)
			if not ok then ui:Notify("Erro", tostring(err), 3) end
		end
	end)
	return btn
end

function Tab:AddToggle(text, default, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local knob = new("TextButton", {Parent = frame, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AutoButtonColor = false, BorderSizePixel = 0, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(48,24), Text = ""})
	roundify(knob, 12)
	applyStroke(knob, ui.Theme.Stroke)
	local dot = new("Frame", {Parent = knob, BackgroundColor3 = ui.Theme.Stroke, Size = UDim2.fromOffset(18,18), Position = UDim2.fromOffset(4,3)})
	roundify(dot, 9)
	local state = default and true or false
	local function render()
		if state then
			knob.BackgroundColor3 = ui.Theme.Accent
			tween(dot, 0.12, {Position = UDim2.fromOffset(26,3)}):Play()
		else
			knob.BackgroundColor3 = ui.Theme.Panel
			tween(dot, 0.12, {Position = UDim2.fromOffset(4,3)}):Play()
		end
	end
	render()
	knob.MouseButton1Click:Connect(function()
		state = not state
		render()
		if callback then callback(state) end
	end)
	return setmetatable({Instance = frame, Get = function() return state end}, Component)
end

function Tab:AddSlider(text, min, max, default, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local bar = new("Frame", {Parent = frame, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-12,0.5,0), Size = UDim2.fromOffset(220,8), BorderSizePixel = 0})
	roundify(bar, 4)
	local fill = new("Frame", {Parent = bar, BackgroundColor3 = ui.Theme.Accent, Size = UDim2.fromScale(0,1), BorderSizePixel = 0})
	roundify(fill, 4)
	local value = default or min
	local dragging
	local function setFromX(x)
		local alpha = clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
		value = math.floor(min + (max-min)*alpha + 0.5)
		fill.Size = UDim2.fromScale(alpha, 1)
		label.Text = ("%s  (%d)"):format(text, value)
		if callback then callback(value) end
	end
	bar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true setFromX(input.Position.X) end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then setFromX(input.Position.X) end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
	end)
	setFromX(bar.AbsolutePosition.X + bar.AbsoluteSize.X * ((default or min)-min)/(max-min))
	return setmetatable({Instance = frame, Get = function() return value end}, Component)
end

function Tab:AddDropdown(text, options, defaultIndex, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local current = defaultIndex or 1
	local btn = new("TextButton", {Parent = frame, Text = options[current] or "--", Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(160,28), AutoButtonColor = false, BorderSizePixel = 0})
	roundify(btn, 8)
	applyStroke(btn, ui.Theme.Stroke)
	local open = false
	local menu
	local function setIndex(i)
		current = i
		btn.Text = options[current]
		if callback then callback(options[current], current) end
	end
	btn.MouseButton1Click:Connect(function()
		open = not open
		if open then
			menu = new("Frame", {Parent = frame, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.06 + ui.Theme.Glass, Size = UDim2.fromOffset(160, math.min(6, #options)*28 + 8), AnchorPoint = Vector2.new(1,0), Position = UDim2.new(1,-8,1,6)})
			roundify(menu, 8); applyStroke(menu, ui.Theme.Stroke)
			new("UIPadding", {Parent = menu, PaddingTop = UDim.new(0,4), PaddingBottom = UDim.new(0,4)})
			local ml = new("UIListLayout", {Parent = menu, Padding = UDim.new(0,4)})
			for i, opt in ipairs(options) do
				local item = new("TextButton", {Parent = menu, Text = opt, Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundTransparency = 1, Size = UDim2.new(1,0,0,24)})
				item.MouseButton1Click:Connect(function() setIndex(i); open=false; menu:Destroy() end)
			end
		else
			if menu then menu:Destroy() end
		end
	end)
	setIndex(current)
	return setmetatable({Instance = frame, Get = function() return options[current] end}, Component)
end

function Tab:AddTextbox(text, placeholder, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local box = new("TextBox", {Parent = frame, PlaceholderText = placeholder or "", Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(200,28), BorderSizePixel = 0})
	roundify(box, 8)
	applyStroke(box, ui.Theme.Stroke)
	box.FocusLost:Connect(function(enter)
		if callback then callback(box.Text) end
	end)
	return setmetatable({Instance = frame, Get = function() return box.Text end, Set = function(_,v) box.Text=v end}, Component)
end

function Tab:AddKeybind(text, defaultKeyCode, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local btn = new("TextButton", {Parent = frame, Text = defaultKeyCode and defaultKeyCode.Name or "[None]", Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(120,28), AutoButtonColor = false, BorderSizePixel = 0})
	roundify(btn, 8)
	applyStroke(btn, ui.Theme.Stroke)
	local capture = false
	local keyCode = defaultKeyCode
	btn.MouseButton1Click:Connect(function()
		capture = true
		btn.Text = "[Pressione...]"
	end)
	UserInputService.InputBegan:Connect(function(input, gp)
		if capture and input.UserInputType == Enum.UserInputType.Keyboard then
			capture = false
			keyCode = input.KeyCode
			btn.Text = "[" .. keyCode.Name .. "]"
			if callback then callback(keyCode) end
		end
	end)
	return setmetatable({Instance = frame, Get = function() return keyCode end}, Component)
end

-- Simplified ColorPicker (HSV sliders)
function Tab:AddColorPicker(text, defaultColor, callback)
	local ui = self.Parent
	local frame, label = makeControlBase(self:AddSection(""), ui, text)
	local preview = new("Frame", {Parent = frame, BackgroundColor3 = defaultColor or ui.Theme.Accent, AnchorPoint = Vector2.new(1,0.5), Position = UDim2.new(1,-8,0.5,0), Size = UDim2.fromOffset(40,28), BorderSizePixel = 0})
	roundify(preview, 8)
	applyStroke(preview, ui.Theme.Stroke)
	local btn = new("TextButton", {Parent = frame, BackgroundTransparency = 1, Text = "", Size = UDim2.new(1,0,1,0)})
	btn.ZIndex = 2
	btn.MouseButton1Click:Connect(function()
		-- modal com 3 sliders HSV
		local modal = new("Frame", {Parent = ui.Root, BackgroundColor3 = ui.Theme.Bg, BackgroundTransparency = 0.2 + ui.Theme.Glass, Size = UDim2.fromScale(1,1), ZIndex = 200})
		local panel = new("Frame", {Parent = modal, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.1 + ui.Theme.Glass, Size = UDim2.fromOffset(320, 200), AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5)})
		roundify(panel, 12); applyStroke(panel, ui.Theme.Stroke)
		local function slider(lbl, y, onChange)
			local lb = new("TextLabel", {Parent = panel, BackgroundTransparency = 1, Text = lbl, Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, Position = UDim2.fromOffset(16,y), Size = UDim2.fromOffset(280, 18), TextXAlignment = Enum.TextXAlignment.Left})
			local bar = new("Frame", {Parent = panel, BackgroundColor3 = ui.Theme.Bg2, BackgroundTransparency = 0.2 + ui.Theme.Glass, Position = UDim2.fromOffset(16,y+22), Size = UDim2.fromOffset(288, 8)})
			roundify(bar, 4)
			local fill = new("Frame", {Parent = bar, BackgroundColor3 = ui.Theme.Accent, Size = UDim2.fromScale(0,1)})
			roundify(fill, 4)
			local dragging
			local function setFromX(x)
				local alpha = clamp((x - bar.AbsolutePosition.X)/bar.AbsoluteSize.X, 0,1)
				fill.Size = UDim2.fromScale(alpha, 1)
				onChange(alpha)
			end
			bar.InputBegan:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true setFromX(input.Position.X) end
			end)
			UserInputService.InputChanged:Connect(function(input)
				if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then setFromX(input.Position.X) end
			end)
			UserInputService.InputEnded:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
			end)
			return fill
		end
		local h,s,v = 0,1,1
		slider("Hue", 16, function(a) h=a; preview.BackgroundColor3 = Color3.fromHSV(h,s,v); if callback then callback(preview.BackgroundColor3) end end)
		slider("Saturation", 72, function(a) s=a; preview.BackgroundColor3 = Color3.fromHSV(h,s,v); if callback then callback(preview.BackgroundColor3) end end)
		slider("Value", 128, function(a) v=a; preview.BackgroundColor3 = Color3.fromHSV(h,s,v); if callback then callback(preview.BackgroundColor3) end end)
		local close = new("TextButton", {Parent = panel, Text = "Fechar", Font = ui.Theme.Font, TextSize = 14, TextColor3 = ui.Theme.TextColor, BackgroundColor3 = ui.Theme.Panel, BackgroundTransparency = 0.2 + ui.Theme.Glass, AnchorPoint = Vector2.new(1,1), Position = UDim2.fromOffset(304,184), Size = UDim2.fromOffset(72,28), AutoButtonColor = false, BorderSizePixel = 0})
		roundify(close, 8); applyStroke(close, ui.Theme.Stroke)
		close.MouseButton1Click:Connect(function() modal:Destroy() end)
	end)
	return setmetatable({Instance = frame, Get = function() return preview.BackgroundColor3 end}, Component)
end

-- Public API --------------------------------------------------
function NebulaUI:Window() return setmetatable(self, Window) end
function Window:GetRoot() return self.Root end

-- Config save/load (sessão). Para persistência verdadeira, conecte seus próprios métodos via callbacks.
function NebulaUI:SaveConfig(name)
	name = name or "default"
	SESSION.config[name] = SESSION.config[name] or {}
	-- aqui você pode copiar estados dos componentes, se desejar controlar via API
	self:Notify("Config", "Salvo (sessão) como '" .. name .. "'", 2)
end
function NebulaUI:LoadConfig(name)
	name = name or "default"
	if SESSION.config[name] then
		self:Notify("Config", "Carregado (sessão) '" .. name .. "'", 2)
	else
		self:Notify("Config", "Não encontrado: '" .. name .. "'", 2)
	end
end

-- Slots HVH (PLACEHOLDERS SEGUROS) ---------------------------
NebulaUI.Visuals = {
	-- Render apenas nos targets fornecidos pelo servidor
	Render = function(self, targetList, color)
		for _, inst in ipairs(targetList or {}) do
			if typeof(inst) == "Instance" then
				local h = Instance.new("Highlight")
				h.FillTransparency = 1
				h.OutlineColor = color or Color3.fromRGB(126, 144, 255)
				h.OutlineTransparency = 0
				h.Parent = inst
				task.delay(0.25, function() if h then h:Destroy() end end)
			end
		end
	end,
}

NebulaUI.Targeting = {
	SetTargets = function(self, targetList) self._targets = targetList or {} end,
	Acquire = function(self)
		-- retorna dados sem mover câmera/mouse
		return self._targets or {}
	end
}

NebulaUI.Movement = {
	Request = function(self, params)
		-- meramente demonstração — o servidor deveria validar/realizar qualquer ação real
		print("Movement.Request", params)
	end
}

return NebulaUI

--[[
==============================================================
=                    NebulaUI_Demo (LocalScript)             =
=  COPIE a partir daqui para um LocalScript separado.        =
==============================================================

local NebulaUI = require(path.to.NebulaUI) -- ajuste o caminho

local ui = NebulaUI.new({
	Name = "TSUO Nebula++",
	IconAssetId = "rbxassetid://0",
	ThemePreset = "NebulaDark",
	Glass = 0.12,
})

local win = ui:Window()

-- Tabs --------------------------------------------------------
local tabTheme = win:AddTab("Theme", "🎨")
local tabVisuals = win:AddTab("Visuals", "👁️")
local tabTarget = win:AddTab("Targeting", "🎯")
local tabCamera = win:AddTab("Camera", "🎥")
local tabMove = win:AddTab("Movement", "🕹️")
local tabUtil = win:AddTab("Util", "🧩")

-- THEME TAB ---------------------------------------------------
local secTheme = tabTheme:AddSection("Gerenciador de Tema")
local ddTheme = tabTheme:AddDropdown("Preset", {"NebulaDark","NebulaLight","Matrix","Sakura","Aqua"}, 1, function(opt)
	ui:SetThemeByName(opt)
end)

local cpAccent = tabTheme:AddColorPicker("Accent", Color3.fromRGB(126,144,255), function(c)
	ui.Theme.Accent = c; ui:_applyThemeToDescendants()
end)

local cpAccent2 = tabTheme:AddColorPicker("Accent2", Color3.fromRGB(88,204,237), function(c)
	ui.Theme.Accent2 = c; ui:_applyThemeToDescendants()
end)

local slGlass = tabTheme:AddSlider("Transparência (Glass)", 0, 60, 12, function(v)
	ui.Theme.Glass = v/100; ui:_applyThemeToDescendants()
end)

local tbIcon = tabUtil:AddTextbox("Icon RBX ID", "rbxassetid://...", function(txt)
	-- simples troca visual do ícone
	-- (no módulo, o ícone fica em TitleBar.Icon)
	local bar = ui.Window:FindFirstChild("TitleBar")
	if bar and bar:FindFirstChild("Icon") then bar.Icon.Image = txt end
end)

-- VISUALS TAB (PLACEHOLDER) ----------------------------------
local ddMethod = tabVisuals:AddDropdown("Método Visual", {"Outline","Highlight","Chams (demo)"}, 2, function(opt) end)
local cpVisual = tabVisuals:AddColorPicker("Cor Visual", Color3.fromRGB(126,144,255), function(c) end)
local btRender = tabVisuals:AddButton("Render nos targets do servidor (demo)", function()
	-- Exemplo de chamada segura (apenas nos targets recebidos)
	local targets = {} -- preencha via RemoteEvent do seu servidor
	NebulaUI.Visuals:Render(targets, cpVisual:Get())
end)

-- TARGETING TAB (PLACEHOLDER) --------------------------------
local ddTeam = tabTarget:AddDropdown("Modo", {"Time","Inimigo","Neutro"}, 2, function(mode) end)
local slFOV = tabTarget:AddSlider("FOV (UI)", 30, 120, 80, function(v) end)
local kbActivate = tabTarget:AddKeybind("Ativar Targeting (UI)", Enum.KeyCode.Q, function(kc) end)

-- CAMERA TAB --------------------------------------------------
local slCamFOV = tabCamera:AddSlider("Camera FOV", 40, 100, workspace.CurrentCamera.FieldOfView, function(v)
	workspace.CurrentCamera.FieldOfView = v
end)

-- MOVEMENT TAB (PLACEHOLDER) ---------------------------------
local slSpeed = tabMove:AddSlider("WalkSpeed (pedido)", 16, 32, 16, function(v) end)
local btReq = tabMove:AddButton("Enviar pedido ao servidor", function()
	NebulaUI.Movement:Request({WalkSpeed = slSpeed:Get()})
	ui:Notify("Movement", "Aguardando servidor...", 2)
end)

-- UTIL TAB ----------------------------------------------------
local btSave = tabUtil:AddButton("Salvar Config (sessão)", function() ui:SaveConfig("default") end)
local btLoad = tabUtil:AddButton("Carregar Config (sessão)", function() ui:LoadConfig("default") end)

ui:Notify("TSUO Nebula++", "discord.gg/tsuo", 4)

]]
