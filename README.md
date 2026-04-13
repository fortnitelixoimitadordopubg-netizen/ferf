--[[
    Bridger: WESTERN - Auto Fishing & ESP (Fixed Version)
--]]

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager") -- Para simular teclas
local LocalPlayer = Players.LocalPlayer

-- Estado dos sistemas
local autoFishing = false
local autoMinigame = false
local espPlayers = false
local espBodyParts = false
local fishingActive = false

-- Configurações
local config = {
    fishingDelay = 0.1,
    minigameSpeed = 0.05,
    espColor = Color3.fromRGB(0, 255, 0)
}

local bodyColors = {
    ["LeftArm"] = Color3.fromRGB(255, 50, 50),
    ["RightArm"] = Color3.fromRGB(50, 255, 50),
    ["LeftLeg"] = Color3.fromRGB(50, 50, 255),
    ["RightLeg"] = Color3.fromRGB(255, 255, 50),
    ["Rib Cage"] = Color3.fromRGB(255, 50, 255)
}

-- ==================== FUNÇÕES DE SUPORTE ====================

local function pressKey(keyCode)
    VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
    task.wait(0.05)
    VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
end

-- Encontrar a boia (Bobber)
local function findBobber()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
    
    -- Busca mais específica para evitar lag
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name:lower():find("bobber") or obj.Name:lower():find("floater") then
            return obj
        end
    end
    return nil
end

local function detectBite(bobber)
    local root = bobber:FindFirstChild("RootPart") or bobber:FindFirstChild("Handle") or bobber:FindFirstChildWhichIsA("BasePart")
    if root and root.AssemblyLinearVelocity.Magnitude > 1.5 then
        return true
    end
    
    for _, child in ipairs(bobber:GetDescendants()) do
        if child:IsA("ParticleEmitter") and child.Enabled then
            return true
        end
    end
    return false
end

-- ==================== SISTEMA DE PESCA ====================

local function castLine()
    local char = LocalPlayer.Character
    local tool = char and char:FindFirstChildOfClass("Tool")
    if tool then
        tool:Activate() -- Simula o clique para lançar
        Rayfield:Notify({Title = "Pesca", Content = "Linha lançada!", Duration = 2})
    end
end

task.spawn(function()
    while true do
        if autoFishing then
            local bobber = findBobber()
            
            if not bobber and not fishingActive then
                castLine()
                fishingActive = true
                task.wait(2)
            elseif bobber then
                if detectBite(bobber) then
                    pressKey(Enum.KeyCode.E)
                    fishingActive = false
                    task.wait(1)
                end
            end
        end
        task.wait(config.fishingDelay)
    end
end)

-- ==================== AUTO MINIGAME ====================

local function solveMinigame()
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return end

    -- Tenta encontrar textos de teclas (R, F, T, G)
    for _, gui in ipairs(playerGui:GetDescendants()) do
        if gui:IsA("TextLabel") or gui:IsA("TextButton") then
            local t = gui.Text:upper()
            if t == "R" or t == "F" or t == "T" or t == "G" then
                pressKey(Enum.KeyCode[t])
            end
        end
    end
end

task.spawn(function()
    while true do
        if autoMinigame then
            solveMinigame()
        end
        task.wait(config.minigameSpeed)
    end
end)

-- ==================== SISTEMA ESP ====================

local playerHighlights = {}

local function updateESP()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local char = plr.Character
            if espPlayers then
                if not playerHighlights[plr] then
                    local hl = Instance.new("Highlight")
                    hl.FillColor = config.espColor
                    hl.OutlineTransparency = 0
                    hl.Parent = char
                    playerHighlights[plr] = hl
                end
            else
                if playerHighlights[plr] then
                    playerHighlights[plr]:Destroy()
                    playerHighlights[plr] = nil
                end
            end
        end
    end
end

task.spawn(function()
    while true do
        updateESP()
        task.wait(1)
    end
end)

-- ==================== INTERFACE ====================

local Window = Rayfield:CreateWindow({
    Name = "Bridger: WESTERN",
    LoadingTitle = "Iniciando...",
    LoadingSubtitle = "by Bridger",
    ConfigurationSaving = {
        Enabled = true,
        FileName = "BridgerConfig"
    }
})

local MainTab = Window:CreateTab("🎣 Pesca", 4483362458)
MainTab:CreateToggle({
    Name = "Pesca Automática",
    CurrentValue = false,
    Callback = function(v) autoFishing = v end
})

MainTab:CreateToggle({
    Name = "Auto Minigame",
    CurrentValue = false,
    Callback = function(v) autoMinigame = v end
})

local EspTab = Window:CreateTab("👁️ Visual", 4483345998)
EspTab:CreateToggle({
    Name = "ESP Jogadores",
    CurrentValue = false,
    Callback = function(v) espPlayers = v end
})

EspTab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = Color3.fromRGB(0, 255, 0),
    Callback = function(v) config.espColor = v end
})

Rayfield:LoadConfiguration()
