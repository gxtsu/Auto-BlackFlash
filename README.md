local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Skill3 = ReplicatedStorage:WaitForChild("Skill3")
local BlackFlashEvent = ReplicatedStorage:WaitForChild("BlackFlashEvent")

local lastUse = {}

Skill3.OnServerEvent:Connect(function(player)
	local now = os.clock()

	if lastUse[player] and (now - lastUse[player]) <= 0.5 then
		BlackFlashEvent:FireClient(player)
	end

	lastUse[player] = now
end)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local event = ReplicatedStorage:WaitForChild("BlackFlashEvent")

local frame = script.Parent:WaitForChild("FlashFrame")
local text = script.Parent:WaitForChild("FlashText")

local flashTweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local fadeTweenInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

event.OnClientEvent:Connect(function()
	frame.BackgroundTransparency = 1
	text.TextTransparency = 1

	TweenService:Create(frame, flashTweenInfo, {
		BackgroundTransparency = 0.4
	}):Play()

	TweenService:Create(text, flashTweenInfo, {
		TextTransparency = 0
	}):Play()

	task.delay(0.35, function()
		TweenService:Create(frame, fadeTweenInfo, {
			BackgroundTransparency = 1
		}):Play()

		TweenService:Create(text, fadeTweenInfo, {
			TextTransparency = 1
		}):Play()
	end)
end)
