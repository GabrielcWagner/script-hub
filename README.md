local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Sinners Hub",
   Icon = 0, -- Ícone na barra superior. Pode usar Ícones Lucide (string) ou Imagem Roblox (número). 0 para não usar ícone (padrão).
   LoadingTitle = "Sinners Hub",
   LoadingSubtitle = "by Rennis",
   ShowText = "Rayfield", -- para usuários móveis desocultarem o rayfield, altere se desejar
   Theme = "default", -- Verifique https://docs.sirius.menu/rayfield/configuration/themes

   ToggleUIKeybind = "K", -- A tecla para alternar a visibilidade da UI (string como "K" ou Enum.KeyCode)

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false, -- Impede que o Rayfield avise quando o script tem incompatibilidade de versão com a interface

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil, -- Criar uma pasta personalizada para seu hub/jogo
      FileName = "Sinners Hub"
   },

   Discord = {
      Enabled = false, -- Solicitar ao usuário para entrar no seu servidor Discord se o executor suportar
      Invite = "noinvitelink", -- O código de convite do Discord, não inclua discord.gg/. Ex: discord.gg/ ABCD seria ABCD
      RememberJoins = true -- Defina como false para fazê-los entrar no discord toda vez que carregarem
   },

   KeySystem = true, -- Defina como true para usar nosso sistema de chaves
   KeySettings = {
      Title = "Key hub",
      Subtitle = "Sinners Hub",
      Note = "Para Conseguir a key, procure um sinner", -- Use isso para dizer ao usuário como obter uma chave
      FileName = "Key", -- É recomendado usar algo único pois outros scripts usando Rayfield podem sobrescrever seu arquivo de chave
      SaveKey = false, -- A chave do usuário será salva, mas se você alterar a chave, eles não conseguirão usar seu script
      GrabKeyFromSite = false, -- Se isso for true, defina Key abaixo para o site RAW que você gostaria que o Rayfield obtivesse a chave
      Key = {"sins"} -- Lista de chaves que serão aceitas pelo sistema, pode ser links de arquivos RAW (pastebin, github etc) ou strings simples ("olá","chave22")
   }
})

local MenuTab = Window:CreateTab("Menu🎮")
local JogadorTab = Window:CreateTab("Jogador👩‍💻")

local godModeConnection = nil
local godModeHealthConnection = nil
local godModeStateConnection = nil
local originalMaxHealth = 100

local Toggle = JogadorTab:CreateToggle({
    Name = "God Mode",
    CurrentValue = false,
    Flag = "GodModeToggle",
    Callback = function(Value)
        local player = game.Players.LocalPlayer

        -- Função para limpar conexões
        local function limparConexoes()
            if godModeConnection then
                godModeConnection:Disconnect()
                godModeConnection = nil
            end
            if godModeHealthConnection then
                godModeHealthConnection:Disconnect()
                godModeHealthConnection = nil
            end
            if godModeStateConnection then
                godModeStateConnection:Disconnect()
                godModeStateConnection = nil
            end
        end

        -- Função para ativar o God Mode
        local function ativarGodMode()
            if player and player.Character then
                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    -- Salva a vida máxima original
                    originalMaxHealth = humanoid.MaxHealth
                    
                    -- Define vida máxima e atual para um valor alto mas não infinito
                    local godHealth = 999999
                    humanoid.MaxHealth = godHealth
                    humanoid.Health = godHealth

                    -- Limpa conexões anteriores
                    limparConexoes()

                    -- Monitora mudanças de vida
                    godModeHealthConnection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                        if humanoid.Health < godHealth then
                            humanoid.Health = godHealth
                        end
                    end)

                    -- Monitora mudanças de estado
                    godModeStateConnection = humanoid.StateChanged:Connect(function(_, newState)
                        if newState == Enum.HumanoidStateType.Dead then
                            humanoid.Health = godHealth
                            -- Tenta reabilitar o humanoid
                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                        end
                    end)

                    -- Proteção adicional usando RunService
                    godModeConnection = game:GetService("RunService").Heartbeat:Connect(function()
                        if humanoid.Health < godHealth then
                            humanoid.Health = godHealth
                        end
                        -- Impede estados de morte
                        if humanoid:GetState() == Enum.HumanoidStateType.Dead then
                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                        end
                    end)
                end
            end
        end

        -- Função para desativar o God Mode
        local function desativarGodMode()
            limparConexoes()
            
            if player and player.Character then
                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.MaxHealth = originalMaxHealth
                    if humanoid.Health > originalMaxHealth then
                        humanoid.Health = originalMaxHealth
                    end
                end
            end
        end

        -- Função para reaplicar God Mode após respawn
        local function onCharacterAdded(character)
            wait(1) -- Aguarda o character carregar completamente
            if Value then -- Se o toggle ainda estiver ativo
                ativarGodMode()
            end
        end

        if Value then
            ativarGodMode()
            -- Conecta para reaplicar após respawn
            if player.Character then
                player.CharacterAdded:Connect(onCharacterAdded)
            end
        else
            desativarGodMode()
        end
    end,
})

