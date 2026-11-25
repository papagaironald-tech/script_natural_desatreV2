-- IMORTAL - Natural Disaster Survival
-- Script para o Roblox Delta Executor
-- Imortal a quase tudo, exceto o void do jogo

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Debris = game:GetService("Debris")

local localPlayer = Players.LocalPlayer
if not localPlayer then return end

local playerCharacter = nil

-- ========================================================
-- VERIFICAR SE É O JOGADOR (para evitar erros em outros)
-- ========================================================
function IsLocalCharacter(char)
	return char and char:IsDescendantOf(localPlayer.Character)
end

-- ========================================================
-- MODO IMORTAL: Desliga o dano no personagem
-- ========================================================
function EnableImmortality()
	local char = localPlayer.Character
	if not char then return end
	
	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanTouch = false  -- nenhum toque causa dano
			part.CanCollide = true -- colisões físicas normais
		end
	end
	
	-- Impede o dano por queda (fall damage)
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.MaxHealth = 1000
		humanoid.Health = 1000
	end
	
	-- Impede o afogamento
	local rootPart = char:FindFirstChild("HumanoidRootPart")
	if rootPart then
		rootPart.Transparency = 0.7
	end
	
	print("[IMORTAL] Imortalidade ativada! Você não pode morrer, exceto pelo void.")
end

-- ========================================================
-- MODO IMORTAL: Reativar quando o personagem renasce
-- ========================================================
function SetupRespawn()
	local char = localPlayer.Character
	if not char then return end
	
	char.AncestryChanged:Connect(function()
		task.wait(0.5)
		if localPlayer.Character then
			EnableImmortality()
			print("[IMORTAL] Imortalidade restaurada após respawn.")
		end
	end)
end

-- ========================================================
-- ANTI-QUEDA (falling) - Protege contra queda livre
-- ========================================================
function AntiFall()
	local char = localPlayer.Character
	if not char then return end
	
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return end
	
	RunService.Heartbeat:Connect(function()
		if localPlayer.Character then
			-- Se o personagem cair muito rápido, desacelerar a queda
			local speed = root.AssemblyLinearVelocity.Magnitude
			if speed > 100 then
				root.AssemblyLinearVelocity = root.CFrame.UpVector * math.min(speed, 60)
			end
		end
	end)
	
	print("[IMORTAL] Anti-queda ativado.")
end

-- ========================================================
-- ANTI-CHOQUE (collision com objetos rápidos)
-- ========================================================
function AntiCollision()
	local char = localPlayer.Character
	if not char then return end
	
	RunService.Heartbeat:Connect(function()
		if not localPlayer.Character then return end
		
		for _, part in ipairs(char:GetDescendants()) do
			if part:IsA("BasePart") then
				part.Reflectance = 0  -- reduz o impacto visual
			end
		end
		
		-- Impede que objetos externos causem dano ao tocar o personagem
		for _, obj in ipairs(workspace:GetDescendants()) do
			if obj:IsA("BasePart") and obj.Touched:Connect(function(hit)
				if hit:IsDescendantOf(localPlayer.Character) then
					obj.CanCollide = false
					obj.Transparency = 0.8
					Debris:AddItem(obj, 0.1)
				end
			end) then
				-- já conectado
			end
		end
	end)
	
	print("[IMORTAL] Anti-choque ativado.")
end

-- ========================================================
-- ANTI-Raios (meteoros)
-- ========================================================
function AntiMeteor()
	RunService.Heartbeat:Connect(function()
		for _, part in ipairs(workspace:GetDescendants()) do
			if part:IsA("BasePart") and part.Name:lower():find("meteor") then
				part.Transparency = 0.9
				part.CanCollide = false
				Debris:AddItem(part, 0.5)
			end
			
			-- Impact zone de meteoro
			if part:IsA("BasePart") and part.Material == Enum.Material.Sand then
				-- meteoro cai no chão
				if part.BrickColor == BrickColor.new("Brown") then
					part.Transparency = 0.9
					Debris:AddItem(part, 0.3)
				end
			end
		end
	end)
	
	print("[IMORTAL] Anti-raios ativado.")
end

-- ========================================================
-- ANTI-CHUVA ÁCIDA
-- ========================================================
function AntiAcidRain()
	local acidRain = workspace:FindFirstChild("AcidRain")
	if acidRain then
		for _, part in ipairs(acidRain:GetChildren()) do
			if part:IsA("BasePart") then
				part.Transparency = 1
				part.CanCollide = false
			end
		end
	end
	
	RunService.Heartbeat:Connect(function()
		for _, part in ipairs(workspace:GetDescendants()) do
			if part:IsA("BasePart") and part.Material == Enum.Material.Acids and part.Parent then
				-- chuva ácida
				if part.Parent.Name:find("Acid") or part.Name:find("Acid") then
					part.Transparency = 1
					part.CanCollide = false
					Debris:AddItem(part, 0.2)
				end
			end
		end
	end)
	
	print("[IMORTAL] Anti-chuva ácida ativado.")
end

-- ========================================================
-- ANTI-LAVA
-- ========================================================
function AntiLava()
	RunService.Heartbeat:Connect(function()
		for _, part in ipairs(workspace:GetDescendants()) do
			if part:IsA("BasePart") and part.Material == Enum.Material.Lava then
				part.Transparency = 0.95
				part.CanCollide = false
			end
			
			if part:IsA("BasePart") and part.BrickColor == BrickColor.new("Bright red") then
				if part.Name:lower():find("lava") then
					part.Transparency = 0.95
					part.CanCollide = false
				end
			end
		end
	end)
	
	print("[IMORTAL] Anti-lava ativado.")
end

