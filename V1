-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Setup
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local communicate = character:WaitForChild("Communicate")

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "LoopdashGUI"
screenGui.ResetOnSpawn = false

local button = Instance.new("TextButton", screenGui)
button.Size = UDim2.new(0, 150, 0, 50)
button.Position = UDim2.new(0.5, -75, 0.85, 0)
button.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
button.TextColor3 = Color3.new(1, 1, 1)
button.Text = "Tap Aimloop 360"
button.Font = Enum.Font.GothamBold
button.TextSize = 20
button.Draggable = true

-- Character refresh
player.CharacterAdded:Connect(function(char)
	character = char
	hrp = character:WaitForChild("HumanoidRootPart")
	communicate = character:WaitForChild("Communicate")
end)

-- Get Closest Enemy (NPC / Player / Bot)
local function getClosestTarget()
	local closest, dist = nil, math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local mag = (p.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
			if mag < dist then
				closest, dist = p.Character, mag
			end
		end
	end
	for _, model in ipairs(workspace:GetDescendants()) do
		if model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") and model:FindFirstChildOfClass("Humanoid") then
			if model ~= character and not Players:GetPlayerFromCharacter(model) then
				local mag = (model.HumanoidRootPart.Position - hrp.Position).Magnitude
				if mag < dist then
					closest, dist = model, mag
				end
			end
		end
	end
	return closest
end

-- Simulate Q Dash
local function simulateQPress()
	local args = {
		[1] = {
			["Dash"] = Enum.KeyCode.W,
			["Key"] = Enum.KeyCode.Q,
			["Goal"] = "KeyPress"
		}
	}
	if communicate then
		communicate:FireServer(unpack(args))
	end
end

-- Toggle Noclip
local function setNoclip(state)
	for _, part in ipairs(character:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = not state
		end
	end
end

-- 360 Spin While Aimlocked
local function aimlockedSpin(target)
	local spinTime = 0.4
	local start = tick()
	local connection
	local lastAngle = 0

	connection = RunService.RenderStepped:Connect(function()
		local elapsed = tick() - start
		if elapsed >= spinTime then
			connection:Disconnect()
			return
		end

		if target and target:FindFirstChild("HumanoidRootPart") then
			local tPos = target.HumanoidRootPart.Position
			local dir = (tPos - hrp.Position).Unit
			local flatDir = Vector3.new(dir.X, 0, dir.Z)
			local baseLook = CFrame.lookAt(Vector3.zero, flatDir)

			local angle = (elapsed / spinTime) * math.pi * 2
			local delta = angle - lastAngle
			lastAngle = angle

			local spinOffset = CFrame.Angles(0, delta, 0)
			hrp.CFrame = CFrame.new(hrp.Position) * spinOffset:ToWorldSpace(baseLook)
		end
	end)
end

-- Main Logic
local isActive = false
button.MouseButton1Click:Connect(function()
	if isActive then return end
	isActive = true

	local target = getClosestTarget()
	if not target or not target:FindFirstChild("HumanoidRootPart") then
		isActive = false
		return
	end

	simulateQPress()
	setNoclip(true)

	local startTick = tick()
	local duration = 0.7
	local connection

	connection = RunService.RenderStepped:Connect(function()
		local elapsed = tick() - startTick
		if elapsed > duration then
			connection:Disconnect()
			setNoclip(false)
			isActive = false
			aimlockedSpin(target) -- 🔁 360 spin in-place, still aimlocked
			return
		end

		-- AIMLOCK ROTATION
		if target and target:FindFirstChild("HumanoidRootPart") then
			local tPos = target.HumanoidRootPart.Position
			local dir = (tPos - hrp.Position).Unit
			local flatDir = Vector3.new(dir.X, 0, dir.Z)
			local look = CFrame.lookAt(hrp.Position, hrp.Position + flatDir)
			hrp.CFrame = hrp.CFrame:Lerp(look, 0.35)
		end
	end)
end)
