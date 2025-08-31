--[[
    TSUO Purple UI Lib - v1.0 (Roblox Lua)
    Autor: ChatGPT-5 (pra Kayo)
    - Janela roxa, tabs com emoji, dock minimizável/móvel, ícone rbxassetid
    - Componentes: Toggle, Slider, Dropdown, Button, Keybind
    - Hooks: OnStep, OnInput
    - Config: Save/Load (usa writefile/readfile se existir)
    - Demos: ESP (names/dist/box/tracers), Fly/Speed/Jump/Noclip/TP/WalkOnWater/BigHitbox
--]]

-- // ========= UTIL =========
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local function safe(fn, ...)
    local ok, res = pcall(fn, ...)
    if not ok then warn("[TSUO LIB]", res) end
    return ok, res
end

local function toAsset(id)
    id = tostring(id or "")
    if id == "" then return "" end
    if id:match("^rbxassetid://") then return id end
    return "rbxassetid://" .. id
end

local function tsv(gui, props, dur, style, dir)
    local tw = TweenService:Create(gui, TweenInfo.new(dur or 0.15, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out), props)
    tw:Play()
    return tw
end

local function MakeDraggable(handle, target)
    target = target or handle
    local dragging, startPos, startInputPos = false, nil, nil
    handle.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            startPos = target.Position
            startInputPos = i.Position
            i.Changed:Connect(function()
                if i.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    handle.InputChanged:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = i.Position - startInputPos
                target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
end

local function new(class, props)
    local inst = Instance.new(class)
    for k,v in pairs(props or {}) do inst[k] = v end
    return inst
end

local function roundify(gui, radius)
    local r = new("UICorner", {CornerRadius = UDim.new(0, radius or 10)})
    r.Parent = gui
    return r
end

local function strokify(gui, thicc, color, trans)
    local s = new("UIStroke", {Thickness = thicc or 1, Color = color or Color3.fromRGB(60,60,60), Transparency = trans or 0})
    s.Parent = gui
    return s
end

local function pad(gui, px)
    local p = new("UIPadding", {PaddingLeft = UDim.new(0, px), PaddingRight = UDim.new(0, px), PaddingTop = UDim.new(0, px), PaddingBottom = UDim.new(0, px)})
    p.Parent = gui
end

-- // ========= THEME =========
local Theme = {
    bg = Color3.fromRGB(20, 14, 32),
    panel = Color3.fromRGB(30, 20, 55),
    panel2 = Color3.fromRGB(36, 24, 66),
    accent = Color3.fromRGB(158, 99, 255),
    accent2 = Color3.fromRGB(120, 82, 220),
    text = Color3.fromRGB(235, 230, 255),
    subtext = Color3.fromRGB(180, 170, 210),
    stroke = Color3.fromRGB(80, 60, 110),
    green = Color3.fromRGB(90, 200, 120),
    red = Color3.fromRGB(220, 80, 100)
}

-- // ========= EVENT BUS / HOOKS =========
local function Signal()
    local ev = Instance.new("BindableEvent")
    return {
        Connect = function(_, f) return ev.Event:Connect(f) end,
        Fire = function(_, ...) ev:Fire(...) end
    }
end

-- // ========= CONFIG =========
local function hasfs()
    return (typeof(writefile) == "function") and (typeof(readfile) == "function") and (typeof(isfile) == "function")
end

-- // ========= LIB =========
local Lib = {}
Lib.__index = Lib

local UI = {} -- classes
UI.__index = UI

function Lib:CreateWindow(opts)
    opts = opts or {}
    local title = opts.title or "TSUO HUB"
    local iconId = opts.icon or ""
    local toggleKey = opts.toggleKey or Enum.KeyCode.RightShift

    local sg = new("ScreenGui", {Name = "TSUO_Purple_Lib", ResetOnSpawn = false, ZIndexBehavior = Enum.ZIndexBehavior.Global})
    sg.Parent = game:GetService("CoreGui")

    -- Dock (quadradinho) – aparece quando minimiza
    local dock = new("Frame", {
        Name = "Dock",
        Size = UDim2.new(0, 44, 0, 44),
        Position = UDim2.new(0, 20, 0, 200),
        BackgroundColor3 = Theme.panel2,
        Visible = false,
        Active = true
    })
    dock.Parent = sg
    roundify(dock, 10)
    strokify(dock, 1, Theme.stroke)
    local dockBtn = new("ImageButton", {
        Size = UDim2.new(1, -8, 1, -8),
        Position = UDim2.new(0, 4, 0, 4),
        BackgroundTransparency = 1,
        Image = (iconId ~= "" and toAsset(iconId)) or "rbxassetid://7072718362"
    })
    dockBtn.Parent = dock
    MakeDraggable(dock, dock)

    -- Janela principal
    local win = new("Frame", {
        Name = "Window",
        Size = UDim2.new(0, 560, 0, 420),
        Position = UDim2.new(0.5, -280, 0.5, -210),
        BackgroundColor3 = Theme.panel,
        Active = true
    })
    win.Parent = sg
    roundify(win, 14)
    strokify(win, 1, Theme.stroke)
    MakeDraggable(win, win)

    -- Header
    local header = new("Frame", {
        Name = "Header",
        Size = UDim2.new(1, 0, 0, 46),
        BackgroundColor3 = Theme.panel2
    })
    header.Parent = win
    roundify(header, 14)
    strokify(header, 1, Theme.stroke)
    pad(header, 10)

    local icon = new("ImageLabel", {
        Size = UDim2.new(0, 28, 0, 28),
        Position = UDim2.new(0, 10, 0.5, -14),
        BackgroundTransparency = 1,
        Image = (iconId ~= "" and toAsset(iconId)) or "rbxassetid://7072718362"
    })
    icon.Parent = header

    local titleLbl = new("TextLabel", {
        Size = UDim2.new(1, -120, 1, 0),
        Position = UDim2.new(0, 48, 0, 0),
        BackgroundTransparency = 1,
        Text = title,
        TextColor3 = Theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    titleLbl.Parent = header

    -- Minimizar
    local minimize = new("TextButton", {
        Size = UDim2.new(0, 34, 0, 34),
        Position = UDim2.new(1, -44, 0.5, -17),
        BackgroundColor3 = Theme.accent,
        Text = "–",
        TextScaled = true,
        TextColor3 = Color3.new(1,1,1)
    })
    minimize.Parent = header
    roundify(minimize, 8)

    -- Tab bar
    local tabbar = new("Frame", {
        Name = "TabBar",
        Size = UDim2.new(1, -20, 0, 40),
        Position = UDim2.new(0, 10, 0, 56),
        BackgroundColor3 = Theme.bg
    })
    tabbar.Parent = win
    roundify(tabbar, 10)
    strokify(tabbar, 1, Theme.stroke)
    pad(tabbar, 8)
    local tabLayout = new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder})
    tabLayout.Parent = tabbar

    -- Page container
    local pages = new("Frame", {
        Name = "Pages",
        Size = UDim2.new(1, -20, 1, -116),
        Position = UDim2.new(0, 10, 0, 106),
        BackgroundColor3 = Theme.bg
    })
    pages.Parent = win
    roundify(pages, 12)
    strokify(pages, 1, Theme.stroke)
    pad(pages, 10)

    local TabClass = {}
    TabClass.__index = TabClass

    function TabClass:AddSection(title)
        local sec = new("Frame", {Size = UDim2.new(1, -10, 0, 36), BackgroundTransparency = 1})
        sec.Parent = self.body
        local lbl = new("TextLabel", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Text = title or "Seção",
            TextColor3 = Theme.subtext, Font = Enum.Font.GothamBold, TextSize = 16,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        lbl.Parent = sec
        return sec
    end

    local function addRow(parent, height)
        local row = new("Frame", {
            Size = UDim2.new(1, 0, 0, height or 40),
            BackgroundColor3 = Theme.panel
        })
        row.Parent = parent
        roundify(row, 10)
        strokify(row, 1, Theme.stroke)
        pad(row, 10)
        local layout = new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 10)})
        layout.Parent = row
        return row
    end

    function TabClass:AddToggle(text, default, callback)
        local row = addRow(self.body, 40)
        local lbl = new("TextLabel", {
            Size = UDim2.new(1, -90, 1, 0), BackgroundTransparency = 1,
            Text = text, TextColor3 = Theme.text, TextXAlignment = Enum.TextXAlignment.Left,
            Font = Enum.Font.GothamSemibold, TextSize = 16
        })
        lbl.Parent = row
        local btn = new("TextButton", {
            Size = UDim2.new(0, 70, 0, 28),
            BackgroundColor3 = Theme.red, Text = "OFF",
            TextColor3 = Color3.new(1,1,1), Font = Enum.Font.GothamBold, TextSize = 14
        })
        btn.Parent = row
        roundify(btn, 8)

        local state = default and true or false
        local function apply()
            btn.Text = state and "ON" or "OFF"
            tsv(btn, {BackgroundColor3 = state and Theme.green or Theme.red})
            if callback then safe(callback, state) end
        end
        apply()
        btn.MouseButton1Click:Connect(function() state = not state; apply() end)
        return {
            Set = function(_, v) state = v; apply() end,
            Get = function(_) return state end
        }
    end

    function TabClass:AddSlider(text, min, max, default, callback, suffix)
        local row = addRow(self.body, 56)
        local lbl = new("TextLabel", {
            Size = UDim2.new(1, -180, 1, -16), Position = UDim2.new(0,0,0,0),
            BackgroundTransparency = 1, Text = text, TextColor3 = Theme.text,
            TextXAlignment = Enum.TextXAlignment.Left, Font = Enum.Font.GothamSemibold, TextSize = 16
        })
        lbl.Parent = row

        local valueLbl = new("TextLabel", {
            Size = UDim2.new(0, 70, 0, 24), Position = UDim2.new(1, -80, 0, 8),
            BackgroundTransparency = 1, Text = tostring(default or min) .. (suffix or ""),
            TextColor3 = Theme.subtext, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Right
        })
        valueLbl.Parent = row

        local bar = new("Frame", {
            Size = UDim2.new(1, -20, 0, 10), Position = UDim2.new(0, 10, 1, -18),
            BackgroundColor3 = Theme.bg
        })
        bar.Parent = row
        roundify(bar, 6)
        strokify(bar, 1, Theme.stroke)

        local fill = new("Frame", {
            Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = Theme.accent
        })
        fill.Parent = bar
        roundify(fill, 6)

        local cur = math.clamp(default or min, min, max)
        local down = false
        local function render()
            local pct = (cur - min) / (max - min)
            fill.Size = UDim2.new(pct, 0, 1, 0)
            valueLbl.Text = string.format("%d%s", cur, suffix or "")
            if callback then safe(callback, cur) end
        end
        render()
        bar.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 then
                down = true
                local rel = (i.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X
                cur = math.floor(min + math.clamp(rel, 0, 1) * (max - min) + 0.5)
                render()
            end
        end)
        bar.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 then down = false end
        end)
        bar.InputChanged:Connect(function(i)
            if down and i.UserInputType == Enum.UserInputType.MouseMovement then
                local rel = (i.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X
                cur = math.floor(min + math.clamp(rel, 0, 1) * (max - min) + 0.5)
                render()
            end
        end)
        return {
            Set = function(_, v) cur = math.clamp(v, min, max); render() end,
            Get = function(_) return cur end
        }
    end

    function TabClass:AddDropdown(text, list, default, callback)
        local row = addRow(self.body, 40)
        local lbl = new("TextLabel", {
            Size = UDim2.new(1, -180, 1, 0), BackgroundTransparency = 1, Text = text,
            TextColor3 = Theme.text, Font = Enum.Font.GothamSemibold, TextSize = 16,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        lbl.Parent = row
        local btn = new("TextButton", {
            Size = UDim2.new(0, 160, 0, 28),
            BackgroundColor3 = Theme.panel2, Text = default or "Selecionar",
            TextColor3 = Theme.text, Font = Enum.Font.Gotham, TextSize = 14
        })
        btn.Parent = row
        roundify(btn, 8)
        strokify(btn, 1, Theme.stroke)

        local open = false
        local menu = new("Frame", {
            Size = UDim2.new(0, 160, 0, 0), Position = UDim2.new(0, btn.AbsolutePosition.X, 0, btn.AbsolutePosition.Y + btn.AbsoluteSize.Y + 4),
            BackgroundColor3 = Theme.panel2, Visible = false
        })
        menu.Parent = row
        roundify(menu, 8)
        strokify(menu, 1, Theme.stroke)
        local vlist = new("UIListLayout", {Padding = UDim.new(0, 2)})
        vlist.Parent = menu

        local current = default
        local function set(val)
            current = val
            btn.Text = tostring(val)
            if callback then safe(callback, val) end
        end
        btn.MouseButton1Click:Connect(function()
            open = not open
            menu.Visible = open
            tsv(menu, {Size = UDim2.new(0, 160, 0, open and math.min(#list*28+8, 200) or 0)}, 0.12)
        end)
        for _,opt in ipairs(list or {}) do
            local o = new("TextButton", {
                Size = UDim2.new(1, -8, 0, 26), Position = UDim2.new(0, 4, 0, 4),
                BackgroundColor3 = Theme.panel, Text = tostring(opt),
                TextColor3 = Theme.text, Font = Enum.Font.Gotham, TextSize = 14
            })
            o.Parent = menu; roundify(o, 6)
            o.MouseButton1Click:Connect(function() set(opt); open=false; menu.Visible=false end)
        end
        if default then set(default) end
        return {Get=function() return current end, Set=function(_,v) set(v) end}
    end

    function TabClass:AddButton(text, callback)
        local row = addRow(self.body, 40)
        local btn = new("TextButton", {
            Size = UDim2.new(1, 0, 1, 0), BackgroundColor3 = Theme.accent,
            Text = text, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.GothamBold, TextSize = 16
        })
        btn.Parent = row; roundify(btn, 10)
        btn.MouseButton1Click:Connect(function() if callback then safe(callback) end end)
        return btn
    end

    function TabClass:AddKeybind(text, defaultKey, callback)
        local row = addRow(self.body, 40)
        local lbl = new("TextLabel", {
            Size = UDim2.new(1, -180, 1, 0), BackgroundTransparency = 1, Text = text,
            TextColor3 = Theme.text, Font = Enum.Font.GothamSemibold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left
        }); lbl.Parent = row
        local btn = new("TextButton", {
            Size = UDim2.new(0, 160, 0, 28), BackgroundColor3 = Theme.panel2,
            Text = defaultKey and defaultKey.Name or "Pressione...",
            TextColor3 = Theme.text, Font = Enum.Font.Gotham, TextSize = 14
        }); btn.Parent = row; roundify(btn, 8); strokify(btn, 1, Theme.stroke)
        local current = defaultKey
        btn.MouseButton1Click:Connect(function()
            btn.Text = "Aguardando..."
            local conn; conn = UserInputService.InputBegan:Connect(function(i, gpe)
                if gpe then return end
                if i.KeyCode ~= Enum.KeyCode.Unknown then
                    current = i.KeyCode
                    btn.Text = current.Name
                    if callback then safe(callback, current) end
                    conn:Disconnect()
                end
            end)
        end)
        return {Get=function() return current end}
    end

    local tabs = {}
    local activeTab

    local function openTab(t)
        for _,tb in ipairs(tabs) do
            tb.page.Visible = false
            tsv(tb.btn, {BackgroundColor3 = Theme.panel2}, 0.1)
        end
        t.page.Visible = true
        tsv(t.btn, {BackgroundColor3 = Theme.accent}, 0.1)
        activeTab = t
    end

    function UI:AddTab(label, emoji)
        local tbtn = new("TextButton", {
            Size = UDim2.new(0, 150, 1, -8),
            BackgroundColor3 = Theme.panel2,
            Text = (emoji and (emoji .. " ") or "") .. tostring(label),
            TextColor3 = Theme.text, Font = Enum.Font.GothamBold, TextSize = 14
        })
        tbtn.Parent = tabbar; roundify(tbtn, 8); strokify(tbtn, 1, Theme.stroke)

        local page = new("ScrollingFrame", {
            Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Visible = false, CanvasSize = UDim2.new(0,0,0,0), ScrollBarThickness = 4
        })
        page.Parent = pages
        local vlist = new("UIListLayout", {Padding = UDim.new(0, 8)})
        vlist.Parent = page

        local tab = setmetatable({btn=tbtn, page=page, body=page}, TabClass)
        tbtn.MouseButton1Click:Connect(function() openTab(tab) end)
        table.insert(tabs, tab)
        if not activeTab then openTab(tab) end
        return tab
    end

    -- Toggle UI by hotkey
    UserInputService.InputBegan:Connect(function(i, gpe)
        if gpe then return end
        if i.KeyCode == toggleKey then
            win.Visible = not win.Visible
            dock.Visible = not win.Visible
        end
    end)

    minimize.MouseButton1Click:Connect(function()
        win.Visible = false
        dock.Visible = true
    end)
    dockBtn.MouseButton1Click:Connect(function()
        dock.Visible = false
        win.Visible = true
    end)

    -- Hooks
    local stepSig, inputSig = Signal(), Signal()
    RunService.RenderStepped:Connect(function(dt) stepSig:Fire(dt) end)
    UserInputService.InputBegan:Connect(function(i, g) inputSig:Fire("began", i, g) end)
    UserInputService.InputEnded:Connect(function(i, g) inputSig:Fire("ended", i, g) end)

    local uiObj = setmetatable({
        _sg = sg, _win = win, _dock = dock, _title = titleLbl, _icon = icon,
        OnStep = function(_, fn) return stepSig:Connect(fn) end,
        OnInput = function(_, fn) return inputSig:Connect(fn) end,
        AddTab = UI.AddTab,
        SetIcon = function(_, id) icon.Image = toAsset(id) dockBtn.Image = toAsset(id) end,
        SetTitle = function(_, t) titleLbl.Text = t end,
        SaveConfig = function(_, name, data)
            local json = HttpService:JSONEncode(data)
            if hasfs() then writefile(name..".json", json) end
            return json
        end,
        LoadConfig = function(_, name)
            if hasfs() and isfile(name..".json") then
                return HttpService:JSONDecode(readfile(name..".json"))
            end
            return nil
        end
    }, UI)

    return uiObj
end

-- // ========= DEMO (ESP + FUNÇÕES) =========
-- Configura aqui o ícone da janela:
local ICON_ID = "7072718362" -- mude para seu rbxassetid (apenas números ok)

local Window = Lib:CreateWindow({
    title = "🟣 TSUO Script Hub",
    icon = ICON_ID,
    toggleKey = Enum.KeyCode.RightShift, -- abre/fecha pela tecla também
})

-- ==== TAB ESP ====
local espTab = Window:AddTab("ESP", "🛰️")
espTab:AddSection("ESP Players (Nome/Dist/Box/Tracers)")

local ESP = {
    enabled = false,
    showNames = true,
    showDistance = true,
    showBoxes = true,
    showTracers = false,
    color = Color3.fromRGB(158, 99, 255),

    drawings = {}, -- por player: {nameLabel, distLabel, box, tracer}
    usingDrawing = false
}

-- Detecta Drawing API
ESP.usingDrawing = (typeof(Drawing) == "table" or typeof(Drawing) == "function")

local function clearESPFor(plr)
    local rec = ESP.drawings[plr]
    if not rec then return end
    if ESP.usingDrawing then
        for _,d in pairs(rec) do if d and d.Remove then pcall(function() d:Remove() end) end end
    else
        for _,g in pairs(rec) do if g and g.Destroy then pcall(function() g:Destroy() end) end end
    end
    ESP.drawings[plr] = nil
end

local function makeESPFor(plr)
    if plr == LocalPlayer then return end
    clearESPFor(plr)
    ESP.drawings[plr] = {}

    if ESP.usingDrawing then
        -- Simple Drawing setup (labels + tracer line)
        local txt = Drawing.new("Text"); txt.Size = 14; txt.Center = true; txt.Outline = true; txt.Color = Color3.new(1,1,1)
        local dist = Drawing.new("Text"); dist.Size = 13; dist.Center = true; dist.Outline = true; dist.Color = Color3.fromRGB(200,200,255)
        local line = Drawing.new("Line"); line.Thickness = 1.5; line.Color = ESP.color
        local box = Drawing.new("Square"); box.Filled = false; box.Thickness = 1.5; box.Color = ESP.color
        ESP.drawings[plr] = {txt = txt, dist = dist, line = line, box = box}
    else
        -- Fallback com BillboardGui + BoxHandleAdornment
        local char = plr.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        local hrp = char.HumanoidRootPart

        local bb = Instance.new("BillboardGui")
        bb.AlwaysOnTop = true; bb.Size = UDim2.new(0,200,0,50); bb.Adornee = hrp; bb.Name = "ESP_BB_"..plr.UserId
        bb.Parent = CoreGui or Window._sg

        local nameLbl = Instance.new("TextLabel")
        nameLbl.BackgroundTransparency = 1; nameLbl.TextColor3 = Color3.new(1,1,1); nameLbl.TextStrokeTransparency = 0.5
        nameLbl.Font = Enum.Font.GothamBold; nameLbl.TextSize = 14; nameLbl.Size = UDim2.new(1,0,0,20)
        nameLbl.Parent = bb

        local distLbl = Instance.new("TextLabel")
        distLbl.BackgroundTransparency = 1; distLbl.TextColor3 = Color3.fromRGB(200,200,255); distLbl.TextStrokeTransparency = 0.5
        distLbl.Font = Enum.Font.Gotham; distLbl.TextSize = 13; distLbl.Position = UDim2.new(0,0,0,20); distLbl.Size = UDim2.new(1,0,0,20)
        distLbl.Parent = bb

        local bha = Instance.new("BoxHandleAdornment")
        bha.AlwaysOnTop = true; bha.Color3 = ESP.color; bha.Transparency = 0; bha.ZIndex = 2; bha.Adornee = char; bha.Size = Vector3.new(4,6,3)
        bha.Parent = Window._sg

        ESP.drawings[plr] = {bb = bb, nameLbl = nameLbl, distLbl = distLbl, box = bha}
    end
end

local function refreshAllESP()
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then makeESPFor(plr) end
    end
end

Players.PlayerAdded:Connect(makeESPFor)
Players.PlayerRemoving:Connect(clearESPFor)

local togESP = espTab:AddToggle("ESP Geral", false, function(v)
    ESP.enabled = v
    if v then refreshAllESP() else
        for plr,_ in pairs(ESP.drawings) do clearESPFor(plr) end
    end
end)
local togNames = espTab:AddToggle("Mostrar Nomes", true, function(v) ESP.showNames = v end)
local togDist = espTab:AddToggle("Mostrar Distância", true, function(v) ESP.showDistance = v end)
local togBox = espTab:AddToggle("Mostrar Box", true, function(v) ESP.showBoxes = v end)
local togTr = espTab:AddToggle("Tracers (linha até a tela)", false, function(v) ESP.showTracers = v end)

Window:OnStep(function(dt)
    if not ESP.enabled then return end
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local char = plr.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                local dist = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) and (hrp.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude or 0

                local rec = ESP.drawings[plr]
                if not rec then makeESPFor(plr) rec = ESP.drawings[plr] end

                if ESP.usingDrawing and rec then
                    rec.txt.Visible = ESP.showNames and onScreen
                    rec.dist.Visible = ESP.showDistance and onScreen
                    rec.box.Visible = ESP.showBoxes and onScreen
                    rec.line.Visible = ESP.showTracers and onScreen

                    if rec.txt.Visible then
                        rec.txt.Text = plr.Name
                        rec.txt.Position = Vector2.new(pos.X, pos.Y - 18)
                    end
                    if rec.dist.Visible then
                        rec.dist.Text = string.format("[%.0f m]", dist)
                        rec.dist.Position = Vector2.new(pos.X, pos.Y)
                    end
                    if rec.box.Visible then
                        -- approx 2D box
                        local h = 55 * (1 / (pos.Z/25))
                        local w = h * .6
                        rec.box.Size = Vector2.new(w, h)
                        rec.box.Position = Vector2.new(pos.X - w/2, pos.Y - h/2)
                        rec.box.Color = ESP.color
                    end
                    if rec.line.Visible then
                        local scr = Camera.ViewportSize
                        local start = Vector2.new(scr.X/2, scr.Y)
                        rec.line.From = start
                        rec.line.To = Vector2.new(pos.X, pos.Y)
                        rec.line.Color = ESP.color
                    end
                elseif rec then
                    if rec.nameLbl then rec.nameLbl.Visible = ESP.showNames end
                    if rec.distLbl then rec.distLbl.Visible = ESP.showDistance end
                    if rec.bb then rec.bb.Enabled = onScreen end
                    if rec.box then rec.box.Visible = ESP.showBoxes end
                    if rec.nameLbl then rec.nameLbl.Text = plr.Name end
                    if rec.distLbl then rec.distLbl.Text = string.format("[%.0f m]", dist) end
                end
            else
                clearESPFor(plr)
            end
        end
    end
end)

-- ==== TAB FUNÇÕES ====
local fnTab = Window:AddTab("Funções", "🛠️")
fnTab:AddSection("Movimentação/Utilidades")

local char = function() return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait() end

-- WalkSpeed & Jump
local wsSlider = fnTab:AddSlider("WalkSpeed", 16, 200, 16, function(v)
    local h = char():FindFirstChildOfClass("Humanoid"); if h then h.WalkSpeed = v end
end, " u")
local jpSlider = fnTab:AddSlider("JumpPower", 50, 300, 50, function(v)
    local h = char():FindFirstChildOfClass("Humanoid"); if h then h.UseJumpPower = true; h.JumpPower = v end
end, "")

-- Fly
local Fly = {enabled=false, speed=60, con=nil, gyro=nil, bv=nil}
local flyToggle = fnTab:AddToggle("Fly (WASD)", false, function(v)
    Fly.enabled = v
    local c = char(); local hrp = c:FindFirstChild("HumanoidRootPart")
    local hum = c:FindFirstChildOfClass("Humanoid")
    if v and hrp and hum then
        Fly.gyro = Instance.new("BodyGyro"); Fly.gyro.P = 9e4; Fly.gyro.MaxTorque = Vector3.new(9e9,9e9,9e9); Fly.gyro.CFrame = hrp.CFrame; Fly.gyro.Parent = hrp
        Fly.bv = Instance.new("BodyVelocity"); Fly.bv.MaxForce = Vector3.new(9e9,9e9,9e9); Fly.bv.Velocity = Vector3.new(); Fly.bv.Parent = hrp
        hum.PlatformStand = true
        Fly.con = RunService.Heartbeat:Connect(function()
            local dir = Vector3.new()
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
            Fly.bv.Velocity = dir * Fly.speed
            Fly.gyro.CFrame = Camera.CFrame
        end)
    else
        if Fly.con then Fly.con:Disconnect() Fly.con = nil end
        if Fly.gyro then Fly.gyro:Destroy() Fly.gyro=nil end
        if Fly.bv then Fly.bv:Destroy() Fly.bv=nil end
        if hum then hum.PlatformStand = false end
    end
end)
local flySpeed = fnTab:AddSlider("Velocidade do Fly", 10, 250, 60, function(v) Fly.speed = v end, " u")

-- Noclip
local NoClip = {enabled=false, con=nil}
fnTab:AddToggle("NoClip", false, function(v)
    NoClip.enabled = v
    if v then
        NoClip.con = RunService.Stepped:Connect(function()
            local c = char()
            for _,p in ipairs(c:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end)
    else
        if NoClip.con then NoClip.con:Disconnect(); NoClip.con=nil end
    end
end)

-- TP Player
local function otherPlayers()
    local l = {}
    for _,p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then table.insert(l, p.Name) end end
    if #l==0 then table.insert(l, "Ninguém") end
    return l
end
local tpDrop = fnTab:AddDropdown("Teleportar para Player", otherPlayers(), nil, function() end)
fnTab:AddButton("TP", function()
    local name = tpDrop:Get()
    if name and Players:FindFirstChild(name) then
        local target = Players[name].Character and Players[name].Character:FindFirstChild("HumanoidRootPart")
        if target then
            local c = char(); local hrp = c:FindFirstChild("HumanoidRootPart")
            if hrp then hrp.CFrame = target.CFrame + target.CFrame.LookVector * 3 end
        end
    end
end)

-- Walk on Water (flutua quando entra em Swimming)
local WaterWalk = {enabled=false, pad=nil, con=nil}
fnTab:AddToggle("Walk on Water", false, function(v)
    WaterWalk.enabled = v
    local h = char():FindFirstChildOfClass("Humanoid")
    if v and h then
        WaterWalk.pad = Instance.new("Part"); WaterWalk.pad.Anchored=true; WaterWalk.pad.CanCollide=true; WaterWalk.pad.Transparency=1
        WaterWalk.pad.Size = Vector3.new(6, 0.2, 6); WaterWalk.pad.Parent = workspace
        WaterWalk.con = RunService.Heartbeat:Connect(function()
            local c = char(); local hrp = c:FindFirstChild("HumanoidRootPart"); local hum = c:FindFirstChildOfClass("Humanoid")
            if hrp and hum then
                if hum:GetState() == Enum.HumanoidStateType.Swimming then
                    WaterWalk.pad.Position = Vector3.new(hrp.Position.X, hrp.Position.Y - 3, hrp.Position.Z)
                else
                    -- guarda fora de vista quando não está na água
                    WaterWalk.pad.Position = Vector3.new(0, -1000, 0)
                end
            end
        end)
    else
        if WaterWalk.con then WaterWalk.con:Disconnect(); WaterWalk.con=nil end
        if WaterWalk.pad then WaterWalk.pad:Destroy(); WaterWalk.pad=nil end
    end
end)

-- Big Hitbox (local: aumenta RootPart dos OUTROS no cliente)
local BigHB = {enabled=false, size=Vector3.new(6,6,6), original={}}
fnTab:AddToggle("Big Hitbox (local)", false, function(v)
    BigHB.enabled = v
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local c = plr.Character; local hrp = c and c:FindFirstChild("HumanoidRootPart")
            if hrp then
                if v then
                    if not BigHB.original[plr] then BigHB.original[plr] = hrp.Size end
                    hrp.Size = BigHB.size; hrp.Massless = true; hrp.CanCollide = false
                else
                    if BigHB.original[plr] then
                        hrp.Size = BigHB.original[plr]
                    end
                end
            end
        end
    end
end)
fnTab:AddSlider("HB Tamanho (aresta)", 2, 12, 6, function(v) BigHB.size = Vector3.new(v,v,v) end, "")

-- ==== TAB CONFIG ====
local cfgTab = Window:AddTab("Config", "⚙️")
cfgTab:AddSection("Salvar / Carregar")

local cfgName = "tsuo_demo"
cfgTab:AddButton("Salvar Config", function()
    local data = {
        ws = wsSlider:Get(), jp = jpSlider:Get(),
        fly = {enabled=false, speed=flySpeed:Get()},
        esp = {main=togESP:Get(), names=togNames:Get(), dist=togDist:Get(), box=togBox:Get(), tracers=togTr:Get()}
    }
    local json = Window:SaveConfig(cfgName, data)
    print("[Config salva]", json)
end)
cfgTab:AddButton("Carregar Config", function()
    local data = Window:LoadConfig(cfgName)
    if data then
        wsSlider:Set(data.ws or 16)
        jpSlider:Set(data.jp or 50)
        flySpeed:Set(data.fly and data.fly.speed or 60)
        togESP.Set(togESP, data.esp and data.esp.main or false)
        togNames.Set(togNames, data.esp and data.esp.names or true)
        togDist.Set(togDist, data.esp and data.esp.dist or true)
        togBox.Set(togBox, data.esp and data.esp.box or true)
        togTr.Set(togTr, data.esp and data.esp.tracers or false)
        print("[Config carregada]")
    else
        warn("[Sem config salva ou FS indisponível]")
    end
end)

-- ==== TAB SOBRE ====
local aboutTab = Window:AddTab("Sobre", "💜")
aboutTab:AddSection("Créditos")
aboutTab:AddButton("Discord TSUO (placeholder)", function() print("coloca seu link aqui na versão final") end)

-- Fim: a UI já está rodando 👌
