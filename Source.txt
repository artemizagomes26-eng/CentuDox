--[[
    Script Completo para Blox Fruits com Menu GUI
    Funções:
    - Auto Farm, Auto Quest, Auto Stats, Auto Haki, Auto Buy, ESP, Auto Raid, Anti AFK
    - Menu interativo para controlar tudo
]]

-- Configurações iniciais (podem ser alteradas pelo menu)
local Config = {
    AutoFarm = true,
    AutoQuest = true,
    AutoStats = true,
    StatPriority = "Melee",
    AutoHaki = true,
    AutoBuy = false,
    BuyItem = "Sword",
    ESP = true,
    TeleportTo = "Start",
    AutoRaid = true,
    RaidType = "Flame",
    AntiAFK = true,
    FarmDistance = 50,
}

-- Serviços
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")

-- Variáveis
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Funções Auxiliares (as mesmas do script anterior, omitidas para brevidade, mas presentes no código completo)
-- ... (copie as funções anteriores: Notify, GetNearestEnemy, AttackEnemy, TeleportToIsland, AcceptQuest, CompleteQuest, AutoStats, AutoHaki, AutoBuyItem, ESP, StartRaid, AutoRaidLoop, AntiAFK)

-- ================== CRIAÇÃO DO MENU ==================
local function CreateMenu()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "BloxFruitsMenu"
    ScreenGui.Parent = Player:WaitForChild("PlayerGui")
    ScreenGui.ResetOnSpawn = false

    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Parent = ScreenGui
    MainFrame.Size = UDim2.new(0, 250, 0, 400)
    MainFrame.Position = UDim2.new(0.5, -125, 0.5, -200)
    MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.ClipsDescendants = true

    -- Título
    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Parent = MainFrame
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Title.Text = "Blox Fruits Script"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.SourceSansBold
    Title.TextSize = 18

    -- Botão de minimizar
    local MinimizeButton = Instance.new("TextButton")
    MinimizeButton.Parent = MainFrame
    MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
    MinimizeButton.Position = UDim2.new(1, -30, 0, 0)
    MinimizeButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    MinimizeButton.Text = "-"
    MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeButton.Font = Enum.Font.SourceSansBold
    MinimizeButton.TextSize = 20
    MinimizeButton.AutoButtonColor = false
    MinimizeButton.MouseButton1Click:Connect(function()
        local scroll = MainFrame:FindFirstChild("ScrollFrame")
        if scroll then
            scroll.Visible = not scroll.Visible
            MainFrame.Size = scroll.Visible and UDim2.new(0, 250, 0, 400) or UDim2.new(0, 250, 0, 30)
        end
    end)

    -- ScrollFrame para os botões
    local ScrollFrame = Instance.new("ScrollingFrame")
    ScrollFrame.Name = "ScrollFrame"
    ScrollFrame.Parent = MainFrame
    ScrollFrame.Size = UDim2.new(1, 0, 1, -30)
    ScrollFrame.Position = UDim2.new(0, 0, 0, 30)
    ScrollFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    ScrollFrame.BorderSizePixel = 0
    ScrollFrame.ScrollBarThickness = 5
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 600)

    local function AddToggle(yPos, text, default, callback)
        local Toggle = Instance.new("TextButton")
        Toggle.Name = text
        Toggle.Parent = ScrollFrame
        Toggle.Size = UDim2.new(1, -20, 0, 30)
        Toggle.Position = UDim2.new(0, 10, 0, yPos)
        Toggle.BackgroundColor3 = default and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        Toggle.Text = text
        Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        Toggle.Font = Enum.Font.SourceSans
        Toggle.TextSize = 14
        Toggle.AutoButtonColor = false
        Toggle.MouseButton1Click:Connect(function()
            local newState = not (Toggle.BackgroundColor3 == Color3.fromRGB(0, 170, 0))
            Toggle.BackgroundColor3 = newState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
            callback(newState)
        end)
    end

    local function AddCycle(yPos, text, options, currentIndex, callback)
        local CycleButton = Instance.new("TextButton")
        CycleButton.Name = text
        CycleButton.Parent = ScrollFrame
        CycleButton.Size = UDim2.new(1, -20, 0, 30)
        CycleButton.Position = UDim2.new(0, 10, 0, yPos)
        CycleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        CycleButton.Text = text .. ": " .. options[currentIndex]
        CycleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        CycleButton.Font = Enum.Font.SourceSans
        CycleButton.TextSize = 14
        CycleButton.AutoButtonColor = false
        local current = currentIndex
        CycleButton.MouseButton1Click:Connect(function()
            current = current % #options + 1
            CycleButton.Text = text .. ": " .. options[current]
            callback(options[current])
        end)
    end

    -- Adicionar toggles
    AddToggle(10, "Auto Farm", Config.AutoFarm, function(state) Config.AutoFarm = state end)
    AddToggle(45, "Auto Quest", Config.AutoQuest, function(state) Config.AutoQuest = state end)
    AddToggle(80, "Auto Stats", Config.AutoStats, function(state) Config.AutoStats = state end)
    AddToggle(115, "Auto Haki", Config.AutoHaki, function(state) Config.AutoHaki = state end)
    AddToggle(150, "Auto Buy", Config.AutoBuy, function(state) Config.AutoBuy = state end)
    AddToggle(185, "ESP", Config.ESP, function(state) Config.ESP = state end)
    AddToggle(220, "Auto Raid", Config.AutoRaid, function(state) Config.AutoRaid = state end)
    AddToggle(255, "Anti AFK", Config.AntiAFK, function(state) Config.AntiAFK = state end)

    -- Ciclos para opções
    local statOptions = {"Melee", "Defense", "Sword", "Gun", "Blox Fruit"}
    local statIndex = table.find(statOptions, Config.StatPriority) or 1
    AddCycle(290, "Stat Priority", statOptions, statIndex, function(value) Config.StatPriority = value end)

    local raidOptions = {"Flame", "Ice", "Quake", "Dark", "Light", "String", "Rumble", "Magma", "Buddha", "Sand"}
    local raidIndex = table.find(raidOptions, Config.RaidType) or 1
    AddCycle(325, "Raid Type", raidOptions, raidIndex, function(value) Config.RaidType = value end)

    local teleportOptions = {"Start", "Jungle", "Pirate Village", "Desert", "Frozen Village", "Marine Fortress", "Skylands", "Prison", "Colosseum", "Magma Village", "Underwater City", "Fountain City", "Shank's Room", "Mob Island", "Raid Island", "Second Sea", "Third Sea"}
    local teleportIndex = table.find(teleportOptions, Config.TeleportTo) or 1
    AddCycle(360, "Teleport To", teleportOptions, teleportIndex, function(value) Config.TeleportTo = value end)
