-- Script de Invisibilidade Universal para Roblox
-- Executa em qualquer jogo

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Variáveis de controle
local isInvisible = false
local originalTransparencies = {}
local connections = {}

-- Função para salvar transparências originais
local function saveOriginalTransparencies(character)
    originalTransparencies = {}
    
    -- Salvar transparências das partes do corpo
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            originalTransparencies[part] = part.Transparency
        end
    end
    
    -- Salvar transparências dos acessórios
    for _, accessory in pairs(character:GetChildren()) do
        if accessory:IsA("Accessory") then
            local handle = accessory:FindFirstChild("Handle")
            if handle then
                originalTransparencies[handle] = handle.Transparency
            end
        end
    end
end

-- Função para tornar invisível
local function makeInvisible(character)
    if not character then return end
    
    -- Tornar partes do corpo invisíveis
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.Transparency = 1
        end
    end
    
    -- Tornar acessórios invisíveis
    for _, accessory in pairs(character:GetChildren()) do
        if accessory:IsA("Accessory") then
            local handle = accessory:FindFirstChild("Handle")
            if handle then
                handle.Transparency = 1
            end
        end
    end
    
    -- Tornar ferramentas invisíveis
    for _, tool in pairs(character:GetChildren()) do
        if tool:IsA("Tool") then
            for _, part in pairs(tool:GetChildren()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                end
            end
        end
    end
end

-- Função para tornar visível
local function makeVisible(character)
    if not character then return end
    
    -- Restaurar transparências das partes do corpo
    for part, transparency in pairs(originalTransparencies) do
        if part and part.Parent then
            part.Transparency = transparency
        end
    end
    
    -- Restaurar ferramentas visíveis
    for _, tool in pairs(character:GetChildren()) do
        if tool:IsA("Tool") then
            for _, part in pairs(tool:GetChildren()) do
                if part:IsA("BasePart") then
                    part.Transparency = 0
                end
            end
        end
    end
end

-- Função para alternar invisibilidade
local function toggleInvisibility()
    local character = player.Character
    if not character then return end
    
    isInvisible = not isInvisible
    
    if isInvisible then
        saveOriginalTransparencies(character)
        makeInvisible(character)
        toggleButton.Text = "Ficar Visível"
        toggleButton.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    else
        makeVisible(character)
        toggleButton.Text = "Ficar Invisível"
        toggleButton.BackgroundColor3 = Color3.fromRGB(85, 255, 85)
    end
end

-- Função para monitorar novas ferramentas
local function onToolAdded(tool)
    if isInvisible then
        tool.Equipped:Connect(function()
            wait(0.1) -- Pequeno delay para garantir que a ferramenta foi equipada
            for _, part in pairs(tool:GetChildren()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                end
            end
        end)
    end
end

-- Função para configurar monitoramento do personagem
local function setupCharacter(character)
    if not character then return end
    
    -- Limpar conexões anteriores
    for _, connection in pairs(connections) do
        connection:Disconnect()
    end
    connections = {}
    
    -- Monitorar novas ferramentas
    table.insert(connections, character.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then
            onToolAdded(child)
        end
    end))
    
    -- Salvar transparências originais
    saveOriginalTransparencies(character)
    
    -- Se estava invisível, aplicar invisibilidade ao novo personagem
    if isInvisible then
        makeInvisible(character)
    end
end

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InvisibilityGUI"
screenGui.Parent = playerGui

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 100)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Adicionar bordas arredondadas
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Invisibilidade"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

-- Botão de invisibilidade
toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0.9, 0, 0, 40)
toggleButton.Position = UDim2.new(0.05, 0, 0, 50)
toggleButton.BackgroundColor3 = Color3.fromRGB(85, 255, 85)
toggleButton.Text = "Ficar Invisível"
toggleButton.TextColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.Gotham
toggleButton.BorderSizePixel = 0
toggleButton.Parent = mainFrame

-- Bordas arredondadas do botão
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 6)
buttonCorner.Parent = toggleButton

-- Efeito de hover no botão
toggleButton.MouseEnter:Connect(function()
    local tween = TweenService:Create(toggleButton, TweenInfo.new(0.2), {
        Size = UDim2.new(0.95, 0, 0, 42)
    })
    tween:Play()
end)

toggleButton.MouseLeave:Connect(function()
    local tween = TweenService:Create(toggleButton, TweenInfo.new(0.2), {
        Size = UDim2.new(0.9, 0, 0, 40)
    })
    tween:Play()
end)

-- Conectar botão
toggleButton.MouseButton1Click:Connect(toggleInvisibility)

-- Sistema de arrastar GUI
local dragging = false
local dragStart = nil
local startPos = nil

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Configurar personagem atual
if player.Character then
    setupCharacter(player.Character)
end

-- Monitorar respawn do jogador
player.CharacterAdded:Connect(setupCharacter)

print("Script de Invisibilidade carregado com sucesso!")
