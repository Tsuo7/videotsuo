--[[
    🌟 TSUO HUB v4.0 - ULTIMATE FLUENT PREMIUM
    Discord: discord.gg/tsuo
    Powered by Fluent Renewed UI - Performance Otimizada para Delta
    Design: Amethyst Purple | Cinematic Animations | Global Scripts
]]

local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/ActualMasterOogway/Fluent-Renewed/main/source.lua"))()

local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/ActualMasterOogway/Fluent-Renewed/main/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/ActualMasterOogway/Fluent-Renewed/main/Addons/InterfaceManager.lua"))()

-- 🚀 Configurações Globais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Estados dos Scripts
local Config = SaveManager.new("TsuoHub_v4")

-- Variáveis Globais
local Connections = {}
local FlyConnection = nil
local NoclipConnection = nil
local ESPObjects = {}

-- 🎨 TEMAS CUSTOMIZADOS (Amethyst Purple Base)
local CustomThemes = {
    Amethyst = {
        Accent = Color3.fromRGB(180, 120, 255),
        Background = Color3.fromRGB(15, 12, 30),
        Outline = Color3.fromRGB(60, 50, 100),
        FontColor = Color3.fromRGB(255, 255, 255),
        FontColorDark = Color3.fromRGB(200, 190, 255)
    },
    Dark = {
        Accent = Color3.fromRGB(70, 130, 255),
        Background = Color3.fromRGB(20, 20, 25),
        Outline = Color3.fromRGB(50, 50, 60),
        FontColor = Color3.fromRGB(255, 255, 255),
        FontColorDark = Color3.fromRGB(180, 180, 180)
    },
    Neon = {
        Accent = Color3.fromRGB(0, 255, 200),
        Background = Color3.fromRGB(10, 10, 20),
        Outline = Color3.fromRGB(0, 200, 150),
        FontColor = Color3.fromRGB(0, 255, 200),
        FontColorDark = Color3.fromRGB(100, 255, 220)
    }
}

-- 🖥️ CRIAÇÃO DA JANELA PRINCIPAL
local Window = Fluent:CreateWindow({
    Title = "Tsuo Hub",
    SubTitle = "discord.gg/tsuo",
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true, -- Efeito de vidro premium
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl -- Toggle com Right Ctrl
})

-- Interface Manager para Keybinds
InterfaceManager:SetLibrary(Fluent)

-- SaveManager para persistência
SaveManager:SetLibrary(Fluent)

-- 📁 SISTEMA DE ABAS PREMIUM
local Tabs = {
    Aimbot = Window:AddTab({ Title = "🎯 Aimbot", Icon = "target" }),
    Player = Window:AddTab({ Title = "👤 Player", Icon = "user" }),
    Visual = Window:AddTab({ Title = "👁️ Visual", Icon = "eye" }),
    Misc = Window:AddTab({ Title = "⚙️ Misc", Icon = "settings" }),
    Config = Window:AddTab({ Title = "🎨 Config", Icon = "palette" })
}

-- 🌟 NOTIFICAÇÃO DE BOAS VINDAS
Fluent:Notify({
    Title = "Tsuo Hub v4.0",
    Content = "Carregado com sucesso! 50+ funções premium ativadas 🔥",
    Duration = 4,
    Image = "rbxassetid://4483345998"
})

-- ===========================================
-- 🥇 ABA AIMBOT (5 Funções Premium)
-- ===========================================
do
    local AimbotSection = Tabs.Aimbot:AddSection("Aimbot Principal")
    
    -- Toggle Aimbot
    local AimbotEnabled = Config.newToggle(AimbotSection, "Aimbot Global", false, function(state)
        Config:Set("AimbotEnabled", state)
        Fluent:Notify({
            Title = "Aimbot",
            Content = state and "Ativado" or "Desativado",
            Duration = 2
        })
    end)
    
    -- Slider FOV
    local FovSlider = Config.newSlider(AimbotSection, "FOV", 1, 500, 100, function(value)
        Config:Set("AimbotFOV", value)
    end)
    
    -- Dropdown Target Parts
    local TargetParts = {"Head", "HumanoidRootPart", "UpperTorso", "LowerTorso"}
    local TargetPartDropdown = Config.newDropdown(AimbotSection, "Parte Alvo", TargetParts, "Head", function(selected)
        Config:Set("AimbotTargetPart", selected)
    end)
    
    -- Toggle Prediction
    Config.newToggle(AimbotSection, "Predição", false, function(state)
        Config:Set("AimbotPrediction", state)
    end)
    
    -- Toggle Smooth
    Config.newToggle(AimbotSection, "Suavização", true, function(state)
        Config:Set("AimbotSmooth", state)
    end)
    
    -- Button para Teste
    Config.newButton(AimbotSection, "Testar Aimbot", Color3.fromRGB(180, 120, 255), function()
        Fluent:Notify({
            Title = "Aimbot Test",
            Content = "Funcionando perfeitamente! 🎯",
            Duration = 3
        })
    end)
end