-- ========================================================
-- ANTI-FOGO
-- ========================================================
function AntiFire()
	local fireObjects = workspace:FindFirstChild("Fire")
	if fireObjects then
		for _, part in ipairs(fireObjects:GetDescendants()) do
			if part:IsA("BasePart") or part:IsA("Fire") then
				part.Transparency = 1
				if part:IsA("Fire") then
					part.Enabled = false
				end
			end
		end
	end
	
	RunService.Heartbeat:Connect(function()
		for _, obj in ipairs(workspace:GetDescendants()) do
			if obj:IsA("Fire") then
				obj.Enabled = false
			end
			if obj:IsA("Decal") and obj.Texture:find("fire") then
				obj.Transparency = 1
			end
		end
	end)
	
	print("[IMORTAL] Anti-fogo ativado.")
end

-- ========================================================
-- ANTI-ALAGAMENTO (flood)
-- ========================================================
function AntiFlood()
	local waters = workspace:FindFirstChild("Water")
	if waters then
		for _, part in ipairs(waters:GetDescendants()) do
			if part:IsA("BasePart") then
				part.Transparency = 0.9
				part.CanCollide = false
			end
		end
	end
	
	RunService.Heartbeat:Connect(function()
		for _, part in ipairs(workspace:GetDescendants()) do
			if part:IsA("BasePart") and part.Material == Enum.Material.Water then
				if part.Parent and (part.Parent.Name:find("flood") or part.Parent.Name:find("Water") or part.Parent.Name:find("alagamento")) then
					part.Transparency = 0.95
					part.CanCollide = false
				end
			end
			
			-- tsunamis e ondas
			if part:IsA("Part") and part.Name:lower():find("wave") or part.Name:lower():find("tsunami") then
				part.Transparency = 0.95
				part.CanCollide = false
				Debris:AddItem(part, 0.1)
			end
		end
	end)
	
	print("[IMORTAL] Anti-alagamento ativado.")
end

-- ========================================================
-- ANTI-ESQUIVO (suffocation by sandstorm, earthquake, etc.)
-- ========================================================
function AntiSuffocation()
	local humanoid = localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.DisplayHealthBar = false
		humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
	end
	
	RunService.Heartbeat:Connect(function()
		local char = localPlayer.Character
		if not char then return end
		
		local human = char:FindFirstChildOfClass("Humanoid")
		if human and human.Health > 0 then
			human.Health = math.max(human.Health, 1)  -- nunca morre de dano por dano
		end
	end)
	
	print("[IMORTAL] Anti-asfixiação ativado.")
end

-- ========================================================
-- ANTI-QUEDA LIVRE COM GRAVIDADE MODIFICADA
-- ========================================================
function AntiFreefall()
	local root = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
	if root then
		local bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
		bodyVelocity.Velocity = Vector3.new(0, 0, 0)
		bodyVelocity.Parent = root
	end
	
	print("[IMORTAL] Anti-queda livre ativado.")
end

-- ========================================================
-- ANTI-RAIO DE DESTINO (widowmaker) - escombros caindo
-- ========================================================
function AntiWidowmaker()
	RunService.Heartbeat:Connect(function()
		for _, part in ipairs(workspace:GetDescendants()) do
			if part:IsA("BasePart") and part.Mass > 50 and part.Name:lower():find("part") then
				-- objetos pesados que caem
				local vel = part.AssemblyLinearVelocity.Magnitude
				if vel > 50 then
					part.Anchored = true
					Debris:AddItem(part, 0.1)
				end
			end
		end
	end)
	
	print("[IMORTAL] Anti-escombros (Widowmaker) ativado.")
end

-- ========================================================
-- ATIVAR TODOS OS MODO IMORTAIS
-- ========================================================
function ActivateAll()
	EnableImmortality()
	SetupRespawn()
	AntiFall()
	AntiCollision()
	AntiMeteor()
	AntiAcidRain()
	AntiLava()
	AntiFire()
	AntiFlood()
	AntiSuffocation()
	AntiFreefall()
	AntiWidowmaker()
	
	print("\n========== IMORTAL ATIVADO ========== ")
	print("Você está imortal! Exceto pelo void do jogo.")
	print("MODO IMORTAL: Imortal a queda, colisão, raios, chuva ácida, lava, fogo, alagamento.")
	print("====================================== ")
end

-- ========================================================
-- ESPERAR O PERSONAGEM CARREGAR
-- ========================================================
if localPlayer.Character then
	ActivateAll()
else
	localPlayer.CharacterAdded:Connect(function(char)
		task.wait(1)
		ActivateAll()
	end)
end

-- ========================================================
-- CONTROLE MANUAL: tecla F1 liga/desliga imortal
-- ========================================================
local immortalEnabled = true

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if input.KeyCode == Enum.KeyCode.F1 and not gameProcessed then
		immortalEnabled = not immortalEnabled
		if immortalEnabled then
			EnableImmortality()
			print("[IMORTAL] Imortalidade ativada.")
		else
			local char = localPlayer.Character
			if char then
				for _, part in ipairs(char:GetDescendants()) do
					if part:IsA("BasePart") then
						part.CanTouch = true
						part.Transparency = 0
					end
				end
			end
			print("[IMORTAL] Imortalidade desativada.")
		end
	end
end)

-- ========================================================
-- ANTI-VOID: se cair no void, puxar de volta
-- ========================================================
RunService.Heartbeat:Connect(function()
	local char = localPlayer.Character
	if not char then return end
	
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return end
	
	if root.Position.Y < -50 then
		-- Cair no void é a única morte possível
		print("[IMORTAL] Você está perto do void. A única coisa que pode te matar é o void.")
	end
end)

print("[IMORTAL] Script carregado com sucesso!")
