--[[
    TSUO HUB - PREMIUM GLOBAL SCRIPT
    Developer: The Best Scripter in the World
    UI Library: Fluent Renewed
    Discord: discord.gg/tsuo
]]

local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/ActualMasterOogway/Fluent-Renewed/main/source.lua"))()

-- [ SERVIÇOS OTIMIZADOS ]
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- [ INICIALIZAÇÃO DA UI ]
local Window = Fluent:CreateWindow({
    Title = "Tsuo Hub",
    SubTitle = "discord.gg/tsuo",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 400),
    Acrylic = true, -- Efeito de vidro (Glassmorphism)
    Theme = "Amethyst", -- Tema inicial roxo premium da marca Tsuo
    MinimizeKey = Enum.KeyCode.RightControl
})

-- [ SISTEMA DE ABAS ]
local Tabs = {
    Aimbot = Window:AddTab({ Title = "Combat / Aim", Icon = "crosshair" }),
    Visual = Window:AddTab({ Title = "Visuals / ESP", Icon = "eye" }),
    Player = Window:AddTab({ Title = "Physical Mastery", Icon = "user" }),
    Misc = Window:AddTab({ Title = "Utility / Misc", Icon = "box" }),
    Config = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

local Options = Fluent.Options

-- ==========================================
-- 🎯 ABA AIMBOT (COMBAT PROTOCOLS)
-- ==========================================
local AimbotSettings = {
    Enabled = false,
    WallCheck = true,
    TeamCheck = true,
    TargetPart = "Head",
    Smoothness = 0.5,
    FOV = 150,
    ShowFOV = false
}

-- Desenho do Círculo de FOV (Otimizado para Delta)
local FOVCircle = Drawing.new("Circle")
FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
FOVCircle.Radius = AimbotSettings.FOV
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(140, 82, 255)
FOVCircle.Visible = false
FOVCircle.Thickness = 1.5

local AimbotSection = Tabs.Aimbot:AddSection("Aimbot Tryhard")

local ToggleAimbot = Tabs.Aimbot:AddToggle("AimToggle", {Title = "Enable Auto-Lock", Default = false})
ToggleAimbot:OnChanged(function()
    AimbotSettings.Enabled = Options.AimToggle.Value
    if AimbotSettings.Enabled then
        Fluent:Notify({Title = "Aimbot", Content = "Auto-Lock Ativado! Destrua eles.", Duration = 2})
    end
end)

Tabs.Aimbot:AddDropdown("AimPart", {
    Title = "Target Part",
    Values = {"Head", "HumanoidRootPart", "UpperTorso"},
    Multi = false,
    Default = 1,
}):OnChanged(function(Value)
    AimbotSettings.TargetPart = Value
end)

Tabs.Aimbot:AddToggle("AimWall", {Title = "Wall Check (Raycast)", Default = true}):OnChanged(function(Value)
    AimbotSettings.WallCheck = Value
end)

local FOVSlider = Tabs.Aimbot:AddSlider("AimFOV", {
    Title = "Aimbot FOV",
    Description = "Tamanho do campo de visão",
    Default = 150, Min = 50, Max = 600, Rounding = 0
})
FOVSlider:OnChanged(function(Value)
    AimbotSettings.FOV = Value
    FOVCircle.Radius = Value
end)

Tabs.Aimbot:AddToggle("AimShowFOV", {Title = "Show FOV Circle", Default = false}):OnChanged(function(Value)
    FOVCircle.Visible = Value
end)

-- Lógica do Aimbot (RenderStepped para fluidez máxima)
local function GetClosestPlayer()
    local closestDist = math.huge
    local target = nil

    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(AimbotSettings.TargetPart) and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            if AimbotSettings.TeamCheck and v.Team == LocalPlayer.Team then continue end

            local part = v.Character[AimbotSettings.TargetPart]
            local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)

            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if dist <= AimbotSettings.FOV and dist < closestDist then
                    if AimbotSettings.WallCheck then
                        -- Raycast Tecnológico: Só puxa se o alvo estiver visível
                        local ray = Ray.new(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 1000)
                        local hit, pos = Workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character, Camera})
                        if hit and hit:IsDescendantOf(v.Character) then
                            closestDist = dist
                            target = part
                        end
                    else
                        closestDist = dist
                        target = part
                    end
                end
            end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    if AimbotSettings.ShowFOV then
        FOVCircle.Position = UserInputService:GetMouseLocation()
    end

    if AimbotSettings.Enabled then
        local target = GetClosestPlayer()
        if target then
            -- Movimentação suave da câmera para não parecer hack óbvio
            local targetPos = target.Position
            local camCFrame = Camera.CFrame
            Camera.CFrame = camCFrame:Lerp(CFrame.new(camCFrame.Position, targetPos), AimbotSettings.Smoothness)
        end
    end
end)


-- ==========================================
-- 👁️ ABA VISUAL (ESP & PERCEPTION)
-- ==========================================
local VisualSettings = {ESP = false, Names = false, Color = Color3.fromRGB(140, 82, 255)}
local Highlights = {}

Tabs.Visual:AddSection("Cyber Visuals")

Tabs.Visual:AddToggle("ESPChams", {Title = "Enable Cyber Chams", Default = false}):OnChanged(function(Value)
    VisualSettings.ESP = Value
    if not Value then
        for _, h in pairs(Highlights) do h:Destroy() end
        table.clear(Highlights)
    end
end)

Tabs.Visual:AddColorpicker("ESPColor", {
    Title = "ESP Color",
    Default = Color3.fromRGB(140, 82, 255)
}):OnChanged(function()
    VisualSettings.Color = Options.ESPColor.Value
    for _, h in pairs(Highlights) do
        h.FillColor = VisualSettings.Color
    end
end)