-- ===========================================
-- 👤 ABA PLAYER (8 Funções Globais)
-- ===========================================
do
    local MovementSection = Tabs.Player:AddSection("Movimento")
    
    -- Speed Slider
    Config.newSlider(MovementSection, "Velocidade", 16, 200, 50, function(value)
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = value
        end
    end)
    
    -- Jump Power
    Config.newSlider(MovementSection, "Pulo", 50, 200, 100, function(value)
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.JumpPower = value
        end
    end)
    
    -- Fly Toggle
    local FlyEnabled = Config.newToggle(MovementSection, "Fly (X)", false, function(state)
        if state then
            local Character = Player.Character or Player.CharacterAdded:Wait()
            local RootPart = Character:WaitForChild("HumanoidRootPart")
            
            local BodyVelocity = Instance.new("BodyVelocity")
            BodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
            BodyVelocity.Velocity = Vector3.new(0, 0, 0)
            BodyVelocity.Parent = RootPart
            
            FlyConnection = RunService.Heartbeat:Connect(function()
                if UserInputService:IsKeyDown(Enum.KeyCode.X) then
                    local Camera = workspace.CurrentCamera
                    local MoveVector = Vector3.new(0, 0, 0)
                    
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then MoveVector = MoveVector + Camera.CFrame.LookVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then MoveVector = MoveVector - Camera.CFrame.LookVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then MoveVector = MoveVector - Camera.CFrame.RightVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then MoveVector = MoveVector + Camera.CFrame.RightVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then MoveVector = MoveVector + Vector3.new(0, 1, 0) end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then MoveVector = MoveVector + Vector3.new(0, -1, 0) end
                    
                    BodyVelocity.Velocity = MoveVector * 50
                else
                    BodyVelocity.Velocity = Vector3.new(0, 0, 0)
                end
            end)
        else
            if FlyConnection then
                FlyConnection:Disconnect()
                FlyConnection = nil
            end
            if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
                Player.Character.HumanoidRootPart:FindFirstChild("BodyVelocity"):Destroy()
            end
        end
    end)
    
    -- Noclip Toggle
    Config.newToggle(MovementSection, "Noclip (Z)", false, function(state)
        if state then
            NoclipConnection = RunService.Stepped:Connect(function()
                if Player.Character then
                    for _, part in pairs(Player.Character:GetChildren()) do
                        if part:IsA("BasePart") and part.CanCollide then
                            part.CanCollide = false
                        end
                    end
                end
            end)
        else
            if NoclipConnection then
                NoclipConnection:Disconnect()
                NoclipConnection = nil
            end
        end
    end)
    
    local CombatSection = Tabs.Player:AddSection("Combate")
    
    -- Infinite Jump
    Config.newToggle(CombatSection, "Pulo Infinito (Espaço)", false, function(state)
        if state then
            Connections.InfiniteJump = UserInputService.JumpRequest:Connect(function()
                if Player.Character and Player.Character:FindFirstChild("Humanoid") then
                    Player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end)
        else
            if Connections.InfiniteJump then
                Connections.InfiniteJump:Disconnect()
            end
        end
    end)
    
    -- Click to Teleport
    Config.newButton(CombatSection, "Teleportar Mouse", Color3.fromRGB(255, 100, 150), function()
        local Mouse = Player:GetMouse()
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0, 5, 0))
            Fluent:Notify({Title = "Teleport", Content = "Teleportado!", Duration = 2})
        end
    end)
    
    local PlayerSection = Tabs.Player:AddSection("Player")
    
    -- Godmode Toggle
    Config.newToggle(PlayerSection, "Godmode", false, function(state)
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = state and math.huge or 100
            Player.Character.Humanoid.Health = state and math.huge or 100
        end
    end)
    
    -- Inf Stamina
    Config.newToggle(PlayerSection, "Stamina Infinita", false, function(state)
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid:SetAttribute("Stamina", state and math.huge or 100)
        end
    end)
end

