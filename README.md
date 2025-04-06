-- Cria a tela (ScreenGui)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MinhaTela"
screenGui.ResetOnSpawn = false

-- Cria um texto na tela
local texto = Instance.new("TextLabel")
texto.Size = UDim2.new(1, 0, 1, 0)
texto.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
texto.TextColor3 = Color3.fromRGB(255, 255, 255)
texto.TextScaled = true
texto.Text = "Bem-vindo ao Script do Criador!"
texto.Parent = screenGui

-- Coloca na tela do jogador
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Espera 5 segundos e depois remove
wait(5)
screenGui:Destroy()
