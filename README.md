repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local vim = game:GetService("VirtualInputManager")

local lp = Players.LocalPlayer
repeat task.wait() until lp

-- SETTINGS
local enabled = false
local target = nil

_G.HeadSize = 15
_G.Disabled = true

-- HITBOX EXCLUDE
local excludeName = "Bn_Quangthuc"
local hitboxEnabled = true

-- SPEED
local normalFly = 250
local highFly = 500
local startTeleportDistance = 18

local bv

-- TAP
local function tap(x,y)
	vim:SendTouchEvent(0, Enum.UserInputState.Begin, x, y)
	vim:SendTouchEvent(0, Enum.UserInputState.End, x, y)
end

-- BV
local function setupBV()
	local c = lp.Character
	if not c then return end
	local hrp = c:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	
	if bv then bv:Destroy() end
	
	bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.new(9e9,9e9,9e9)
	bv.Velocity = Vector3.zero
	bv.Parent = hrp
end

lp.CharacterAdded:Connect(function()
	task.wait(1)
	if enabled then setupBV() end
end)

-- FIX mất BV
RunService.Heartbeat:Connect(function()
	if enabled and not bv then
		setupBV()
	end
end)

-- NOCLIP
RunService.Stepped:Connect(function()
	if not enabled then return end
	
	local char = lp.Character
	if not char then return end
	
	for _,v in pairs(char:GetDescendants()) do
		if v:IsA("BasePart") then
			v.CanCollide = false
		end
	end
end)

-- HITBOX (có loại trừ)
task.spawn(function()
	while true do
		task.wait(0.2)
		
		if _G.Disabled and hitboxEnabled then
			for _,v in pairs(Players:GetPlayers()) do
				if v ~= lp and v.Character and v.Name ~= excludeName then
					
					local hrp = v.Character:FindFirstChild("HumanoidRootPart")
					if hrp then
						hrp.Size = Vector3.new(_G.HeadSize,_G.HeadSize,_G.HeadSize)
						hrp.Transparency = 0.6
						hrp.CanCollide = false
					end
					
				end
			end
		end
	end
end)

-- AUTO TARGET
local function getAlivePlayers()
	local t = {}
	for _,plr in ipairs(Players:GetPlayers()) do
		if plr ~= lp and plr.Character then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then
				table.insert(t, plr)
			end
		end
	end
	return t
end

local function pickRandomTarget()
	local list = getAlivePlayers()
	if #list > 0 then
		target = list[math.random(1,#list)]
	end
end

-- GUI
local gui = Instance.new("ScreenGui", game.CoreGui)

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,230,0,300)
frame.Position = UDim2.new(0,80,0,120)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Text = "⚔ Combat FIX PRO"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.BackgroundTransparency = 1

local toggle = Instance.new("TextButton", frame)
toggle.Size = UDim2.new(1,0,0,35)
toggle.Position = UDim2.new(0,0,0,35)
toggle.Text = "OFF"

toggle.MouseButton1Click:Connect(function()
	enabled = not enabled
	toggle.Text = enabled and "ON" or "OFF"
	
	if enabled then setupBV()
	else if bv then bv:Destroy() bv=nil end end
end)

-- HITBOX TOGGLE BUTTON
local hitboxBtn = Instance.new("TextButton", frame)
hitboxBtn.Size = UDim2.new(1,0,0,30)
hitboxBtn.Position = UDim2.new(0,0,0,75)
hitboxBtn.Text = "Hitbox: ON"

hitboxBtn.MouseButton1Click:Connect(function()
	hitboxEnabled = not hitboxEnabled
	hitboxBtn.Text = hitboxEnabled and "Hitbox: ON" or "Hitbox: OFF"
end)

-- PLAYER LIST
local list = Instance.new("Frame", frame)
list.Position = UDim2.new(0,0,0,110)
list.Size = UDim2.new(1,0,1,-110)
list.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", list)

local function updateList()
	for _,v in pairs(list:GetChildren()) do
		if v:IsA("TextButton") then v:Destroy() end
	end
	
	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= lp then
			local btn = Instance.new("TextButton")
			btn.Parent = list
			btn.Size = UDim2.new(1,0,0,25)
			btn.Text = plr.Name
			
			btn.MouseButton1Click:Connect(function()
				target = plr
				title.Text = "Target: "..plr.Name
			end)
		end
	end
end

updateList()
Players.PlayerAdded:Connect(function() task.wait(1) updateList() end)
Players.PlayerRemoving:Connect(updateList)

-- AUTO HIT + AUTO TARGET
task.spawn(function()
	while true do
		task.wait(0.08)
		
		if enabled then
			
			if not target then
				pickRandomTarget()
			end
			
			if target and target.Character then
				local hum = target.Character:FindFirstChildOfClass("Humanoid")
				if not hum or hum.Health <= 0 or hum.Health < 10 then
					pickRandomTarget()
				end
			end
			
			local char = lp.Character
			local tchar = target and target.Character
			
			if char and tchar then
				
				local hrp = char:FindFirstChild("HumanoidRootPart")
				local tr = tchar:FindFirstChild("HumanoidRootPart")
				
				if hrp and tr then
					local dist = (hrp.Position - tr.Position).Magnitude
					
					if dist < 25 then
						tap(825,315)
						task.wait(0.02)
						tap(895,315)
					end
				end
			end
		end
	end
end)

-- MAIN
RunService.Heartbeat:Connect(function()

	local char = lp.Character
	if not char then return end
	
	local hrp = char:FindFirstChild("HumanoidRootPart")
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum then return end

	if hum.JumpPower == 0 then
		if bv then
			local startY = hrp.Position.Y
			
			for i = 1,6 do
				task.wait(0.03)
				
				local h = hrp.Position.Y - startY
				
				local dir = Vector3.new(
					math.random(-100,100)/100,
					0.8,
					math.random(-100,100)/100
				)
				
				if dir.Magnitude < 0.1 then
					dir = Vector3.new(0,1,0)
				end
				
				dir = dir.Unit
				
				if h > 50 then
					dir = Vector3.new(dir.X,-0.6,dir.Z).Unit
				end
				
				bv.Velocity = dir * 500
			end
		end
		return
	end

	if enabled and target and bv then
		
		local tchar = target.Character
		if not tchar then return end
		
		local troot = tchar:FindFirstChild("HumanoidRootPart")
		if not troot then return end
		
		local dist = (hrp.Position - troot.Position).Magnitude
		
		local dir = (troot.Position - hrp.Position)
		
		if dir.Magnitude < 0.1 then
			dir = Vector3.new(0,0.1,0)
		end
		
		dir = dir.Unit
		
		if dist > startTeleportDistance then
			
			local speed = highFly
			if dist < 40 then speed = normalFly end
			
			local targetVel = dir * speed
			
			if dist > 100 then
				bv.Velocity = targetVel
			else
				bv.Velocity = bv.Velocity * 0.75 + targetVel * 0.25
			end
			
		else
			
			bv.Velocity = Vector3.zero
			
			local x = math.cos(tick()*220)*10
			local z = math.sin(tick()*220)*10
			local y = math.sin(tick()*150)*5
			
			hrp.CFrame = troot.CFrame * CFrame.new(x,y,z)
		end
	end

end)