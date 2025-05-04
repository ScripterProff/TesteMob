-- GUI do botão
local Player = game.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui", PlayerGui)
ScreenGui.Name = "ESP_Interface"

local ToggleButton = Instance.new("TextButton", ScreenGui)
ToggleButton.Size = UDim2.new(0, 120, 0, 40)
ToggleButton.Position = UDim2.new(0, 20, 0, 100)
ToggleButton.Text = "Ativar ESP"
ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 18

-- Variáveis
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local ESPEnabled = false
local ESPObjects = {}

-- Função para criar ESP
local function CreateESP(player)
    if player == Player then return end
    local esp = {}

    -- Caixa
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Thickness = 1.5
    box.Transparency = 1
    esp.Box = box

    -- Nome
    local nameTag = Drawing.new("Text")
    nameTag.Visible = false
    nameTag.Center = true
    nameTag.Outline = true
    nameTag.Color = Color3.fromRGB(255, 255, 255)
    nameTag.Size = 14
    esp.Name = nameTag

    ESPObjects[player] = esp

    -- Atualização por frame
    RunService.RenderStepped:Connect(function()
        if not ESPEnabled or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or player.Team == Player.Team then
            box.Visible = false
            nameTag.Visible = false
            return
        end

        local character = player.Character
        local hrp = character:FindFirstChild("HumanoidRootPart")
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not hrp or not humanoid then return end

        -- Posições 3D para 2D
        local pos, onscreen = Camera:WorldToViewportPoint(hrp.Position)
        if onscreen then
            local size = Vector3.new(2, 3, 1.5) * humanoid.HipHeight
            local topLeft = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(-size.X, size.Y, 0))
            local bottomRight = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(size.X, -size.Y, 0))
            
            box.Size = Vector2.new(math.abs(bottomRight.X - topLeft.X), math.abs(topLeft.Y - bottomRight.Y))
            box.Position = Vector2.new(topLeft.X, topLeft.Y)
            box.Visible = true

            nameTag.Text = player.Name
            nameTag.Position = Vector2.new(pos.X, pos.Y - 40)
            nameTag.Visible = true
        else
            box.Visible = false
            nameTag.Visible = false
        end
    end)
end

-- Criar ESP para todos os jogadores
for _, p in ipairs(game.Players:GetPlayers()) do
    CreateESP(p)
end

-- Novos jogadores
game.Players.PlayerAdded:Connect(CreateESP)

-- Jogadores saindo
game.Players.PlayerRemoving:Connect(function(p)
    if ESPObjects[p] then
        ESPObjects[p].Box:Remove()
        ESPObjects[p].Name:Remove()
        ESPObjects[p] = nil
    end
end)

-- Botão de toggle
ToggleButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    ToggleButton.Text = ESPEnabled and "Desativar ESP" or "Ativar ESP"
end)
