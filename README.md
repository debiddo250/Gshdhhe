-- Configurações iniciais
local ESP = {
    Enabled = true,
    ShowNames = true,
    ShowBoxes = true,
    ShowTracers = true,
    TeamCheck = true
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- Função para criar UI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 150)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local function createButton(name, posY, callback)
    local button = Instance.new("TextButton", Frame)
    button.Size = UDim2.new(0, 180, 0, 30)
    button.Position = UDim2.new(0, 10, 0, posY)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.new(1,1,1)
    button.MouseButton1Click:Connect(callback)
end

createButton("Toggle ESP", 10, function() ESP.Enabled = not ESP.Enabled end)
createButton("Toggle Names", 50, function() ESP.ShowNames = not ESP.ShowNames end)
createButton("Toggle Boxes", 90, function() ESP.ShowBoxes = not ESP.ShowBoxes end)
createButton("Toggle Tracers", 130, function() ESP.ShowTracers = not ESP.ShowTracers end)

-- Função para criar ESP para cada player
local function DrawESP(player)
    local Box = Drawing.new("Square")
    Box.Color = Color3.new(1,0,0)
    Box.Thickness = 1
    Box.Transparency = 1
    Box.Filled = false

    local Tracer = Drawing.new("Line")
    Tracer.Color = Color3.new(0,1,0)
    Tracer.Thickness = 1
    Tracer.Transparency = 1

    local Name = Drawing.new("Text")
    Name.Size = 16
    Name.Color = Color3.new(1,1,1)
    Name.Center = true
    Name.Outline = true
    Name.Transparency = 1

    RunService.RenderStepped:Connect(function()
        if ESP.Enabled and player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            if ESP.TeamCheck and player.Team == LocalPlayer.Team then
                Box.Visible = false
                Tracer.Visible = false
                Name.Visible = false
                return
            end

            local Vector, onScreen = workspace.CurrentCamera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onScreen then
                if ESP.ShowBoxes then
                    Box.Visible = true
                    Box.Size = Vector2.new(50, 100)
                    Box.Position = Vector2.new(Vector.X - 25, Vector.Y - 50)
                else
                    Box.Visible = false
                end

                if ESP.ShowTracers then
                    Tracer.Visible = true
                    Tracer.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y)
                    Tracer.To = Vector2.new(Vector.X, Vector.Y)
                else
                    Tracer.Visible = false
                end

                if ESP.ShowNames then
                    Name.Visible = true
                    Name.Position = Vector2.new(Vector.X, Vector.Y - 60)
                    Name.Text = player.Name
                else
                    Name.Visible = false
                end
            else
                Box.Visible = false
                Tracer.Visible = false
                Name.Visible = false
            end
        else
            Box.Visible = false
            Tracer.Visible = false
            Name.Visible = false
        end
    end)
end

-- Aplicar ESP a todos os jogadores
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        DrawESP(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        DrawESP(player)
    end
end)
