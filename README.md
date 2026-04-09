--[[
    🌟 TSUO HUB v3.0 - ULTIMATE PREMIUM EDITION
    Discord: discord.gg/tsuo
    Design: Glassmorphism 2.0 | Advanced Neon | Cinematic Animations
    Performance: Ultra Optimized | 120FPS | Zero Lag
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- 🎨 SISTEMA DE TEMAS ULTRA PREMIUM (7 Temas Profissionais)
local Themes = {
    Dark = {
        Primary = Color3.fromRGB(12, 12, 22),
        Secondary = Color3.fromRGB(22, 22, 38),
        Accent = Color3.fromRGB(138, 93, 255),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(185, 185, 210),
        Glow = Color3.fromRGB(170, 120, 255),
        Background = Color3.fromRGB(8, 8, 18),
        Glass = Color3.fromRGB(255, 255, 255)
    },
    White = {
        Primary = Color3.fromRGB(248, 248, 255),
        Secondary = Color3.fromRGB(255, 255, 255),
        Accent = Color3.fromRGB(99, 155, 255),
        Text = Color3.fromRGB(35, 35, 48),
        TextSecondary = Color3.fromRGB(95, 95, 115),
        Glow = Color3.fromRGB(130, 185, 255),
        Background = Color3.fromRGB(242, 242, 250),
        Glass = Color3.fromRGB(0, 0, 0)
    },
    Amethyst = {
        Primary = Color3.fromRGB(18, 13, 36),
        Secondary = Color3.fromRGB(38, 28, 72),
        Accent = Color3.fromRGB(193, 133, 255),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(210, 190, 255),
        Glow = Color3.fromRGB(213, 150, 255),
        Background = Color3.fromRGB(13, 8, 31),
        Glass = Color3.fromRGB(255, 255, 255)
    },
    Vulcan = {
        Primary = Color3.fromRGB(28, 16, 16),
        Secondary = Color3.fromRGB(48, 26, 26),
        Accent = Color3.fromRGB(255, 87, 87),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(230, 190, 190),
        Glow = Color3.fromRGB(255, 107, 107),
        Background = Color3.fromRGB(23, 11, 11),
        Glass = Color3.fromRGB(255, 255, 255)
    },
    Neon = {
        Primary = Color3.fromRGB(8, 8, 18),
        Secondary = Color3.fromRGB(18, 18, 36),
        Accent = Color3.fromRGB(0, 255, 204),
        Text = Color3.fromRGB(0, 255, 204),
        TextSecondary = Color3.fromRGB(110, 255, 235),
        Glow = Color3.fromRGB(0, 255, 204),
        Background = Color3.fromRGB(3, 3, 13),
        Glass = Color3.fromRGB(255, 255, 255)
    },
    Rose = {
        Primary = Color3.fromRGB(32, 20, 26),
        Secondary = Color3.fromRGB(52, 37, 47),
        Accent = Color3.fromRGB(255, 157, 187),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(255, 210, 230),
        Glow = Color3.fromRGB(255, 177, 207),
        Background = Color3.fromRGB(27, 15, 21),
        Glass = Color3.fromRGB(255, 255, 255)
    },
    Ruby = {
        Primary = Color3.fromRGB(22, 10, 15),
        Secondary = Color3.fromRGB(42, 20, 25),
        Accent = Color3.fromRGB(255, 67, 107),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(255, 187, 217),
        Glow = Color3.fromRGB(255, 87, 127),
        Background = Color3.fromRGB(17, 5, 10),
        Glass = Color3.fromRGB(255, 255, 255)
    }
}

local CurrentTheme = Themes.Amethyst
local HubOpen = true
local Dragging = false
local DragStart = nil
local StartPos = nil
local CurrentTabContent = nil

-- 🔧 NOTIFICAÇÕES PREMIUM (Cinematic Toasts)
local NotificationContainer = Instance.new("Frame")
NotificationContainer.Name = "TsuoNotifications"
NotificationContainer.Size = UDim2.new(0, 400, 1, 0)
NotificationContainer.Position = UDim2.new(0, 30, 0, 30)
NotificationContainer.BackgroundTransparency = 1
NotificationContainer.Parent = PlayerGui

local function premiumNotify(title, message, icon, duration)
    local toast = Instance.new("Frame")
    toast.Size = UDim2.new(1, 0, 0, 80)
    toast.Position = UDim2.new(1, 20, 0, 0)
    toast.BackgroundColor3 = CurrentTheme.Secondary
    toast.BackgroundTransparency = 0.2
    toast.BorderSizePixel = 0
    toast.Parent = NotificationContainer
    
    local toastCorner = Instance.new("UICorner")
    toastCorner.CornerRadius = UDim.new(0, 16)
    toastCorner.Parent = toast
    
    local toastGradient = Instance.new("UIGradient")
    toastGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, CurrentTheme.Primary),
        ColorSequenceKeypoint.new(1, CurrentTheme.Secondary)
    }
    toastGradient.Rotation = 90
    toastGradient.Parent = toast
    
    local glow = Instance.new("UIStroke")
    glow.Color = CurrentTheme.Glow
    glow.Thickness = 2
    glow.Transparency = 0.3
    glow.Parent = toast
    
    -- Icone
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(0, 50, 0, 50)
    iconLabel.Position = UDim2.new(0, 15, 0.5, -25)
    iconLabel.BackgroundTransparency = 1
    iconLabel.Text = icon or "🌟"
    iconLabel.TextColor3 = CurrentTheme.Accent
    iconLabel.TextScaled = true
    iconLabel.Font = Enum.Font.FredokaOne
    iconLabel.Parent = toast
    
    -- Conteudo
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -90, 0.4, 0)
    titleLabel.Position = UDim2.new(0, 80, 0, 10)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = CurrentTheme.Text
    titleLabel.TextScaled = true
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = toast
    
    local messageLabel = Instance.new("TextLabel")
    messageLabel.Size = UDim2.new(1, -90, 0.4, 0)
    messageLabel.Position = UDim2.new(0, 80, 0.5, 5)
    messageLabel.BackgroundTransparency = 1
    messageLabel.Text = message
    messageLabel.TextColor3 = CurrentTheme.TextSecondary
    messageLabel.TextScaled = true
    messageLabel.Font = Enum.Font.Gotham
    messageLabel.TextXAlignment = Enum.TextXAlignment.Left
    messageLabel.Parent = toast
    
    -- Animações Cinematic
    TweenService:Create(toast, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0, 0, 0, 0),
        BackgroundTransparency = 0.1
    }):Play()
    
    task.wait(duration or 4)
    TweenService:Create(toast, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
        Position = UDim2.new(1, 20, 0, 0),
        BackgroundTransparency = 1
    }):Play()
    task.wait(0.4)
    toast:Destroy()
