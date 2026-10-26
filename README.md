--[[
    SCRIPT: Reroll Manager (Foco em Defesa do Reino dos Doces)
    FUNÇÃO: Tentar segurar o envio de dados e forçar desconexão rápida.
    
    AVISO: Este script não cria itens do nada. Ele apenas tenta enganar
    a comunicação entre o seu celular e o servidor da Roblox.
]]

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- Variáveis de Estado
local isSaveActive = false
local savedJobId = game.JobId
local originalNamecall = nil

-- ==========================================
-- 1. CRIAÇÃO DA INTERFACE (UI)
-- ==========================================

local oldGui = CoreGui:FindFirstChild("RerollManagerUI")
if oldGui then oldGui:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RerollManagerUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
ScreenGui.DisplayOrder = 999

-- Painel Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 260, 0, 250)
MainFrame.Position = UDim2.new(0, 20, 0, 100)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "REROLL CONTROL"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.Parent = MainFrame

-- Botão Toggle (Ligar/Desligar Save)
local ToggleSaveBtn = Instance.new("TextButton")
ToggleSaveBtn.Size = UDim2.new(1, -20, 0, 40)
ToggleSaveBtn.Position = UDim2.new(0, 10, 0, 45)
ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60) -- Cinza (Desligado)
ToggleSaveBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleSaveBtn.Text = "🔴 Salvar Estado: DESLIGADO"
ToggleSaveBtn.Font = Enum.Font.Gotham
ToggleSaveBtn.TextSize = 13
ToggleSaveBtn.Parent = MainFrame

local ToggleSaveCorner = Instance.new("UICorner")
ToggleSaveCorner.CornerRadius = UDim.new(0, 5)
ToggleSaveCorner.Parent = ToggleSaveBtn

-- Botão Relogar Normal
local RejoinBtn = Instance.new("TextButton")
RejoinBtn.Size = UDim2.new(1, -20, 0, 40)
RejoinBtn.Position = UDim2.new(0, 10, 0, 95)
RejoinBtn.BackgroundColor3 = Color3.fromRGB(40, 80, 150)
RejoinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RejoinBtn.Text = "🔄 Relogar (Rejoin)"
RejoinBtn.Font = Enum.Font.Gotham
RejoinBtn.TextSize = 13
RejoinBtn.Parent = MainFrame

local RejoinCorner = Instance.new("UICorner")
RejoinCorner.CornerRadius = UDim.new(0, 5)
RejoinCorner.Parent = RejoinBtn

-- Botão Desconexão Rápida (Kick)
local QuickKickBtn = Instance.new("TextButton")
QuickKickBtn.Size = UDim2.new(1, -20, 0, 40)
QuickKickBtn.Position = UDim2.new(0, 10, 0, 145)
QuickKickBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
QuickKickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
QuickKickBtn.Text = "🚪 Desconexão Rápida (Kick)"
QuickKickBtn.Font = Enum.Font.Gotham
QuickKickBtn.TextSize = 13
QuickKickBtn.Parent = MainFrame

local KickCorner = Instance.new("UICorner")
KickCorner.CornerRadius = UDim.new(0, 5)
KickCorner.Parent = QuickKickBtn

-- Status Label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 40)
StatusLabel.Position = UDim2.new(0, 10, 0, 195)
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
StatusLabel.Text = "Status: Aguardando..."
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 11
StatusLabel.TextWrapped = true
StatusLabel.Parent = MainFrame

-- ==========================================
-- 2. LÓGICA DO SISTEMA DE SALVAMENTO (TOGGLE)
-- ==========================================

local function updateStatus(text, color)
    StatusLabel.Text = "Status: " .. text
    StatusLabel.TextColor3 = color or Color3.fromRGB(150, 150, 150)
end

-- Função que ativa o Hook (bloqueio de rede)
local function ativarSaveState()
    if isSaveActive then return end
    
    -- Tenta interceptar a função de envio do Roblox
    local success, err = pcall(function()
        originalNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local method = getnamecallmethod()
            local args = {...}
            
            -- Se o save estiver ativo e for um envio pro servidor
            if isSaveActive and method == "FireServer" then
                local remoteName = tostring(self.Name):lower()
                
                -- Bloqueia remotes que tenham a ver com Reroll/Itens
                if remoteName:find("reroll") or remoteName:find("trait") or remoteName:find("crystal") or remoteName:find("roll") then
                    print("[REROLL CONTROL] Interceptado e bloqueado: " .. remoteName)
                    return nil -- Impede o envio
                end
            end
            
            return originalNamecall(self, ...)
        end)
    end)
    
    if success then
        isSaveActive = true
        ToggleSaveBtn.Text = "🟢 Salvar Estado: LIGADO"
        ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(40, 120, 40) -- Verde
        updateStatus("Proteção Ativada! O jogo tentará não gastar itens.", Color3.fromRGB(100, 255, 100))
    else
        updateStatus("Erro ao ativar proteção. Executor sem suporte.", Color3.fromRGB(255, 100, 100))
    end
end

-- Função que desativa o Hook
local function desativarSaveState()
    isSaveActive = false
    -- Em Lua, não conseguimos "desfazer" um hookmetamethod facilmente, mas como
    -- a variável isSaveActive agora é false, o script vai parar de bloquear.
    ToggleSaveBtn.Text = "🔴 Salvar Estado: DESLIGADO"
    ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60) -- Cinza
    updateStatus("Proteção Desativada. O jogo voltou ao normal.", Color3.fromRGB(255, 200, 0))
end

-- Evento de clique do Toggle
ToggleSaveBtn.MouseButton1Click:Connect(function()
    if isSaveActive then
        desativarSaveState()
    else
        ativarSaveState()
    end
end)

-- ==========================================
-- 3. LÓGICA DE REJOIN E DESCONEXÃO RÁPIDA
-- ==========================================

-- Botão Relogar Normal (Tenta voltar pro mesmo servidor)
RejoinBtn.MouseButton1Click:Connect(function()
    updateStatus("Reconectando ao servidor...", Color3.fromRGB(255, 200, 0))
    pcall(function()
        if savedJobId and savedJobId ~= "" then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, savedJobId, LocalPlayer)
        else
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end
    end)
end)

-- Botão de Desconexão Rápida (Kick / Crash)
QuickKickBtn.MouseButton1Click:Connect(function()
    updateStatus("Desconectando forçadamente...", Color3.fromRGB(255, 100, 100))
    
    -- Pausa de 0.5 segundos para a UI atualizar na tela antes de fechar
    task.wait(0.5)
    
    -- O 'LocalPlayer:Kick()' é o método mais rápido de fechar o jogo.
    -- Ele impede que o seu celular envie os pacotes finais para o servidor.
    pcall(function()
        LocalPlayer:Kick("Desconexão Rápida Ativada!")
    end)
end)

updateStatus("Script carregado. Ative a proteção antes de rolar.", Color3.fromRGB(150, 150, 150))