-- Kill Aura Toggle (Versão Melhorada)
local killAuraMobsConnection = nil
local mobsCache = {}

-- Função melhorada para detectar mobs hostis
local function isHostil(entidade)
    if not entidade or not entidade:FindFirstChild("Humanoid") then
        return false
    end
    
    -- Verifica se é um player
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character == entidade then
            return false
        end
    end
    
    -- Verifica se tem nome de mob comum
    local mobNames = {"Zombie", "Skeleton", "Spider", "Creeper", "Mob", "Enemy", "Monster", "Beast"}
    local name = entidade.Name:lower()
    for _, mobName in pairs(mobNames) do
        if name:find(mobName:lower()) then
            return true
        end
    end
    
    -- Se não for player e tiver Humanoid, assume que é hostil
    return true
end

-- Função para aplicar dano
local function applyDamage(humanoid, damage)
    -- Tenta diferentes métodos de dano
    if humanoid.TakeDamage then
        humanoid:TakeDamage(damage)
    else
        humanoid.Health = math.max(humanoid.Health - damage, 0)
    end
end

-- Atualiza cache de mobs periodicamente
local function updateMobsCache()
    mobsCache = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        if isHostil(obj) and obj:FindFirstChild("HumanoidRootPart") then
          table.insert(mobsCache, obj)
        end
    end
end

local ToggleKillAura = MenuTab:CreateToggle({
    Name = "Kill Aura",
    CurrentValue = false,
    Flag = "KillAuraMobs",
    Callback = function(Value)
        local player = game.Players.LocalPlayer

        if Value then
            -- Desconecta conexão anterior se existir
            if killAuraMobsConnection then
                killAuraMobsConnection:Disconnect()
                killAuraMobsConnection = nil
            end

            -- Atualiza cache inicial
            updateMobsCache()
            
            local lastCacheUpdate = tick()
            local damagePerSecond = 100 -- Dano por segundo
            local range = 50 -- Alcance

            killAuraMobsConnection = game:GetService("RunService").Heartbeat:Connect(function(dt)
                if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local playerPos = player.Character.HumanoidRootPart.Position
                    
                    -- Atualiza cache a cada 0.5 segundos para melhor performance
                    if tick() - lastCacheUpdate > 0.5 then
                        updateMobsCache()
                        lastCacheUpdate = tick()
                    end

                    -- Aplica dano nos mobs próximos
                    for _, mob in pairs(mobsCache) do
                        if mob and mob.Parent and mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") then
                            local dist = (mob.HumanoidRootPart.Position - playerPos).Magnitude
                            if dist <= range then
                                local humanoid = mob.Humanoid
                                if humanoid.Health > 0 then
                                    local damage = damagePerSecond * dt
                                    applyDamage(humanoid, damage)
                                    
                                    -- Debug: print para verificar se está funcionando
                                    print("Aplicando dano:", damage, "em", mob.Name, "Distância:", dist)
                                end
                            end
                        end
                    end
                end
            end)
        else
            if killAuraMobsConnection then
                killAuraMobsConnection:Disconnect()
                killAuraMobsConnection = nil
            end
            mobsCache = {}
        end
    end,
})

-- Inf Jump Toggle (Versão Melhorada)
local infJumpConnection = nil
local infJumpConnection2 = nil
local infJumpActive = false

local InfJumpToggle = JogadorTab:CreateToggle({
    Name = "Inf Jump",
    CurrentValue = false,
    Flag = "InfJumpToggle",
    Callback = function(Value)
        local UserInputService = game:GetService("UserInputService")
        local RunService = game:GetService("RunService")
        local player = game.Players.LocalPlayer

        local function disconnectAll()
            if infJumpConnection then
                infJumpConnection:Disconnect()
                infJumpConnection = nil
            end
            if infJumpConnection2 then
                infJumpConnection2:Disconnect()
                infJumpConnection2 = nil
            end
            infJumpActive = false
        end

        local function canJump()
            if not player or not player.Character then
                return false
            end
            
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if not humanoid then
                return false
            end
            
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if not rootPart then
                return false
            end
            
            -- Verifica se não está morto
            if humanoid:GetState() == Enum.HumanoidStateType.Dead then
                return false
            end
            
            return true
        end

        local function performJump()
            if not canJump() then
                return
            end
            
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            
            -- Método 1: Usar ChangeState
            if humanoid:GetState() ~= Enum.HumanoidStateType.Jumping then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
            
            -- Método 2: Aplicar força vertical (backup)
            if rootPart and rootPart:FindFirstChild("BodyVelocity") then
                rootPart.BodyVelocity.Velocity = Vector3.new(rootPart.BodyVelocity.Velocity.X, 50, rootPart.BodyVelocity.Velocity.Z)
            elseif rootPart then
                -- Cria BodyVelocity temporário se não existir
                local bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
                bodyVelocity.Velocity = Vector3.new(0, 50, 0)
                bodyVelocity.Parent = rootPart
                
                -- Remove após um tempo
                game:GetService("Debris"):AddItem(bodyVelocity, 0.1)
            end
        end

        if Value then
            disconnectAll()
            infJumpActive = true

            -- Método 1: JumpRequest (método original)
            infJumpConnection = UserInputService.JumpRequest:Connect(function()
                performJump()
            end)

            -- Método 2: InputBegan (backup para jogos que não usam JumpRequest)
            infJumpConnection2 = UserInputService.InputBegan:Connect(function(input, gameProcessed)
                if gameProcessed then return end
                
                if input.KeyCode == Enum.KeyCode.Space then
                    performJump()
                end
            end)
        else
            disconnectAll()
        end
    end,
})