end

-- 🎨 UTILITÁRIOS VISUAIS PREMIUM
local function createPremiumGlow(parent, color, thickness)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or CurrentTheme.Glow
    stroke.Thickness = thickness or 2.5
    stroke.Transparency = 0.35
    stroke.Parent = parent
    
    -- Pulsing glow effect
    spawn(function()
        while parent.Parent do
            TweenService:Create(stroke, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
                Transparency = 0.1
            }):Play()
            task.wait(2)
        end
    end)
    
    return stroke
end

local function createGlassEffect(parent)
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(220, 220, 255))
    }
    gradient.Rotation = 45
    gradient.Transparency = NumberSequence.new{
        NumberSequenceKeypoint.new(0, 0.9),
        NumberSequenceKeypoint.new(0.5, 0.7),
        NumberSequenceKeypoint.new(1, 0.9)
    }
    gradient.Parent = parent
end

local function advancedRipple(button)
    local ripple = Instance.new("Frame")
    ripple.Size = UDim2.new(0, 0, 0, 0)
    ripple.AnchorPoint = Vector2.new(0.5, 0.5)
    ripple.Position = UDim2.new(0.5, 0, 0.5, 0)
    ripple.BackgroundColor3 = CurrentTheme.Glow
    ripple.BackgroundTransparency = 0.6
    ripple.BorderSizePixel = 0
    ripple.ZIndex = button.ZIndex + 2
    ripple.Parent = button
    
    local rippleCorner = Instance.new("UICorner")
    rippleCorner.CornerRadius = UDim.new(1, 0)
    rippleCorner.Parent = ripple
    
    TweenService:Create(ripple, TweenInfo.new(0.7, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Size = UDim2.new(3, 0, 3, 0),
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.7)
    ripple:Destroy()
end

-- 🏗️ HUB PRINCIPAL (DESIGN ULTRA PREMIUM)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TsuoHubPremium"
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 1000
ScreenGui.Parent = PlayerGui

-- Main Frame com Glassmorphism Avançado
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 720, 0, 550)
MainFrame.Position = UDim2.new(0.5, -360, 0.5, -275)
MainFrame.BackgroundColor3 = CurrentTheme.Primary
MainFrame.BackgroundTransparency = 0.08
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 24)
MainCorner.Parent = MainFrame