end

-- Criar menu
CreateMenu()

-- Loop Principal (usa Config, que agora é controlado pelo menu)
RunService.RenderStepped:Connect(function()
    if Config.AntiAFK then
        AntiAFK()
    end

    if Config.AutoFarm then
        local enemy = GetNearestEnemy(Config.FarmDistance)
        if enemy then
            local distance = (RootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
            if distance > 10 then
                Humanoid:MoveTo(enemy.HumanoidRootPart.Position)
            else
                AttackEnemy(enemy)
            end
        else
            local farEnemy = GetNearestEnemy(500)
            if farEnemy then
                Humanoid:MoveTo(farEnemy.HumanoidRootPart.Position)
            end
        end
    end

    if Config.AutoQuest then
        AcceptQuest()
        CompleteQuest()
    end

    if Config.AutoStats and Player.Data and Player.Data.Points > 0 then
        AutoStats()
    end

    if Config.AutoHaki then
        AutoHaki()
    end

    if Config.AutoBuy then
        AutoBuyItem()
    end

    if Config.ESP then
        ESP()
    end

    if Config.AutoRaid and not _G.RaidRunning then
        _G.RaidRunning = true
        spawn(function()
            AutoRaidLoop()
            wait(5)
            _G.RaidRunning = false
        end)
    end
end)

-- Teleporte manual (pressione "T" para teleportar para a ilha configurada)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.T then
        TeleportToIsland(Config.TeleportTo)
    end
end)

Notify("Script Carregado", "Menu criado! Use o menu para controlar as funções.")
