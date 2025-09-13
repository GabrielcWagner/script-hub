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
      SaveKey = true, -- A chave do usuário será salva, mas se você alterar a chave, eles não conseguirão usar seu script
      GrabKeyFromSite = false, -- Se isso for true, defina Key abaixo para o site RAW que você gostaria que o Rayfield obtivesse a chave
      Key = {"sins"} -- Lista de chaves que serão aceitas pelo sistema, pode ser links de arquivos RAW (pastebin, github etc) ou strings simples ("olá","chave22")
   }
})

local MenuTab = Window:CreateTab("Menu🎮")
local JogadorTab = Window:CreateTab("Jogador👩‍💻")

-- God Mode Button
local GodModeButton = JogadorTab:CreateButton({ 
   Name = "God Mode",
   Callback = function()
      local player = game.Players.LocalPlayer
      if player and player.Character then
         local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
         if humanoid then
            -- Torna o jogador imortal
            humanoid.MaxHealth = math.huge
            humanoid.Health = math.huge
            
            -- Mantém a vida sempre no máximo
            humanoid.HealthChanged:Connect(function()
               if humanoid.Health < math.huge then
                  humanoid.Health = math.huge
               end
            end)
            
            -- Impede morte
            humanoid.Died:Connect(function()
               humanoid.Health = math.huge
            end)
         end
      end
   end,
})

-- Kill Aura Toggle
local killAuraMobsConnection = nil

local ToggleKillAura = MenuTab:CreateToggle({
   Name = "Kill Aura",
   CurrentValue = false,
   Flag = "KillAuraMobs",
   Callback = function(Value)
      local player = game.Players.LocalPlayer

      local function isHostil(entidade)
         if entidade and entidade:FindFirstChild("Humanoid") then
            for _, p in pairs(game.Players:GetPlayers()) do
               if p.Character == entidade then
                  return false
               end
            end
            return true
         end
         return false
      end

      if Value then
         -- Desconecta caso já esteja ativo para evitar múltiplas conexões
         if killAuraMobsConnection then
            killAuraMobsConnection:Disconnect()
            killAuraMobsConnection = nil
         end

         killAuraMobsConnection = game:GetService("RunService").Heartbeat:Connect(function(dt)
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
               local playerPos = player.Character.HumanoidRootPart.Position

               -- Dica: Se seus mobs estiverem em uma pasta específica, substitua workspace:GetDescendants() por workspace.Mobs:GetChildren()
               for _, obj in pairs(workspace:GetDescendants()) do
                  if isHostil(obj) and obj:FindFirstChild("HumanoidRootPart") then
                     local dist = (obj.HumanoidRootPart.Position - playerPos).Magnitude
                     if dist <= 50 then
                        local humanoid = obj.Humanoid
                        if humanoid.Health > 0 then
                           humanoid.Health = math.max(humanoid.Health - (50 * dt), 0)
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
      end
   end,
})

-- Inf Jump Toggle
local infJumpConnection = nil
local infJumpConnection2 = nil
local infJumpHeartbeat = nil
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
         if infJumpHeartbeat then
            infJumpHeartbeat:Disconnect()
            infJumpHeartbeat = nil
         end
         infJumpActive = false
      end

      if Value then
         disconnectAll() -- Garante que não vai duplicar eventos

         infJumpConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == Enum.KeyCode.Space then
               infJumpActive = true
            end
         end)
         infJumpConnection2 = UserInputService.InputEnded:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == Enum.KeyCode.Space then
               infJumpActive = false
            end
         end)
         infJumpHeartbeat = RunService.RenderStepped:Connect(function()
            if infJumpActive and player and player.Character and player.Character:FindFirstChild("Humanoid") then
               local humanoid = player.Character.Humanoid
               if humanoid.Health > 0 then
                  humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
               end
            end
         end)
      else
         disconnectAll()
      end
   end,
})

-- WalkSpeed Slider
local WalkSpeedSlider = JogadorTab:CreateSlider({
   Name = "WalkSpeed",
   Range = {5, 200},
   Increment = 5,
   Suffix = "WalkSpeed",
   CurrentValue = 16,
   Flag = "WalkSpeedSlider",
   Callback = function(Value)
      local player = game.Players.LocalPlayer
      if player and player.Character and player.Character:FindFirstChild("Humanoid") then
         player.Character.Humanoid.WalkSpeed = Value
      end
   end,
})

-- Jump Power Slider
local JumpPowerSlider = JogadorTab:CreateSlider({
   Name = "Jump Power",
   Range = {0, 500},
   Increment = 5,
   Suffix = "JumpPower",
   CurrentValue = 50,
   Flag = "JumpPowerSlider",
   Callback = function(Value)
      local player = game.Players.LocalPlayer
      if player and player.Character and player.Character:FindFirstChild("Humanoid") then
         player.Character.Humanoid.JumpPower = Value
      end
   end,
})
