--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--// Config
local AimPart = "Head"
local AimbotEnabled = true
local ESPParts = {}
local JumpCounts = {}

--// Notificação
StarterGui:SetCore("SendNotification", {
    Title = "By ImcomumPlayer",
    Text = "Aimbot + ESP Ativado",
    Icon = "rbxthumb://type=AvatarHeadShot&id=9244921300&w=420&h=420",
    Duration = 5
})

--// Som ao executar
local sound = Instance.new("Sound", workspace)
sound.SoundId = "rbxassetid://6821054463"
sound.Volume = 5
sound:Play()

--// Frame explicativo
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "AimbotInfoGUI"

local infoFrame = Instance.new("Frame", screenGui)
infoFrame.Size = UDim2.new(0,400,0,200)
infoFrame.Position = UDim2.new(0.5,0,0.5,0)
infoFrame.AnchorPoint = Vector2.new(0.5,0.5)
infoFrame.BackgroundColor3 = Color3.fromRGB(0,0,50)
infoFrame.BorderSizePixel = 0
infoFrame.BackgroundTransparency = 0.1
infoFrame.ClipsDescendants = true

local infoLabel = Instance.new("TextLabel", infoFrame)
infoLabel.Size = UDim2.new(1, -20, 1, -40)
infoLabel.Position = UDim2.new(0,10,0,10)
infoLabel.Text = "Aimbot trava bruscamente na cabeça do jogador mais próximo.\nESP mostra todos os jogadores com ponto vermelho, linha até a lua, nome e distância.\nPara parar o ESP de um jogador, pule 2 vezes."
infoLabel.TextColor3 = Color3.fromRGB(255,255,255)
infoLabel.BackgroundTransparency = 1
infoLabel.TextWrapped = true
infoLabel.Font = Enum.Font.SourceSansBold
infoLabel.TextScaled = true

local closeButton = Instance.new("TextButton", infoFrame)
closeButton.Size = UDim2.new(0,80,0,30)
closeButton.Position = UDim2.new(0.5,-40,1,-40)
closeButton.Text = "Fechar"
closeButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
closeButton.TextColor3 = Color3.fromRGB(255,255,255)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextScaled = true
closeButton.BorderSizePixel = 0
closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

--// Função para criar ESP
local function createESP(player)
    if player == LocalPlayer then return end
    local char = player.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    if ESPParts[player] then return end -- Se já existe ESP, não recria

    -- Ponto vermelho pulsante
    local billboard = Instance.new("BillboardGui", head)
    billboard.Size = UDim2.new(0,10,0,10)
    billboard.AlwaysOnTop = true
    local frame = Instance.new("Frame", billboard)
    frame.Size = UDim2.new(1,0,1,0)
    frame.BackgroundColor3 = Color3.fromRGB(255,0,0)
    frame.BorderSizePixel = 0

    spawn(function()
        while billboard.Parent do
            for i=0,1,0.03 do
                frame.BackgroundTransparency = i
                wait(0.03)
            end
            for i=1,0,-0.03 do
                frame.BackgroundTransparency = i
                wait(0.03)
            end
        end
    end)

    -- Linha até a lua
    local line = Instance.new("Part")
    line.Anchored = true
    line.CanCollide = false
    line.Material = Enum.Material.Neon
    line.Color = Color3.fromRGB(255,0,0)
    line.Size = Vector3.new(0.2,0.2,5000)
    line.CFrame = CFrame.new(head.Position, head.Position + Vector3.new(0,5000,0))
    line.Parent = Workspace

    -- Nome do jogador
    local nameLabel = Instance.new("BillboardGui", head)
    nameLabel.Size = UDim2.new(0,100,0,25)
    nameLabel.StudsOffset = Vector3.new(0,3,0)
    nameLabel.AlwaysOnTop = true
    local nameText = Instance.new("TextLabel", nameLabel)
    nameText.Size = UDim2.new(1,0,1,0)
    nameText.BackgroundTransparency = 1
    nameText.TextColor3 = Color3.fromRGB(255,255,255)
    nameText.Text = player.Name
    nameText.Font = Enum.Font.SourceSansBold
    nameText.TextScaled = true

    -- Distância
    local distanceLabel = Instance.new("BillboardGui", head)
    distanceLabel.Size = UDim2.new(0,50,0,20)
    distanceLabel.StudsOffset = Vector3.new(0,2,0)
    distanceLabel.AlwaysOnTop = true
    local distText = Instance.new("TextLabel", distanceLabel)
    distText.Size = UDim2.new(1,0,1,0)
    distText.BackgroundTransparency = 1
    distText.TextColor3 = Color3.fromRGB(255,255,255)
    distText.Font = Enum.Font.SourceSansBold
    distText.TextScaled = true

    ESPParts[player] = {
        Billboard = billboard,
        Line = line,
        NameLabel = nameText,
        DistanceLabel = distText
    }
    JumpCounts[player] = JumpCounts[player] or 0
end

-- Criar ESP para todos os jogadores existentes
for _, plr in pairs(Players:GetPlayers()) do
    if plr.Character then
        createESP(plr)
    end
end

-- Reconectar ESP ao spawn e reaplicar se estiver sem
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        createESP(plr)
    end)
end)

-- Loop para reaplicar ESP se jogador estiver sem
RunService.RenderStepped:Connect(function()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            if not ESPParts[plr] then
                createESP(plr)
            end
        end
    end
end)

-- Duplo jump para remover ESP
LocalPlayer.CharacterAdded:Connect(function(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Jumping:Connect(function()
        for player, parts in pairs(ESPParts) do
            JumpCounts[player] = (JumpCounts[player] or 0) + 1
            if JumpCounts[player] >= 2 then
                if parts.Billboard then parts.Billboard:Destroy() end
                if parts.Line then parts.Line:Destroy() end
                if parts.NameLabel then parts.NameLabel:Destroy() end
                if parts.DistanceLabel then parts.DistanceLabel:Destroy() end
                ESPParts[player] = nil
                JumpCounts[player] = 0
            end
        end
    end)
end)

-- Loop ESP para atualizar linha e distância
RunService.RenderStepped:Connect(function()
    for player, parts in pairs(ESPParts) do
        local char = player.Character
        if char and char:FindFirstChild("Head") then
            local head = char.Head
            if parts.Line then
                parts.Line.CFrame = CFrame.new(head.Position, head.Position + Vector3.new(0,5000,0))
            end
            if parts.DistanceLabel then
                local dist = (head.Position - LocalPlayer.Character.Head.Position).Magnitude
                parts.DistanceLabel.Text = string.format("%.1f studs", dist)
            end
        end
    end
end)

-- Aimbot brusco no jogador mais próximo
RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local closestPlayer = nil
        local shortestDist = math.huge
        for player, _ in pairs(ESPParts) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AimPart) then
                local dist = (player.Character[AimPart].Position - Camera.CFrame.Position).Magnitude
                if dist < shortestDist then
                    closestPlayer = player
                    shortestDist = dist
                end
            end
        end
        if closestPlayer then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, closestPlayer.Character[AimPart].Position)
        end
    end
end)
