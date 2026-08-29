print("TESTE 1: executei")

local gui = Instance.new("ScreenGui")
gui.Name = "TesteRedOnyx"

local txt = Instance.new("TextLabel")
txt.Size = UDim2.new(0, 300, 0, 60)
txt.Position = UDim2.new(0.5, 0, 0.3, 0)
txt.AnchorPoint = Vector2.new(0.5, 0)
txt.BackgroundColor3 = Color3.fromRGB(15, 12, 14)
txt.TextColor3 = Color3.fromRGB(255, 90, 105)
txt.TextSize = 20
txt.Font = Enum.Font.GothamBlack
txt.Text = "RED ONYX FUNCIONA!"

local ok = pcall(function() txt.Parent = game:GetService("CoreGui") end)
if not ok then
    pcall(function()
        txt.Parent = game:GetService("Players").LocalPlayer.PlayerGui
    end)
end
gui.Parent = txt.Parent == nil and nil or txt
print("TESTE 2: UI criada")
