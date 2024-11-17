local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer

-- Configurações do ESP
local espColor = Color3.fromRGB(255, 0, 0) -- Cor do ESP (Vermelho)
local fillTransparency = 0.5
local outlineTransparency = 0

-- Controle de ativação/desativação do ESP
local espEnabled = true -- O ESP começa ativado
local highlights = {} -- Armazena os Highlights criados para facilitar a remoção

-- Função para adicionar ESP a um modelo
local function addESP(model)
    if not espEnabled then return end -- Não faz nada se o ESP estiver desativado

    if model:FindFirstChild("HumanoidRootPart") then
        -- Cria um Highlight
        local highlight = Instance.new("Highlight")
        highlight.Parent = model
        highlight.Adornee = model
        highlight.FillColor = espColor
        highlight.OutlineColor = Color3.new(1, 1, 1) -- Branco
        highlight.FillTransparency = fillTransparency
        highlight.OutlineTransparency = outlineTransparency

        -- Armazena o Highlight para remoção futura
        table.insert(highlights, highlight)

        -- Remove o Highlight quando o modelo for destruído
        model.AncestryChanged:Connect(function()
            if not model:IsDescendantOf(game) then
                highlight:Destroy()
            end
        end)
    end
end

-- Adiciona ESP aos jogadores no jogo
local function setupESPForPlayer(player)
    if not espEnabled then return end -- Não faz nada se o ESP estiver desativado
    if player == localPlayer then return end -- Ignora o próprio jogador

    -- Adiciona ESP quando o personagem do jogador spawnar
    player.CharacterAdded:Connect(function(character)
        if espEnabled then
            character:WaitForChild("HumanoidRootPart") -- Espera o modelo carregar
            addESP(character)
        end
    end)

    -- Se o personagem já existir, adiciona ESP
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        addESP(player.Character)
    end
end

-- Configura ESP para todos os jogadores atuais
for _, player in ipairs(Players:GetPlayers()) do
    setupESPForPlayer(player)
end

-- Configura ESP para novos jogadores
Players.PlayerAdded:Connect(setupESPForPlayer)

-- Adiciona ESP para objetos específicos no jogo (opcional)
local function addESPToParts()
    if not espEnabled then return end -- Não faz nada se o ESP estiver desativado

    for _, part in ipairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") and part.Name == "Target" then -- Substitua "Target" pelo nome do objeto que você deseja destacar
            local highlight = Instance.new("Highlight")
            highlight.Parent = part
            highlight.Adornee = part
            highlight.FillColor = espColor
            highlight.OutlineColor = Color3.new(1, 1, 1) -- Branco
            highlight.FillTransparency = fillTransparency
            highlight.OutlineTransparency = outlineTransparency

            -- Armazena o Highlight para remoção futura
            table.insert(highlights, highlight)
        end
    end
end

-- Função para ativar/desativar o ESP
local function toggleESP(state)
    espEnabled = state

    if not espEnabled then
        -- Destruir todos os Highlights existentes
        for _, highlight in ipairs(highlights) do
            if highlight.Parent then
                highlight:Destroy()
            end
        end
        highlights = {} -- Limpa a lista de Highlights
    else
        -- Reaplicar o ESP
        for _, player in ipairs(Players:GetPlayers()) do
            setupESPForPlayer(player)
        end
        addESPToParts()
    end
end

-- Inicializa o ESP
addESPToParts()

-- Exemplo: Teste de ativar/desativar o ESP
-- toggleESP(false) -- Desativa o ESP
-- toggleESP(true)  -- Reativa o ESP
