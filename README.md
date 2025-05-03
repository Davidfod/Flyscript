-- Configs
local flyspeed = 100
local controls = {
	front = "w",
	back = "s",
	right = "d",
	left = "a",
	up = " ",
	down = "q"
}
-- Fim Configs

local player = game:GetService("Players").LocalPlayer
local mouse = player:GetMouse()
local runservice = game:GetService("RunService")

local flycontrol = {F = 0, R = 0, B = 0, L = 0, U = 0, D = 0}
local flying = false

-- Criação da GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "FlyControlGui"

-- Frame acima do botão
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 100, 0, 30)
frame.Position = UDim2.new(1, -110, 1, -90) -- acima do botão
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.Parent = screenGui

-- Texto "Cyber"
local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.Text = "Cyber'Scripts"
label.TextColor3 = Color3.new(1, 1, 1)
label.TextScaled = true
label.Font = Enum.Font.GothamBold
label.Parent = frame

-- Botão de voo
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 100, 0, 40)
button.Position = UDim2.new(1, -110, 1, -50)
button.AnchorPoint = Vector2.new(0, 0)
button.BackgroundColor3 = Color3.fromRGB(30, 144, 255)
button.Text = "Fly"
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.Gotham
button.TextScaled = true
button.Parent = screenGui

-- Função de voo
local function fly()
	local character = player.Character
	if not character then return end
	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	local humanoid = character:FindFirstChildWhichIsA("Humanoid")
	if not humanoid then return end

	flying = true

	local bv = Instance.new("BodyVelocity")
	local bg = Instance.new("BodyGyro")
	bv.MaxForce = Vector3.new(9e4, 9e4, 9e4)
	bg.CFrame = hrp.CFrame
	bg.MaxTorque = Vector3.new(9e4, 9e4, 9e4)
	bg.P = 9e4
	bv.Parent = hrp
	bg.Parent = hrp

	for _, child in pairs(character:GetDescendants()) do
		if child:IsA("BasePart") then
			coroutine.wrap(function()
				local con = nil
				con = runservice.Stepped:Connect(function()
					if not flying then
						con:Disconnect()
						child.CanCollide = true
					end
					child.CanCollide = false
				end)
			end)()
		end
	end

	local con = nil
	con = runservice.Stepped:Connect(function()
		if not flying then
			con:Disconnect()
			bv:Destroy()
			bg:Destroy()
		end

		humanoid.PlatformStand = true
		bv.Velocity =
			(workspace.CurrentCamera.CFrame.LookVector * ((flycontrol.F - flycontrol.B) * flyspeed)) +
			(workspace.CurrentCamera.CFrame.RightVector * ((flycontrol.R - flycontrol.L) * flyspeed)) +
			(workspace.CurrentCamera.CFrame.UpVector * ((flycontrol.U - flycontrol.D) * flyspeed))

		bg.CFrame = workspace.CurrentCamera.CFrame
	end)

	repeat task.wait() until not flying

	while humanoid.PlatformStand == true do
		humanoid.PlatformStand = false
		task.wait()
	end
end

-- Controles de direção (teclado)
mouse.KeyDown:Connect(function(key)
	if key:lower() == controls.front then
		flycontrol.F = 1
	elseif key:lower() == controls.back then
		flycontrol.B = 1
	elseif key:lower() == controls.right then
		flycontrol.R = 1
	elseif key:lower() == controls.left then
		flycontrol.L = 1
	elseif key:lower() == controls.up then
		flycontrol.U = 1
	elseif key:lower() == controls.down then
		flycontrol.D = 1
	end
end)

mouse.KeyUp:Connect(function(key)
	if key:lower() == controls.front then
		flycontrol.F = 0
	elseif key:lower() == controls.back then
		flycontrol.B = 0
	elseif key:lower() == controls.right then
		flycontrol.R = 0
	elseif key:lower() == controls.left then
		flycontrol.L = 0
	elseif key:lower() == controls.up then
		flycontrol.U = 0
	elseif key:lower() == controls.down then
		flycontrol.D = 0
	end
end)

-- Botão de ativar/desativar voo
button.MouseButton1Click:Connect(function()
	if flying then
		flying = false
		button.Text = "Fly"
	else
		fly()
		button.Text = "Fly off"
	end
end)

-- Reset do personagem
player.CharacterAdded:Connect(function()
	flying = false
	if button then button.Text = "Fly" end
end)