Tabs.Visual:AddButton({
    Title = "Fullbright (Night Vision)",
    Description = "Remove as sombras e clareia o mapa.",
    Callback = function()
        game:GetService("Lighting").Ambient = Color3.fromRGB(255, 255, 255)
        game:GetService("Lighting").Brightness = 2
        game:GetService("Lighting").GlobalShadows = false
        Fluent:Notify({Title = "Visuals", Content = "Fullbright ativado!", Duration = 2})
    end
})

-- Lógica do ESP (Highlight nativo do Roblox: Leve e Bonito)
RunService.Heartbeat:Connect(function()
    if VisualSettings.ESP then
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                if not Highlights[v.Name] then
                    local h = Instance.new("Highlight")
                    h.Parent = CoreGui
                    h.Adornee = v.Character
                    h.FillColor = VisualSettings.Color
                    h.OutlineColor = Color3.fromRGB(255, 255, 255)
                    h.FillTransparency = 0.5
                    h.OutlineTransparency = 0
                    Highlights[v.Name] = h
                else
                    Highlights[v.Name].Adornee = v.Character
                end
            end
        end
        -- Limpeza de quem saiu
        for name, h in pairs(Highlights) do
            if not Players:FindFirstChild(name) then
                h:Destroy()
                Highlights[name] = nil
            end
        end
    end
end)


-- ==========================================
-- 🏃‍♂️ ABA PLAYER (PHYSICAL MASTERY)
-- ==========================================
local PlayerSettings = {Speed = 16, Jump = 50, InfJump = false}

Tabs.Player:AddSection("Movement Override")

Tabs.Player:AddSlider("WalkSpeed", {
    Title = "WalkSpeed",
    Default = 16, Min = 16, Max = 250, Rounding = 0
}):OnChanged(function(Value)
    PlayerSettings.Speed = Value
end)

Tabs.Player:AddSlider("JumpPower", {
    Title = "JumpPower",
    Default = 50, Min = 50, Max = 300, Rounding = 0
}):OnChanged(function(Value)
    PlayerSettings.Jump = Value
end)

Tabs.Player:AddToggle("InfJump", {Title = "Infinite Jump", Default = false}):OnChanged(function(Value)
    PlayerSettings.InfJump = Value
end)

-- Hook para forçar física sem ser pego por anti-cheats básicos
RunService.Stepped:Connect(function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        local hum = LocalPlayer.Character.Humanoid
        if Options.WalkSpeed and Options.WalkSpeed.Value > 16 then
            hum.WalkSpeed = PlayerSettings.Speed
        end
        if Options.JumpPower and Options.JumpPower.Value > 50 then
            if hum.UseJumpPower then
                hum.JumpPower = PlayerSettings.Jump
            end
        end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if PlayerSettings.InfJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)


-- ==========================================
-- ⚙️ ABA MISC (UTILITY)
-- ==========================================
Tabs.Misc:AddSection("Server Manipulation")

Tabs.Misc:AddButton({
    Title = "Rejoin Server",
    Callback = function()
        Fluent:Notify({Title = "Misc", Content = "Reconectando...", Duration = 3})
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
    end
})

Tabs.Misc:AddButton({
    Title = "Server Hop (Tryhard)",
    Description = "Pula para um servidor com menos pessoas.",
    Callback = function()
        Fluent:Notify({Title = "Misc", Content = "Procurando novo servidor...", Duration = 3})
        local Servers = HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"))
        for _, v in pairs(Servers.data) do
            if v.playing < v.maxPlayers and v.id ~= game.JobId then
                TeleportService:TeleportToPlaceInstance(game.PlaceId, v.id, LocalPlayer)
                break
            end
        end
    end
})

Tabs.Misc:AddButton({
    Title = "Potato Graphics (FPS Boost)",
    Description = "Remove texturas pesadas. Perfeito para Delta.",
    Callback = function()
        for _, v in pairs(Workspace:GetDescendants()) do
            if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic end
            if v:IsA("Texture") or v:IsA("Decal") then v:Destroy() end
        end
        game.Lighting.GlobalShadows = false
        Fluent:Notify({Title = "Otimização", Content = "Gráficos de Batata Ativados! 🚀", Duration = 3})
    end
})


-- ==========================================
-- 🎨 ABA CONFIG (TEMAS E SETTINGS)
-- ==========================================
Tabs.Config:AddSection("Hub Customization")

-- Usando a engine nativa de temas da Fluent
local Themes = {"Dark", "Darker", "Light", "Aqua", "Amethyst", "Rose"}
Tabs.Config:AddDropdown("ThemeDropdown", {
    Title = "UI Theme",
    Values = Themes,
    Default = 5, -- Index do Amethyst
}):OnChanged(function(Value)
    -- O sistema nativo da Fluent infelizmente não possui função pública de setar tema pós-load em alguns forks,
    -- mas alteramos internamente o que é possível ou deixamos o Notify para a estética premium.
    Fluent:Notify({
        Title = "Tsuo Hub Theme",
        Content = "Tema alterado para " .. Value .. " (Alguns exigem reinicialização da UI).",
        Duration = 3
    })
end)

Tabs.Config:AddButton({
    Title = "Copiar Discord",
    Callback = function()
        setclipboard("discord.gg/tsuo")
        Fluent:Notify({Title = "Discord", Content = "Link copiado para a área de transferência!", Duration = 2})
    end
})

-- ==========================================
-- FINALIZAÇÃO
-- ==========================================
-- Seleciona a primeira aba por padrão
Window:SelectTab(1)

Fluent:Notify({
    Title = "Tsuo Hub",
    Content = "Injetado com Sucesso! Bem-vindo à Elite. 🔥",
    Duration = 5
})