local noclipConnection = nil

local Toggle = MenuTab:CreateToggle({
   Name = "No Clip",
   CurrentValue = false,
   Flag = "NoClipToggle",
   Callback = function(Value)
      local player = game.Players.LocalPlayer
      local RunService = game:GetService("RunService")

      local function setNoClip(state)
         if player and player.Character then
            for _, part in ipairs(player.Character:GetDescendants()) do
               if part:IsA("BasePart") then
                  part.CanCollide = not state
               end
            end
         end
      end

      if Value then
         -- Ativa o NoClip
         setNoClip(true)
         if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
         end
         noclipConnection = RunService.Stepped:Connect(function()
            setNoClip(true)
         end)
      else
         -- Desativa o NoClip
         if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
         end
         setNoClip(false)
      end
   end,
})


-- WalkSpeed Slider (Com Proteção Extra)
local walkSpeedConnection = nil
local characterConnection = nil
local animationConnection = nil
local currentWalkSpeed = 16

local WalkSpeedSlider = JogadorTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {5, 200},
    Increment = 5,
    Suffix = "WalkSpeed",
    CurrentValue = 16,
    Flag = "WalkSpeedSlider",
    Callback = function(Value)
        local player = game.Players.LocalPlayer
        currentWalkSpeed = Value
        
        -- Limpa conexões anteriores
        if walkSpeedConnection then
            walkSpeedConnection:Disconnect()
        end
        if characterConnection then
            characterConnection:Disconnect()
        end
        if animationConnection then
            animationConnection:Disconnect()
        end
        
        -- Função para aplicar WalkSpeed
        local function applyWalkSpeed()
            if player and player.Character and player.Character:FindFirstChild("Humanoid") then
                player.Character.Humanoid.WalkSpeed = currentWalkSpeed
            end
        end
        
        -- Aplica imediatamente
        applyWalkSpeed()
        
        -- Monitora mudanças no character (respawn)
        characterConnection = player.CharacterAdded:Connect(function(character)
            wait(2) -- Aguarda carregamento completo
            applyWalkSpeed()
            
            -- Monitora animações que podem afetar WalkSpeed
            local humanoid = character:FindFirstChild("Humanoid")
            if humanoid then
                animationConnection = humanoid.AnimationPlayed:Connect(function(animation)
                    -- Reaplica WalkSpeed após animações
                    wait(0.1)
                    applyWalkSpeed()
                end)
            end
        end)
        
        -- Monitora continuamente para manter o valor
        walkSpeedConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if player and player.Character and player.Character:FindFirstChild("Humanoid") then
                local humanoid = player.Character.Humanoid
                if humanoid.WalkSpeed ~= currentWalkSpeed then
                    humanoid.WalkSpeed = currentWalkSpeed
                end
            end
        end)
    end,
})

-- Jump Power Slider (Com Proteção Total)
local jumpPowerConnection = nil
local characterConnection = nil
local currentJumpPower = 50

local JumpPowerSlider = JogadorTab:CreateSlider({
    Name = "Jump Power",
    Range = {0, 500},
    Increment = 5,
    Suffix = "JumpPower",
    CurrentValue = 50,
    Flag = "JumpPowerSlider",
    Callback = function(Value)
        local player = game.Players.LocalPlayer
        currentJumpPower = Value
        
        -- Limpa conexões anteriores
        if jumpPowerConnection then
            jumpPowerConnection:Disconnect()
        end
        if characterConnection then
            characterConnection:Disconnect()
        end
        
        -- Função para aplicar JumpPower
        local function applyJumpPower()
            if player and player.Character and player.Character:FindFirstChild("Humanoid") then
                player.Character.Humanoid.JumpPower = currentJumpPower
            end
        end
        
        -- Aplica imediatamente
        applyJumpPower()
        
        -- Monitora mudanças no character (respawn)
        characterConnection = player.CharacterAdded:Connect(function(character)
            wait(2) -- Aguarda carregamento completo
            applyJumpPower()
        end)
        
        -- Monitora continuamente para manter o valor
        jumpPowerConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if player and player.Character and player.Character:FindFirstChild("Humanoid") then
                local humanoid = player.Character.Humanoid
                if humanoid.JumpPower ~= currentJumpPower then
                    humanoid.JumpPower = currentJumpPower
                end
            end
        end)
    end,
})
