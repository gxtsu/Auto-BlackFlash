-- ============================================
-- AUTO BLACK FLASH - JUJUTSU SHENANIGANS
-- ============================================

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetUserInputService()

local Skill3 = ReplicatedStorage:WaitForChild("Skill3")
local BlackFlashEvent = ReplicatedStorage:WaitForChild("BlackFlashEvent")

-- ============================================
-- SERVER SIDE
-- ============================================

local lastUse = {}

Skill3.OnServerEvent:Connect(function(player)
	local now = os.clock()

	-- Verifica se é a segunda ativação dentro de 0.20 segundos
	if lastUse[player] and (now - lastUse[player]) <= 0.20 then
		-- Black Flash foi ativado com sucesso!
		BlackFlashEvent:FireClient(player)
		lastUse[player] = nil -- Reseta o contador
		return
	end

	-- Primeira ativação registrada
	lastUse[player] = now
end)

-- ============================================
-- CLIENT SIDE
-- ============================================

local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui")
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

-- Frame vermelho para o efeito de flash
local flashFrame = Instance.new("Frame")
flashFrame.Name = "BlackFlashFrame"
flashFrame.Size = UDim2.new(1, 0, 1, 0)
flashFrame.Position = UDim2.new(0, 0, 0, 0)
flashFrame.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho
flashFrame.BackgroundTransparency = 1
flashFrame.BorderSizePixel = 0
flashFrame.Parent = screenGui
flashFrame.ZIndex = 100

-- Texto "BLACK FLASH!!!"
local flashText = Instance.new("TextLabel")
flashText.Name = "BlackFlashText"
flashText.Size = UDim2.new(1, 0, 0.15, 0)
flashText.Position = UDim2.new(0.5, 0, 0.8, 0) -- Mais embaixo na tela
flashText.AnchorPoint = Vector2.new(0.5, 0.5)
flashText.BackgroundTransparency = 1
flashText.Text = "BLACK FLASH!!!"
flashText.TextColor3 = Color3.fromRGB(0, 0, 0) -- Preto
flashText.TextSize = 72
flashText.Font = Enum.Font.GothamBold
flashText.TextTransparency = 1
flashText.ZIndex = 101
flashText.Parent = screenGui

-- Tween Info: entrada rápida (0.1s) e saída suave (0.2s)
local enterTweenInfo = TweenInfo.new(
	0.1,
	Enum.EasingStyle.Quad,
	Enum.EasingDirection.Out
)

local exitTweenInfo = TweenInfo.new(
	0.2,
	Enum.EasingStyle.Quad,
	Enum.EasingDirection.Out
)

-- Variável para controlar o delay da segunda ativação
local lastSkill3Press = 0

-- Detecta quando a skill 3 é pressionada
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	
	if input.KeyCode == Enum.KeyCode.Three then
		lastSkill3Press = os.clock()
	end
end)

-- Evento do servidor quando Black Flash é ativado
BlackFlashEvent.OnClientEvent:Connect(function()
	-- Entrada rápida: frame fica vermelho
	TweenService:Create(flashFrame, enterTweenInfo, {
		BackgroundTransparency = 0.4
	}):Play()

	-- Texto aparece
	TweenService:Create(flashText, enterTweenInfo, {
		TextTransparency = 0
	}):Play()

	-- Após a entrada, começa a saída suave
	task.delay(0.1, function()
		-- Saída suave: frame desaparece gradualmente
		TweenService:Create(flashFrame, exitTweenInfo, {
			BackgroundTransparency = 1
		}):Play()

		-- Texto desaparece
		TweenService:Create(flashText, exitTweenInfo, {
			TextTransparency = 1
		}):Play()
	end)
end)

-- Auto-click para o segundo toque do Black Flash
task.spawn(function()
	while true do
		task.wait(0.01)
		local now = os.clock()
		
		-- Se foi pressionado há 0.20s, simula o segundo clique
		if lastSkill3Press > 0 and (now - lastSkill3Press) >= 0.20 and (now - lastSkill3Press) <= 0.22 then
			Skill3:FireServer()
			lastSkill3Press = 0 -- Reseta
		end
	end
end)