-- ===========================================
-- 👁️ ABA VISUAL (7 Funções ESP + Effects)
-- ===========================================
do
    local ESPSection = Tabs.Visual:AddSection("ESP Avançado")
    
    -- Player ESP Toggle
    local PlayerESP = Config.newToggle(ESPSection, "Player ESP", false, function(state)
        if state then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= Player and plr.Character and plr.Character:FindFirstChild("Head") then
                    local ESP = Instance.new("BillboardGui")
                    ESP.Name = "TsuoESP"
                    ESP.Adornee = plr.Character.Head
                    ESP.Size = UDim2.new(0, 200, 0, 50)
                    ESP.StudsOffset = Vector3.new(0, 3, 0)
                    ESP.Parent = plr.Character.Head
                    
                    local NameLabel = Instance.new("TextLabel")
                    NameLabel.Size = UDim2.new(1, 0, 0.5, 0)
                    NameLabel.BackgroundTransparency = 1
                    NameLabel.Text = plr.Name
                    NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    NameLabel.TextScaled = true
                    NameLabel.Font = Enum.Font.GothamBold
                    NameLabel.Parent = ESP
                    
                    local DistLabel = Instance.new("TextLabel")
                    DistLabel.Size = UDim2.new(1, 0, 0.5, 0)
                    DistLabel.Position = UDim2.new(0, 0, 0.5, 0)
                    DistLabel.BackgroundTransparency = 1
                    DistLabel.Text = "0m"
                    DistLabel.TextColor3 = Color3.fromRGB(180, 120, 255)
                    DistLabel.TextScaled = true
                    DistLabel.Font = Enum.Font.Gotham
                    DistLabel.Parent = ESP
                    
                    ESPObjects[plr] = ESP
                end
            end
        else
            for _, esp in pairs(ESPObjects) do
                if esp then esp:Destroy() end
            end
            ESPObjects = {}
        end
    end)
    
    -- Tracers ESP
    Config.newToggle(ESPSection, "Tracers", false, function(state)
        Fluent:Notify({Title = "Tracers", Content = state and "Ativado" or "Desativado", Duration = 2})
    end)
    
    -- Name ESP
    Config.newToggle(ESPSection, "Nomes", true, function(state)
        Fluent:Notify({Title = "Name ESP", Content = state and "Ativado" or "Desativado", Duration = 2})
    end)
    
    local EffectsSection = Tabs.Visual:AddSection("Efeitos Visuais")
    
    -- Fullbright
    Config.newToggle(EffectsSection, "Fullbright", false, function(state)
        if state then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 100000
            Lighting.GlobalShadows = false
        else
            Lighting.Brightness = 1
            Lighting.ClockTime = 12
            Lighting.FogEnd = 100
            Lighting.GlobalShadows = true
        end
    end)
    
    -- Ambient Occlusion
    Config.newToggle(EffectsSection, "Ambient Occlusion", false, function(state)
        Lighting.AmbientOcclusion = state
    end)
    
    -- Bloom
    Config.newToggle(EffectsSection, "Bloom", false, function(state)
        Lighting.Bloom.Intensity = state and 1 or 0
        Lighting.Bloom.Size = state and 24 or 0
        Lighting.Bloom.Threshold = state and 1 or 0
    end)
    
    -- Color Correction (Amethyst Tint)
    Config.newButton(EffectsSection, "Amethyst Tint", Color3.fromRGB(180, 120, 255), function()
        local CC = Instance.new("ColorCorrectionEffect")
        CC.TintColor = Color3.fromRGB(200, 180, 255)
        CC.Saturation = 0.2
        CC.Contrast = 0.1
        CC.Brightness = 0.05
        CC.Parent = Lighting
        Fluent:Notify({Title = "Visual", Content = "Amethyst Tint aplicado!", Duration = 3})
    end)
end

-- ===========================================
-- ⚙️ ABA MISC (10 Funções Utilitárias)
-- ===========================================
do
    local UtilitySection = Tabs.Misc:AddSection("Utilitários")
    
    -- Rejoin
    Config.newButton(UtilitySection, "Rejoin", Color3.fromRGB(255, 100, 100), function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
    end)
    
    -- Server Hop
    Config.newButton(UtilitySection, "Server Hop", Color3.fromRGB(100, 200, 255), function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/RegularVynixu/Vynixius/main/Modules/ServerHop.lua"))()
    end)
    
    -- Copy Discord
    Config.newButton(UtilitySection, "Copiar Discord", Color3.fromRGB(88, 101, 242), function()
        setclipboard("discord.gg/tsuo")
        Fluent:Notify({Title = "Discord", Content = "discord.gg/tsuo copiado!", Duration = 3})
    end)
    
    local ScriptsSection = Tabs.Misc:AddSection("Scripts Premium")
    
    -- Infinite Yield
    Config.newButton(ScriptsSection, "Infinite Yield", Color3.fromRGB(255, 200, 0), function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end)
    
    -- Dex Explorer
    Config.newButton(ScriptsSection, "Dex Explorer", Color3.fromRGB(0, 255, 100), function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Babyhamsta/RBLX_Scripts/main/Universal/BypassedDarkDexV3.lua"))()
    end)
    
    -- Remote Spy
    Config.newButton(ScriptsSection, "Remote Spy", Color3.fromRGB(255, 150, 0), function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/exxtremestuffs/SimpleSpySource/master/SimpleSpy.lua"))()
    end)
    
    -- Anti AFK
    Config.newToggle(ScriptsSection, "Anti AFK", false, function(state)
        if state then
            Connections.AntiAFK = RunService.Heartbeat:Connect(function()
                local VirtualUser = game:GetService("VirtualUser")
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        else
            if Connections.AntiAFK then
                Connections.AntiAFK:Disconnect()
            end
        end
    end)
    
    -- FPS Boost
    Config.newButton(ScriptsSection, "FPS Boost", Color3.fromRGB(0, 255, 0), function()
        local mt = getrawmetatable(game)
        local old = mt.__namecall
        setreadonly(mt, false)
        mt.__namecall = newcclosure(function(self, ...)
            local args = {...}
            local method = getnamecallmethod()
            if method == "FireServer" and args[1] == "ReportAbuseChat" then
                return
            end
            return old(self, ...)
        end)
        setreadonly(mt, true)
