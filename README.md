--[[
📜 AutoFarm Hub com múltiplos alvos
🛠️ Botões para: Strong Whitesuit (padrão), Arima, Eto Yoshimura, Yoshimura, Amon
]]

-- 🧱 Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- ⚙️ Variáveis principais
local TargetName = "Strong Whitesuit"
local autoFarm = false
local attacking = false
local currentTarget = nil
local died = false

-- 🧠 Funções utilitárias
local function getCombat()
return LocalPlayer.Backpack and LocalPlayer.Backpack:FindFirstChild("Combat")
end

local function getEquip()
local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
return humanoid and humanoid:FindFirstChild("Equip")
end

local function activateCombat()
local combat = getCombat()
if combat and not combat.Value then
combat.Value = true
end
end

local function activateEquip()
local equip = getEquip()
if equip and not equip.Value then
equip.Value = true
end
end

local function getTarget()
for _, v in pairs(workspace:GetDescendants()) do
if v:IsA("Model") and v.Name == TargetName and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") then
if v.Humanoid.Health > 0 then
return v
end
end
end
return nil
end

-- 🖼️ GUI Hub
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "AutoFarmHub"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 200, 0, 300)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
local UIS = game:GetService("UserInputService")
local dragging = false
local dragInput, dragStart, startPos

MainFrame.InputBegan:Connect(function(input)
if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
dragging = true
dragStart = input.Position
startPos = MainFrame.Position

input.Changed:Connect(function()  
		if input.UserInputState == Enum.UserInputState.End then  
			dragging = false  
		end  
	end)  
end

end)

MainFrame.InputChanged:Connect(function(input)
if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
dragInput = input
end
end)

UIS.InputChanged:Connect(function(input)
if input == dragInput and dragging then
local delta = input.Position - dragStart
MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end
end)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Title.Text = "📍 AutoFarm Hub"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20

-- 🟥 Botão AutoFarm
local AutoFarmButton = Instance.new("TextButton", MainFrame)
AutoFarmButton.Position = UDim2.new(0, 10, 0, 40)
AutoFarmButton.Size = UDim2.new(0, 180, 0, 40)
AutoFarmButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
AutoFarmButton.Text = "Auto Farm [OFF]"
AutoFarmButton.TextColor3 = Color3.new(1, 1, 1)
AutoFarmButton.Font = Enum.Font.SourceSansBold
AutoFarmButton.TextSize = 18

AutoFarmButton.MouseButton1Click:Connect(function()
autoFarm = not autoFarm
if autoFarm then
AutoFarmButton.Text = "Auto Farm [ON]"
AutoFarmButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
else
AutoFarmButton.Text = "Auto Farm [OFF]"
AutoFarmButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
end
end)

-- 📌 Função para criar botão de NPC
local function createNPCButton(name, yPosition)
local btn = Instance.new("TextButton", MainFrame)
btn.Position = UDim2.new(0, 10, 0, yPosition)
btn.Size = UDim2.new(0, 180, 0, 30)
btn.BackgroundColor3 = Color3.fromRGB(80, 80, 255)
btn.Text = "Target: " .. name
btn.TextColor3 = Color3.new(1, 1, 1)
btn.Font = Enum.Font.SourceSansBold
btn.TextSize = 16

btn.MouseButton1Click:Connect(function()  
    TargetName = name  
    btn.Text = "🎯 Target: " .. name  
    for _, child in pairs(MainFrame:GetChildren()) do  
        if child:IsA("TextButton") and child ~= btn and string.find(child.Text, "Target:") then  
            child.Text = "Target: " .. child.Text:split(": ")[2]  
        end  
    end  
end)

end

-- 🎯 Criação dos botões de alvo
createNPCButton("Strong Whitesuit", 85)
createNPCButton("Arima", 120)
createNPCButton("Eto Yoshimura", 155)
createNPCButton("Yoshimura", 190)
createNPCButton("Amon", 225)

-- 🚶‍♂️ Movimento automático sob o inimigo
RunService.RenderStepped:Connect(function()
if not autoFarm or not currentTarget or not currentTarget:FindFirstChild("HumanoidRootPart") then return end

local char = LocalPlayer.Character  
if char and char:FindFirstChild("HumanoidRootPart") then  
    char.HumanoidRootPart.CFrame = currentTarget.HumanoidRootPart.CFrame * CFrame.new(0, -8.4, 0)  
end

end)

-- ❤️ Verificador de vida e ativador de Equip/Combat
task.spawn(function()
while true do
local char = LocalPlayer.Character
if char and char:FindFirstChild("Humanoid") then
local health = char.Humanoid.Health

if health == 0 and not died then  
            died = true  
            task.wait(0.5)  
            activateEquip()  
        elseif health > 0 then  
            died = false  
            activateCombat()  
        end  
    end  
    task.wait(0.2)  
end

end)

-- 🔁 Loop principal do AutoFarm
task.spawn(function()
while true do
if autoFarm and not attacking then
local char = LocalPlayer.Character
if char and char:FindFirstChild("HumanoidRootPart") then
local target = getTarget()
if target then
attacking = true
currentTarget = target

local enemyDied = false  
                local diedConn = target.Humanoid.Died:Connect(function()  
                    enemyDied = true  
                    diedConn:Disconnect()  
                end)  

                local start = tick()  
                repeat task.wait(0.1) until enemyDied or tick() - start >= 9  

                currentTarget = nil  
                attacking = false  
                task.wait(0.9)  
            end  
        end  
    end  
    task.wait(0.1)  
end

end)

-- 🚫 Noclip ativado automaticamente com AutoFarm
RunService.Stepped:Connect(function()
if autoFarm then
local char = LocalPlayer.Character
if char then
for _, part in ipairs(char:GetDescendants()) do
if part:IsA("BasePart") and part.CanCollide then
part.CanCollide = false
end
end
end
end
end)

-- 🕊️ Fly anti-arremesso (ativa só com AutoFarm)
RunService.Heartbeat:Connect(function()
	if autoFarm then
		local char = LocalPlayer.Character
		if char and char:FindFirstChild("HumanoidRootPart") then
			local hrp = char.HumanoidRootPart
			local bv = hrp:FindFirstChild("AF_Fly") or Instance.new("BodyVelocity")
			bv.Name = "AF_Fly"
			bv.Velocity = Vector3.new(0, 0, 0)
			bv.MaxForce = Vector3.new(1e6, 1e6, 1e6)
			bv.P = 10000
			bv.Parent = hrp
		end
	else
		local char = LocalPlayer.Character
		if char and char:FindFirstChild("HumanoidRootPart") then
			local bv = char.HumanoidRootPart:FindFirstChild("AF_Fly")
			if bv then bv:Destroy() end
		end
	end
end)
