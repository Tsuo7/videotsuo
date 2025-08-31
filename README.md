--[[
  Nebula UI Lib (TSUO Edition) v2.0 - SAFE BUILD
  Autor: ChatGPT-5 para Kayo (TSUO)
  Foco: A MELHOR UI LIB — perfeita para script hubs (tema roxo translúcido, simétrica,
  proporcional, responsiva), com:
    - Dock minimizável (ícone) arrastável + Janela arrastável
    - Tabs com emojis, autoscroll consistente, paddings e strokes uniformes
    - Componentes: Toggle, Slider, Dropdown, Button, Keybind, ColorPicker (RGB)
    - Hooks: OnStep, OnInput
    - Persistência: Save/Load (writefile/readfile se disponível)
    - Startup overlay com contagem 5..1 e notificação "discord.gg/tsuo"
    - Demo segura: Tema/Notificações, Câmera (FOV & Zoom RMB), Crosshair visual, HUD (FPS)
  
  IMPORTANTE:
    - Esta build não implementa ou instrui recursos de trapaça (ex.: aimbot, ESP/wallhack,
      noclip, fly, alteração de atributos de personagem, etc.).
    - Objetivo é fornecer uma base de UI profissional, performática e pronta para hubs legais.
]]

--// Services
local Players            = game:GetService("Players")
local RunService         = game:GetService("RunService")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")
local StarterGui         = game:GetService("StarterGui")
local Lighting           = game:GetService("Lighting")
local HttpService        = game:GetService("HttpService")
local CoreGui            = game:GetService("CoreGui")

local LocalPlayer        = Players.LocalPlayer
local Camera             = workspace.CurrentCamera

--// Utils
local function safe(fn, ...)
    local ok, r = pcall(fn, ...)
    if not ok then warn("[NebulaUI]", r) end
    return ok, r
end

local function toAsset(id)
    if not id or id == "" then return "" end
    id = tostring(id)
    if id:match("^rbxassetid://") then return id end
    return "rbxassetid://" .. id
end

local function tsv(inst, props, dur, style, dir)
    local tw = TweenService:Create(inst, TweenInfo.new(dur or 0.18, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out), props)
    tw:Play(); return tw
end

local function MakeDraggable(handle, target)
    target = target or handle
    local dragging, startPos, startIn
    handle.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true; startPos = target.Position; startIn = i.Position
            i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then dragging = false end end)
        end
    end)
    handle.InputChanged:Connect(function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - startIn
            target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

local function autoscroll(sf)
    local layout = sf:FindFirstChildOfClass("UIListLayout") or sf:FindFirstChildOfClass("UIGridLayout")
    if not layout then return end
    local function apply() sf.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 12) end
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(apply); apply()
end

local function hasfs()
    return typeof(writefile)=="function" and typeof(readfile)=="function" and typeof(isfile)=="function"
end

local function Signal()
    local ev = Instance.new("BindableEvent")
    return {
        Connect = function(_, f) return ev.Event:Connect(f) end,
        Fire    = function(_, ...) ev:Fire(...) end
    }
end

--// Theme
local Theme = {
    bg      = Color3.fromRGB(18, 12, 28),
    panel   = Color3.fromRGB(28, 20, 44),
    panel2  = Color3.fromRGB(36, 24, 66),
    accent  = Color3.fromRGB(158, 99, 255),
    accent2 = Color3.fromRGB(120, 82, 220),
    text    = Color3.fromRGB(235, 230, 255),
    subtext = Color3.fromRGB(180, 170, 210),
    stroke  = Color3.fromRGB(70, 55, 100),
}

--// Helpers
local function new(class, props) local o = Instance.new(class); for k,v in pairs(props or {}) do o[k]=v end; return o end
local function roundify(gui, px) local c=new("UICorner",{CornerRadius=UDim.new(0, px or 10)}); c.Parent=gui; return c end
local function strokify(gui, thicc, color, trans) local s=new("UIStroke",{Thickness=thicc or 1, Color=color or Theme.stroke, Transparency=trans or 0}); s.Parent=gui; return s end
local function pad(gui, px) local p=new("UIPadding",{PaddingLeft=UDim.new(0,px),PaddingRight=UDim.new(0,px),PaddingTop=UDim.new(0,px),PaddingBottom=UDim.new(0,px)}); p.Parent=gui end