createPremiumGlow(MainFrame, CurrentTheme.Glow, 4)
createGlassEffect(MainFrame)

-- HEADER CINEMATIC
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, 0, 0, 85)
Header.Position = UDim2.new(0, 0, 0, 0)
Header.BackgroundColor3 = CurrentTheme.Background
Header.BackgroundTransparency = 0.05
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 24)
HeaderCorner.Parent = Header

createPremiumGlow(Header, CurrentTheme.Glow, 3)
createGlassEffect(Header)

-- LOGO ANIMADA
local Logo = Instance.new("TextLabel")
Logo.Size = UDim2.new(0, 220, 1, 0)
Logo.Position = UDim2.new(0, 30, 0, 0)
Logo.BackgroundTransparency = 1
Logo.Text = "✨ TSUO HUB v3.0"
Logo.TextColor3 = CurrentTheme.Accent
Logo.TextScaled = true
Logo.Font = Enum.Font.FredokaOne
Logo.Parent = Header

-- Animação da logo
TweenService:Create(Logo, TweenInfo.new(1.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out, 0, true), {
    TextTransparency = 0
}):Play()

-- DRAG SYSTEM PREMIUM
local function updateInput(input)
    local delta = input.Position - DragStart
    MainFrame.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset + delta.X, StartPos.Y.Scale, StartPos.Y.Offset + delta.Y)
end

Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragStart = input.Position
        StartPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and Dragging then
        updateInput(input)
    end
end)

-- BOTÕES HEADER (MICROINTERAÇÕES)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 60, 0, 60)
ToggleButton.Position = UDim2.new(1, -90, 0, 12.5)
ToggleButton.BackgroundColor3 = CurrentTheme.Accent
ToggleButton.BackgroundTransparency = 0.1
ToggleButton.BorderSizePixel = 0
ToggleButton.Text = "⬇️"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextScaled = true
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = Header

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 30)
ToggleCorner.Parent = ToggleButton

createPremiumGlow(ToggleButton)

-- Config Button
local ConfigButton = Instance.new("TextButton")
ConfigButton.Size = UDim2.new(0, 60, 0, 60)
ConfigButton.Position = UDim2.new(1, -165, 0, 12.5)
ConfigButton.BackgroundColor3 = CurrentTheme.Secondary
ConfigButton.BackgroundTransparency = 0.2
ConfigButton.BorderSizePixel = 0
ConfigButton.Text = "🎨"
ConfigButton.TextColor3 = CurrentTheme.Text
ConfigButton.TextScaled =
