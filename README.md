

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- Garantindo que o Delta limpe versões anteriores para evitar lag
if CoreGui:FindFirstChild("TsuoHub_Premium") then
    CoreGui.TsuoHub_Premium:Destroy()
end

local TsuoLib = {
    CurrentTheme = {
        Main = Color3.fromRGB(12, 12, 14),
        Accent = Color3.fromRGB(140, 82, 255),
        Secondary = Color3.fromRGB(20, 20, 24),
        Text = Color3.fromRGB(255, 255, 255),
        Glow = Color3.fromRGB(140, 82, 255)
    }
}

-- [Helper: Animações Suaves]
local function QuickTween(obj, info, goal)
    return TweenService:Create(obj, TweenInfo.new(unpack(info)), goal):Play()
end

function TsuoLib:CreateWindow()
    local theme = self.CurrentTheme
    
    local Screen = Instance.new("ScreenGui")
    Screen.Name = "TsuoHub_Premium"
    Screen.Parent = CoreGui
    Screen.IgnoreGuiInset = true

    -- Glow Externo (Aura)
    local GlowFrame = Instance.new("Frame")
    GlowFrame.Name = "GlowFrame"
    GlowFrame.Size = UDim2.new(0, 580, 0, 380)
    GlowFrame.Position = UDim2.new(0.5, -290, 0.5, -190)
    GlowFrame.BackgroundColor3 = theme.Accent
    GlowFrame.BackgroundTransparency = 0.85
    GlowFrame.Parent = Screen
    
    local GlowCorner = Instance.new("UICorner", GlowFrame)
    GlowCorner.CornerRadius = UDim.new(0, 15)
    
    local Blur = Instance.new("ImageLabel") -- Efeito de profundidade fake
    Blur.Size = UDim2.new(1.1, 0, 1.1, 0)
    Blur.Position = UDim2.new(-0.05, 0, -0.05, 0)
    Blur.BackgroundTransparency = 1
    Blur.Image = "rbxassetid://6015667341" -- Shadow map
    Blur.ImageColor3 = Color3.fromRGB(0,0,0)
    Blur.ImageTransparency = 0.5
    Blur.Parent = GlowFrame

    -- Frame Principal
    local Main = Instance.new("Frame")
    Main.Name = "Main"
    Main.Size = UDim2.new(0, 570, 0, 370)
    Main.Position = UDim2.new(0.5, -285, 0.5, -185)
    Main.BackgroundColor3 = theme.Main
    Main.BorderSizePixel = 0
    Main.Parent = Screen
    
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
    local Stroke = Instance.new("UIStroke", Main)
    Stroke.Color = theme.Accent
    Stroke.Thickness = 1.8
    Stroke.Transparency = 0.3

    -- Sidebar (Menu Lateral)
    local Sidebar = Instance.new("Frame")
    Sidebar.Size = UDim2.new(0, 160, 1, 0)
    Sidebar.BackgroundColor3 = theme.Secondary
    Sidebar.BackgroundTransparency = 0.4
    Sidebar.BorderSizePixel = 0
    Sidebar.Parent = Main
    Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 12)

    -- Título com ID da Logo (Placeholder)
    local Logo = Instance.new("ImageLabel")
    Logo.Size = UDim2.new(0, 40, 0, 40)
    Logo.Position = UDim2.new(0, 15, 0, 15)
    Logo.Image = "rbxassetid://90690292624964" -- Sua ID
    Logo.BackgroundTransparency = 1
    Logo.Parent = Sidebar

    local Title = Instance.new("TextLabel")
    Title.Text = "TSUO HUB"
    Title.Position = UDim2.new(0, 60, 0, 15)
    Title.Size = UDim2.new(0, 100, 0, 40)
    Title.TextColor3 = Color3.fromRGB(255,255,255)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.BackgroundTransparency = 1
    Title.Parent = Sidebar

    -- Container de Abas
    local TabContainer = Instance.new("Frame")
    TabContainer.Position = UDim2.new(0, 10, 0, 70)
    TabContainer.Size = UDim2.new(1, -20, 1, -80)
    TabContainer.BackgroundTransparency = 1
    TabContainer.Parent = Sidebar
    
    local TabList = Instance.new("UIListLayout", TabContainer)
    TabList.Padding = UDim.new(0, 5)

    -- Área de Conteúdo
    local ContentHolder = Instance.new("Frame")
    ContentHolder.Position = UDim2.new(0, 170, 0, 15)
    ContentHolder.Size = UDim2.new(1, -185, 1, -30)
    ContentHolder.BackgroundTransparency = 1
    ContentHolder.Parent = Main

    -- [Funcionalidades]
    local UI = { CurrentTab = nil }

    function UI:CreateTab(name, iconId)
        local TabBtn = Instance.new("TextButton")
        TabBtn.Size = UDim2.new(1, 0, 0, 35)
        TabBtn.BackgroundColor3 = theme.Accent
        TabBtn.BackgroundTransparency = 1
        TabBtn.Text = "    " .. name
        TabBtn.TextColor3 = Color3.fromRGB(150, 150, 150)
        TabBtn.Font = Enum.Font.GothamMedium
        TabBtn.TextSize = 13
        TabBtn.TextXAlignment = Enum.TextXAlignment.Left
        TabBtn.Parent = TabContainer
        Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)

        local Page = Instance.new("ScrollingFrame")
        Page.Size = UDim2.new(1, 0, 1, 0)
        Page.BackgroundTransparency = 1
        Page.Visible = false
        Page.ScrollBarThickness = 0
        Page.Parent = ContentHolder
        Instance.new("UIListLayout", Page).Padding = UDim.new(0, 8)

        TabBtn.MouseButton1Click:Connect(function()
            for _, v in pairs(ContentHolder:GetChildren()) do v.Visible = false end
            for _, v in pairs(TabContainer:GetChildren()) do 
                if v:IsA("TextButton") then
                    TweenService:Create(v, TweenInfo.new(0.3), {BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(150,150,150)}):Play()
                end
            end
            Page.Visible = true
            TweenService:Create(TabBtn, TweenInfo.new(0.3), {BackgroundTransparency = 0.8, TextColor3 = theme.Accent}):Play()
        end)

        local Items = {}

        function Items:AddButton(text, desc, callback)
            local BtnFrame = Instance.new("Frame")
            BtnFrame.Size = UDim2.new(1, -5, 0, 50)
            BtnFrame.BackgroundColor3 = theme.Secondary
            BtnFrame.Parent = Page
            local c = Instance.new("UICorner", BtnFrame)
            local s = Instance.new("UIStroke", BtnFrame)
            s.Color = theme.Accent
            s.Transparency = 0.9

            local T = Instance.new("TextLabel")
            T.Text = text
            T.Size = UDim2.new(1, -60, 0, 30)
            T.Position = UDim2.new(0, 12, 0, 5)
            T.TextColor3 = theme.Text
            T.Font = Enum.Font.GothamBold
            T.TextSize = 14
            T.TextXAlignment = Enum.TextXAlignment.Left
            T.BackgroundTransparency = 1
            T.Parent = BtnFrame

            local D = Instance.new("TextLabel")
            D.Text = desc or "Executa uma função"
            D.Position = UDim2.new(0, 12, 0, 25)
            D.Size = UDim2.new(1, -60, 0, 20)
            D.TextColor3 = Color3.fromRGB(150,150,150)
            D.Font = Enum.Font.Gotham
            D.TextSize = 11
            D.TextXAlignment = Enum.TextXAlignment.Left
            D.BackgroundTransparency = 1
            D.Parent = BtnFrame

            local ExecIcon = Instance.new("ImageButton")
            ExecIcon.Size = UDim2.new(0, 25, 0, 25)
            ExecIcon.Position = UDim2.new(1, -35, 0.5, -12)
            ExecIcon.Image = "rbxassetid://6031094678" -- Play icon
            ExecIcon.ImageColor3 = theme.Accent
            ExecIcon.BackgroundTransparency = 1
            ExecIcon.Parent = BtnFrame

            ExecIcon.MouseButton1Click:Connect(function()
                QuickTween(ExecIcon, {0.1, Enum.EasingStyle.Back}, {Size = UDim2.new(0, 20, 0, 20)})
                task.wait(0.1)
                QuickTween(ExecIcon, {0.1, Enum.EasingStyle.Back}, {Size = UDim2.new(0, 25, 0, 25)})
                pcall(callback)
            end)
        end

        return Items
    end

    -- Sistema de Arrastar (Draggable) Otimizado
    local dragging, dragInput, dragStart, startPos
    Main.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = Main.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            local finalPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            Main.Position = finalPos
            GlowFrame.Position = UDim2.new(finalPos.X.Scale, finalPos.X.Offset - 5, finalPos.Y.Scale, finalPos.Y.Offset - 5)
        end
    end)
    Main.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    return UI
end

-- [EXECUÇÃO]
local Tsuo = TsuoLib:CreateWindow()

local MainTab = Tsuo.CreateTab("Home", 0)
local ScriptTab = Tsuo.CreateTab("Scripts", 0)
local SettingsTab = Tsuo.CreateTab("Configurações", 0)

MainTab:AddButton("Informações", "Usuário: " .. game.Players.LocalPlayer.Name, function()
    print("Log: Usuário verificado.")
end)

ScriptTab:AddButton("Infinite Yield", "O melhor admin script do Roblox.", function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
end)

ScriptTab:AddButton("Dex Explorer", "Explorador de arquivos avançado.", function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/dex.lua"))()
end)

SettingsTab:AddButton("Discord Link", "Copia o convite oficial.", function()
    setclipboard("discord.gg/tsuo")
end)