local function ensureBlur()
    local b = Lighting:FindFirstChild("NebulaBlur") or Instance.new("BlurEffect")
    b.Name = "NebulaBlur"; b.Size = 0; b.Parent = Lighting
    return b
end

--// Core
local Nebula = {}; Nebula.__index = Nebula

function Nebula:Create(opts)
    opts = opts or {}
    local title     = opts.title or "Nebula | TSUO Hub"
    local iconId    = opts.icon or "7072718362"
    local toggleKey = opts.toggleKey or Enum.KeyCode.RightShift

    -- ScreenGui
    local sg = new("ScreenGui", {Name="NebulaUI", ZIndexBehavior=Enum.ZIndexBehavior.Global, ResetOnSpawn=false})
    sg.Parent = CoreGui

    -- Dock
    local dock = new("Frame", {Name="Dock", Size=UDim2.new(0, 44, 0, 44), Position=UDim2.new(0, 20, 0, 200),
        BackgroundColor3=Theme.panel2, BackgroundTransparency=0.05, Active=true, Visible=false})
    dock.Parent = sg; roundify(dock, 10); strokify(dock, 1.2)
    local dockBtn = new("ImageButton", {Size=UDim2.new(1,-8,1,-8), Position=UDim2.new(0,4,0,4), BackgroundTransparency=1, Image=toAsset(iconId)})
    dockBtn.Parent = dock
    MakeDraggable(dock, dock)

    -- Window
    local WIN_W, WIN_H = 640, 500
    local win = new("Frame", {Name="Window", Size=UDim2.new(0, WIN_W, 0, WIN_H), Position=UDim2.new(0.5, -WIN_W/2, 0.5, -WIN_H/2),
        BackgroundColor3=Theme.panel, BackgroundTransparency=0.1, Active=true})
    win.Parent = sg; roundify(win, 16); strokify(win, 1.2)

    -- Header
    local header = new("Frame", {Size=UDim2.new(1,0,0,54), BackgroundColor3=Theme.panel2, BackgroundTransparency=0.05})
    header.Parent=win; roundify(header, 16); strokify(header, 1); pad(header, 12)
    local hList = new("UIListLayout", {FillDirection=Enum.FillDirection.Horizontal, Padding=UDim.new(0,10), VerticalAlignment=Enum.VerticalAlignment.Center}); hList.Parent=header
    local icon = new("ImageLabel", {Size=UDim2.new(0,30,0,30), BackgroundTransparency=1, Image=toAsset(iconId)}); icon.Parent=header
    local titleLbl = new("TextLabel", {Size=UDim2.new(1,-150,1,0), BackgroundTransparency=1, Text=title, Font=Enum.Font.GothamBold, TextSize=20, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); titleLbl.Parent=header
    local minimize = new("TextButton", {Size=UDim2.new(0,38,0,38), Text="–", Font=Enum.Font.GothamBold, TextSize=22,
        TextColor3=Color3.new(1,1,1), BackgroundColor3=Theme.accent, AutoButtonColor=true}); minimize.Parent=header; roundify(minimize,9)

    -- Tabbar
    local tabbar = new("Frame", {Size=UDim2.new(1,-20,0,44), Position=UDim2.new(0,10,0,62), BackgroundColor3=Theme.bg, BackgroundTransparency=0.15})
    tabbar.Parent = win; roundify(tabbar, 10); strokify(tabbar, 1); pad(tabbar, 8)
    local tabLayout = new("UIListLayout", {FillDirection=Enum.FillDirection.Horizontal, Padding=UDim.new(0,8)}); tabLayout.Parent = tabbar

    -- Pages
    local pages = new("Frame", {Size=UDim2.new(1,-20,1,-(62+44+16)), Position=UDim2.new(0,10,0,62+44+10), BackgroundColor3=Theme.bg, BackgroundTransparency=0.15})
    pages.Parent = win; roundify(pages, 12); strokify(pages, 1); pad(pages, 10)

    local bluref = ensureBlur()

    local function setVisible(v)
        win.Visible = v; dock.Visible = not v; tsv(bluref, {Size = v and 8 or 0}, 0.2)
    end
    minimize.MouseButton1Click:Connect(function() setVisible(false) end)
    dockBtn.MouseButton1Click:Connect(function() setVisible(true) end)
    UserInputService.InputBegan:Connect(function(i,gpe) if not gpe and i.KeyCode==toggleKey then setVisible(not win.Visible) end end)

    MakeDraggable(header, win)

    -- Builders
    local function Section(parent, text)
        local t = new("TextLabel", {Text=text, Size=UDim2.new(1,0,0,22), BackgroundTransparency=1, Font=Enum.Font.GothamBold, TextSize=16, TextColor3=Theme.subtext, TextXAlignment=Enum.TextXAlignment.Left})
        t.Parent = parent; return t
    end
    local function Row(parent, height)
        local r = new("Frame", {Size=UDim2.new(1,0,0,height or 46), BackgroundColor3=Theme.panel, BackgroundTransparency=0.08})
        r.Parent=parent; roundify(r, 10); strokify(r, 1); pad(r, 10)
        local lay = new("UIListLayout", {FillDirection=Enum.FillDirection.Horizontal, Padding=UDim.new(0,10), VerticalAlignment=Enum.VerticalAlignment.Center}); lay.Parent=r
        return r
    end

    -- Tabs
    local tabs, activeTab = {}, nil
    local function OpenTab(t)
        for _,tb in ipairs(tabs) do tb.page.Visible=false; tsv(tb.btn, {BackgroundColor3=Theme.panel2}, 0.12) end
        t.page.Visible = true; tsv(t.btn, {BackgroundColor3=Theme.accent}, 0.12)
        activeTab = t
    end

    local UIb = {}
    function UIb:AddTab(label, emoji)
        local btn = new("TextButton", {Size=UDim2.new(0, 165, 1, -8), BackgroundColor3=Theme.panel2, Text=(emoji and (emoji.." ") or "")..label,
            TextColor3=Theme.text, Font=Enum.Font.GothamBold, TextSize=14, AutoButtonColor=true})
        btn.Parent=tabbar; roundify(btn, 8); strokify(btn, 1)

        local sf = new("ScrollingFrame", {Size=UDim2.new(1,0,1,0), BackgroundTransparency=1, Visible=false, ScrollBarThickness=5})
        sf.Parent = pages
        local v = new("UIListLayout", {Padding=UDim.new(0,8)}); v.Parent = sf; autoscroll(sf)

        local tab = {btn=btn, page=sf}
        btn.MouseButton1Click:Connect(function() OpenTab(tab) end)
        table.insert(tabs, tab); if not activeTab then OpenTab(tab) end

        tab.Section = function(_, text) return Section(sf, text) end
        tab.Row     = function(_, h) return Row(sf, h) end

        tab.AddToggle = function(_, label, default, cb)
            local r = Row(sf, 46)
            local t = new("TextLabel",{Size=UDim2.new(1,-110,1,0), BackgroundTransparency=1, Text=label, Font=Enum.Font.GothamSemibold, TextSize=15, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); t.Parent=r
            local b = new("TextButton",{Size=UDim2.new(0,84,0,28), BackgroundColor3=Theme.stroke, Text="OFF", Font=Enum.Font.GothamBold, TextSize=14, TextColor3=Color3.new(1,1,1)}); b.Parent=r; roundify(b,8)
            local state = default and true or false
            local function apply() b.Text = state and "ON" or "OFF"; tsv(b,{BackgroundColor3= state and Theme.accent or Theme.stroke},0.12); if cb then safe(cb, state) end end
            apply(); b.MouseButton1Click:Connect(function() state = not state; apply() end)
            return {Set=function(_,v) state=v; apply() end, Get=function() return state end}
        end

        tab.AddSlider = function(_, label, min, max, default, cb, suffix)
            local r = Row(sf, 60)
            local t = new("TextLabel",{Size=UDim2.new(1,-210,0,22), BackgroundTransparency=1, Text=label, Font=Enum.Font.GothamSemibold, TextSize=15, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); t.Parent=r
            local val = new("TextLabel",{Size=UDim2.new(0,90,0,22), BackgroundTransparency=1, Text="", Font=Enum.Font.Gotham, TextSize=14, TextColor3=Theme.subtext, TextXAlignment=Enum.TextXAlignment.Right}); val.Parent=r
            local bar = new("Frame",{Size=UDim2.new(1,-20,0,10), Position=UDim2.new(0,10,0,36), BackgroundColor3=Theme.bg, BackgroundTransparency=0.2}); bar.Parent=r; roundify(bar,6); strokify(bar,1)
            local fill = new("Frame",{Size=UDim2.new(0,0,1,0), BackgroundColor3=Theme.accent}); fill.Parent=bar; roundify(fill,6)

            local cur = math.clamp(default or min, min, max)
            local down=false
            local function render() local pct=(cur-min)/(max-min); fill.Size=UDim2.new(pct,0,1,0); val.Text=tostring(cur)..(suffix or ""); if cb then safe(cb, cur) end end
            render()
            local function setFromMouse(x) local rel=(x - bar.AbsolutePosition.X)/bar.AbsoluteSize.X; cur = math.floor(min + math.clamp(rel,0,1)*(max-min) + 0.5); render() end
            bar.InputBegan:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then down=true; setFromMouse(i.Position.X) end end)
            bar.InputEnded:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then down=false end end)
            bar.InputChanged:Connect(function(i) if down and i.UserInputType==Enum.UserInputType.MouseMovement then setFromMouse(i.Position.X) end end)
            return {Set=function(_,v) cur=math.clamp(v,min,max); render() end, Get=function() return cur end}
        end

        tab.AddDropdown = function(_, label, list, default, cb)
            local r = Row(sf, 46)
            local t = new("TextLabel",{Size=UDim2.new(1,-210,1,0), BackgroundTransparency=1, Text=label, Font=Enum.Font.GothamSemibold, TextSize=15, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); t.Parent=r
            local btn = new("TextButton",{Size=UDim2.new(0,190,0,28), BackgroundColor3=Theme.panel2, Text=default or "Selecionar", Font=Enum.Font.Gotham, TextSize=14, TextColor3=Theme.text}); btn.Parent=r; roundify(btn,8); strokify(btn,1)
            local open=false
            local menu = new("Frame",{Size=UDim2.new(0,190,0,0), Position=UDim2.new(0,0,0,34), BackgroundColor3=Theme.panel2, Visible=false}); menu.Parent=r; roundify(menu,8); strokify(menu,1); pad(menu,6)
            local v = new("UIListLayout",{Padding=UDim.new(0,4)}); v.Parent=menu
            local cur = default
            local function set(vl) cur=vl; btn.Text=tostring(vl); if cb then safe(cb, vl) end end
            btn.MouseButton1Click:Connect(function() open=not open; menu.Visible=open; tsv(menu,{Size=UDim2.new(0,190,0, open and math.min(#list*28+12, 220) or 0)},0.12) end)
            for _,opt in ipairs(list or {}) do
                local o = new("TextButton",{Size=UDim2.new(1,-8,0,26), BackgroundColor3=Theme.panel, Text=tostring(opt), Font=Enum.Font.Gotham, TextSize=14, TextColor3=Theme.text}); o.Parent=menu; roundify(o,6)
                o.MouseButton1Click:Connect(function() set(opt); open=false; menu.Visible=false end)
            end
            if default then set(default) end
            return {Get=function() return cur end, Set=function(_,v0) set(v0) end}
        end

        tab.AddButton = function(_, label, cb)
            local r = Row(sf, 46)
            local b = new("TextButton",{Size=UDim2.new(1,0,1,0), BackgroundColor3=Theme.accent, Text=label, Font=Enum.Font.GothamBold, TextSize=15, TextColor3=Color3.new(1,1,1)}); b.Parent=r; roundify(b,10)
            b.MouseButton1Click:Connect(function() if cb then safe(cb) end end); return b
        end

        tab.AddKeybind = function(_, label, defaultKey, cb)
            local r = Row(sf, 46)
            local tl = new("TextLabel",{Size=UDim2.new(1,-210,1,0), BackgroundTransparency=1, Text=label, Font=Enum.Font.GothamSemibold, TextSize=15, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); tl.Parent=r
            local b = new("TextButton",{Size=UDim2.new(0,190,0,28), BackgroundColor3=Theme.panel2, Text=(defaultKey and defaultKey.Name) or "Pressione...", Font=Enum.Font.Gotham, TextSize=14, TextColor3=Theme.text}); b.Parent=r; roundify(b,8); strokify(b,1)
            local cur = defaultKey
            b.MouseButton1Click:Connect(function()
                b.Text = "Aguardando..."
                local conn; conn = UserInputService.InputBegan:Connect(function(i, g)
                    if g then return end
                    if i.KeyCode ~= Enum.KeyCode.Unknown then cur=i.KeyCode; b.Text=cur.Name; if cb then safe(cb, cur) end; conn:Disconnect() end
                end)
            end); return {Get=function() return cur end}
        end

        -- ColorPicker (RGB)
        tab.AddColorPicker = function(_, label, defaultColor, cb)
            local r = Row(sf, 94)
            local t = new("TextLabel",{Size=UDim2.new(1,-230,0,22), BackgroundTransparency=1, Text=label, Font=Enum.Font.GothamSemibold, TextSize=15, TextColor3=Theme.text, TextXAlignment=Enum.TextXAlignment.Left}); t.Parent=r
            local preview = new("Frame",{Size=UDim2.new(0,40,0,40), BackgroundColor3=defaultColor or Theme.accent}); preview.Parent=r; roundify(preview,8); strokify(preview,1)
            local container = new("Frame",{Size=UDim2.new(1,-(40+10),0,64), BackgroundTransparency=1}); container.Parent=r
            local v = new("UIListLayout",{FillDirection=Enum.FillDirection.Vertical, Padding=UDim.new(0,6)}); v.Parent=container

            local function mkSlider(name, init)
                local f = new("Frame",{Size=UDim2.new(1,0,0,18), BackgroundTransparency=1}); f.Parent=container
                local lb = new("TextLabel",{Size=UDim2.new(0,24,1,0), BackgroundTransparency=1, Text=name, Font=Enum.Font.Gotham, TextSize=14, TextColor3=Theme.subtext}); lb.Parent=f
                local bar = new("Frame",{Size=UDim2.new(1,-64,1,-6), Position=UDim2.new(0,28,0,3), BackgroundColor3=Theme.bg, BackgroundTransparency=0.2}); bar.Parent=f; roundify(bar,6); strokify(bar,1)
                local fi  = new("Frame",{Size=UDim2.new((init or 0)/255,0,1,0), BackgroundColor3=Theme.accent}); fi.Parent=bar; roundify(fi,6)
                local val = new("TextLabel",{Size=UDim2.new(0,30,1,0), Position=UDim2.new(1,-30,0,0), BackgroundTransparency=1, Text=tostring(init or 0), Font=Enum.Font.Gotham, TextSize=13, TextColor3=Theme.subtext}); val.Parent=f
                local cur = init or 0; local down=false
                local function render() fi.Size=UDim2.new(cur/255,0,1,0); val.Text=tostring(cur) end
                render()
                local function setFromMouse(x) local rel=(x - bar.AbsolutePosition.X)/bar.AbsoluteSize.X; cur = math.clamp(math.floor(rel*255+0.5),0,255); render() end
                bar.InputBegan:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then down=true; setFromMouse(i.Position.X) end end)
                bar.InputEnded:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then down=false end end)
                bar.InputChanged:Connect(function(i) if down and i.UserInputType==Enum.UserInputType.MouseMovement then setFromMouse(i.Position.X) end end)
                return {Get=function() return cur end, Set=function(_,v0) cur=math.clamp(v0,0,255); render() end,
                        OnChange=function(fn) bar.InputChanged:Connect(function(i) if down and i.UserInputType==Enum.UserInputType.MouseMovement then fn() end end);
                                              bar.InputBegan:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then fn() end end) end}
            end

            local R = mkSlider("R", defaultColor and math.floor(defaultColor.R*255) or 158)
            local G = mkSlider("G", defaultColor and math.floor(defaultColor.G*255) or 99)
            local B = mkSlider("B", defaultColor and math.floor(defaultColor.B*255) or 255)
            local function push() local c = Color3.fromRGB(R.Get(), G.Get(), B.Get()); preview.BackgroundColor3=c; if cb then safe(cb, c) end end
            R.OnChange(push); G.OnChange(push); B.OnChange(push); push()
            return {Get=function() return preview.BackgroundColor3 end, Set=function(_,c) R.Set(nil, math.floor(c.R*255)); G.Set(nil, math.floor(c.G*255)); B.Set(nil, math.floor(c.B*255)); push() end}
        end

        return tab
    end

    -- Hooks
    local stepSig, inputSig = Signal(), Signal()
    RunService.RenderStepped:Connect(function(dt) stepSig:Fire(dt) end)
    UserInputService.InputBegan:Connect(function(i,g) inputSig:Fire("began", i, g) end)
    UserInputService.InputEnded:Connect(function(i,g) inputSig:Fire("ended", i, g) end)

    -- API
    local api = setmetatable({
        _sg=sg, _win=win, _dock=dock, _icon=icon, _title=titleLbl,
        OnStep  = function(_,fn) return stepSig:Connect(fn) end,
        OnInput = function(_,fn) return inputSig:Connect(fn) end,
        AddTab  = function(_,...) return UIb:AddTab(...) end,
        SetIcon = function(_,id) icon.Image=toAsset(id); dockBtn.Image=toAsset(id) end,
        SetTitle= function(_,t) titleLbl.Text=t end,
        SaveConfig=function(_,name,data) local j=HttpService:JSONEncode(data); if hasfs() then writefile(name..".json", j) end; return j end,
        LoadConfig=function(_,name) if hasfs() and isfile(name..".json") then return HttpService:JSONDecode(readfile(name..".json")) end end,
        SetAccent=function(_,c) Theme.accent=c end,
    }, Nebula)

    -- Startup + notificação
    local overlay = new("Frame",{Size=UDim2.new(1,0,1,0), BackgroundColor3=Color3.new(0,0,0), BackgroundTransparency=0.45}); overlay.Parent=sg
    local box = new("Frame",{Size=UDim2.new(0,360,0,150), Position=UDim2.new(0.5,-180,0.5,-75), BackgroundColor3=Theme.panel2})
    box.Parent=overlay; roundify(box,12); strokify(box,1); pad(box,14)
    local msg = new("TextLabel",{Size=UDim2.new(1,0,0,36), BackgroundTransparency=1, Text="Carregando NebulaUI...", Font=Enum.Font.GothamBold, TextSize=20, TextColor3=Theme.text}); msg.Parent=box
    local sub = new("TextLabel",{Size=UDim2.new(1,0,0,28), Position=UDim2.new(0,0,0,36), BackgroundTransparency=1, Text="discord.gg/tsuo", Font=Enum.Font.Gotham, TextSize=16, TextColor3=Theme.subtext}); sub.Parent=box
    local count = new("TextLabel",{Size=UDim2.new(1,0,1,-64), Position=UDim2.new(0,0,0,64), BackgroundTransparency=1, Text="", Font=Enum.Font.GothamBold, TextSize=38, TextColor3=Theme.accent}); count.Parent=box
    safe(StarterGui.SetCore, StarterGui, "SendNotification", {Title="TSUO Scripts", Text="NebulaUI iniciada — discord.gg/tsuo", Duration=6})
    task.spawn(function() for i=5,1,-1 do count.Text = tostring(i).." ..."; task.wait(1) end; tsv(overlay,{BackgroundTransparency=1},0.25); task.wait(0.25); overlay:Destroy() end)

    return api
end

--// ===== DEMO SEGURA =====
local ICON_ID = "7072718362" -- troque para seu asset id
local UI = Nebula:Create({title="🟣 NEBULA | TSUO Hub", icon=ICON_ID, toggleKey=Enum.KeyCode.RightShift})

-- Tab: Tema & Notificações
local themeTab = UI:AddTab("Tema", "🎨")
themeTab:Section("Aparência do Hub")
local accentPick = themeTab:AddColorPicker("Cor de Acento", Color3.fromRGB(158,99,255), function(c) UI:SetAccent(c) end)
themeTab:AddButton("Notificar TSUO", function()
    safe(StarterGui.SetCore, StarterGui, "SendNotification", {Title="TSUO", Text="discord.gg/tsuo", Duration=5})
end)

-- Tab: Câmera (FOV/Zoom RMB) — recurso visual seguro
local camTab = UI:AddTab("Câmera", "🎥")
camTab:Section("Controles Visuais de Câmera")
local baseFOV = math.clamp(Camera.FieldOfView, 40, 120)
local fovSlider = camTab:AddSlider("FOV da Câmera", 40, 120, math.floor(baseFOV), function(v) Camera.FieldOfView = v end, "")
local zoomToggle = camTab:AddToggle("Zoom com Botão Direito (segurar)", false, function() end)
local zoomValue  = camTab:AddSlider("FOV do Zoom (RMB)", 20, 90, 60, function() end, "")
local functionFOV = fovSlider.Get
UI:OnInput(function(phase, input, gpe)
    if gpe then return end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        if phase=="began" and zoomToggle:Get() then Camera.FieldOfView = zoomValue:Get()
        elseif phase=="ended" and zoomToggle:Get() then Camera.FieldOfView = fovSlider:Get() end
    end
end)

-- Tab: Crosshair (visual na tela — UI somente)
local crossTab = UI:AddTab("Crosshair", "➕")
crossTab:Section("Mira Visual (UI)")
local crossEnabled = crossTab:AddToggle("Mostrar Crosshair", true, function() end)
local sizeSlider = crossTab:AddSlider("Tamanho", 4, 40, 14, function() end, "px")
local gapSlider  = crossTab:AddSlider("Gap", 0, 20, 6, function() end, "px")
local thickSlider= crossTab:AddSlider("Espessura", 1, 8, 2, function() end, "px")
local colorCross = crossTab:AddColorPicker("Cor da Mira", Color3.fromRGB(255,255,255), function() end)

-- Desenha crosshair como 4 frames
local ScreenGui = Instance.new("ScreenGui"); ScreenGui.Name = "NebulaCrosshair"; ScreenGui.ResetOnSpawn=false; ScreenGui.ZIndexBehavior=Enum.ZIndexBehavior.Global; ScreenGui.Parent = CoreGui
local l1 = Instance.new("Frame", ScreenGui); local l2=Instance.new("Frame", ScreenGui); local l3=Instance.new("Frame", ScreenGui); local l4=Instance.new("Frame", ScreenGui)
for _,f in ipairs({l1,l2,l3,l4}) do f.BackgroundColor3 = Color3.new(1,1,1); f.BorderSizePixel=0; end
local function updateCrosshair()
    local sz   = sizeSlider:Get()
    local gap  = gapSlider:Get()
    local th   = thickSlider:Get()
    local col  = colorCross:Get()
    for _,f in ipairs({l1,l2,l3,l4}) do f.BackgroundColor3 = col; f.Visible = crossEnabled:Get() end
    local vx, vy = Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2
    l1.Size = UDim2.new(0, th, 0, sz); l1.Position = UDim2.new(0, vx - th/2, 0, vy - gap - sz) -- up
    l2.Size = UDim2.new(0, th, 0, sz); l2.Position = UDim2.new(0, vx - th/2, 0, vy + gap)     -- down
    l3.Size = UDim2.new(0, sz, 0, th); l3.Position = UDim2.new(0, vx + gap,   0, vy - th/2)   -- right
    l4.Size = UDim2.new(0, sz, 0, th); l4.Position = UDim2.new(0, vx - gap - sz, 0, vy - th/2)-- left
end
UI:OnStep(function() updateCrosshair() end)

-- Tab: HUD (FPS aproximado)
local hudTab = UI:AddTab("HUD", "📊")
hudTab:Section("Indicadores de Sessão")
local fpsLabel = Instance.new("TextLabel"); fpsLabel.BackgroundTransparency=1; fpsLabel.TextColor3=Theme.text; fpsLabel.Font=Enum.Font.Gotham; fpsLabel.TextSize=14
fpsLabel.Size = UDim2.new(0, 140, 0, 22); fpsLabel.Position = UDim2.new(0, 12, 0, 12); fpsLabel.TextXAlignment = Enum.TextXAlignment.Left
local fpsGui = Instance.new("ScreenGui"); fpsGui.Name="NebulaHUD"; fpsGui.ResetOnSpawn=false; fpsGui.ZIndexBehavior=Enum.ZIndexBehavior.Global; fpsGui.Parent=CoreGui
fpsLabel.Parent = fpsGui
local acc, frames = 0, 0
UI:OnStep(function(dt)
    acc += dt; frames += 1
    if acc >= 0.5 then
        local fps = math.floor(frames/acc + 0.5)
        fpsLabel.Text = ("FPS: %d"):format(fps)
        acc, frames = 0, 0
    end
end)

-- Tab: Config
local cfg = UI:AddTab("Config", "⚙️")
cfg:Section("Salvar / Carregar")
cfg:AddButton("Salvar Config", function()
    local data = {
        ui = {accent={r=Theme.accent.R, g=Theme.accent.G, b=Theme.accent.B}},
        cam = {fov = fovSlider:Get(), zoom=zoomValue:Get(), zoomHold=zoomToggle:Get()},
        cross={size=sizeSlider:Get(), gap=gapSlider:Get(), thick=thickSlider:Get(),
               color={r=colorCross:Get().R, g=colorCross:Get().G, b=colorCross:Get().B}, enabled=crossEnabled:Get()}
    }
    UI:SaveConfig("nebula_safe_cfg", data)
    print("[Nebula] Config salva.")
end)
cfg:AddButton("Carregar Config", function()
    local d = UI:LoadConfig("nebula_safe_cfg")
    if d then
        local c = Color3.new(d.ui.accent.r, d.ui.accent.g, d.ui.accent.b); accentPick:Set(c); UI:SetAccent(c)
        fovSlider:Set(d.cam.fov or 70); Camera.FieldOfView = d.cam.fov or 70
        zoomValue:Set(d.cam.zoom or 60); zoomToggle:Set(d.cam.zoomHold or false)
        sizeSlider:Set(d.cross.size or 14); gapSlider:Set(d.cross.gap or 6); thickSlider:Set(d.cross.thick or 2)
        local cc = d.cross.color and Color3.new(d.cross.color.r, d.cross.color.g, d.cross.color.b) or Color3.new(1,1,1); colorCross:Set(cc); crossEnabled:Set(d.cross.enabled ~= false)
        print("[Nebula] Config carregada.")
    else
        warn("[Nebula] Sem config salva.")
    end
end)

-- fim
