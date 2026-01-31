--// INTRO - LOGO ONLY + SOUND GRADIENT
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local ProximityPromptService = game:GetService("ProximityPromptService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

--------------------------------------------------
-- VARIÁVEIS GLOBAIS
--------------------------------------------------
local Character, HRP, Humanoid
local SpeedEnabled, SpeedValue = false, 30
local SpeedEnabled2, SpeedValue2 = false, 60  -- NOVO SPEED
local JumpEnabled, JumpValue = false, 50
local AntiLagEnabled = false
local AutoGrabEnabled = false
local AutoGrabScript = nil
local XRayEnabled = false
local XRayConnection = nil
local AutoBatEnabled = false
local AntiRagdollEnabled = false

-- Configurações
local TARGET_TOOL = "Flying Carpet"
local XRAY_TRANSPARENCY = 0.7
local XRAY_BASE_NAME = "Base"

--------------------------------------------------
-- INTRO
--------------------------------------------------
local SplashGui = Instance.new("ScreenGui")
SplashGui.Name = "FluxoIntro"
SplashGui.IgnoreGuiInset = true
SplashGui.ResetOnSpawn = false
SplashGui.Parent = PlayerGui

local Logo = Instance.new("ImageLabel")
Logo.Parent = SplashGui
Logo.AnchorPoint = Vector2.new(0.5, 0.5)
Logo.Position = UDim2.fromScale(0.5, 0.5)
Logo.Size = UDim2.fromScale(0.28, 0.28)
Logo.BackgroundTransparency = 1
Logo.Image = "rbxassetid://132564087673178"
Logo.ImageTransparency = 1

local Sound = Instance.new("Sound")
Sound.Parent = SplashGui
Sound.SoundId = "rbxassetid://184352963"
Sound.Volume = 0
Sound.Looped = false

Sound:Play()
TweenService:Create(Logo, TweenInfo.new(0.8), {ImageTransparency = 0.25}):Play()
TweenService:Create(Sound, TweenInfo.new(1.2), {Volume = 1}):Play()
task.wait(2.3)

TweenService:Create(Logo, TweenInfo.new(0.6), {ImageTransparency = 1}):Play()
TweenService:Create(Sound, TweenInfo.new(0.6), {Volume = 0}):Play()
task.wait(0.8)
SplashGui:Destroy()

--------------------------------------------------
-- SETUP DO PERSONAGEM
--------------------------------------------------
local function SetupCharacter(char)
    Character = char
    HRP = char:WaitForChild("HumanoidRootPart")
    Humanoid = char:WaitForChild("Humanoid")
end

if player.Character then
    SetupCharacter(player.Character)
end
player.CharacterAdded:Connect(SetupCharacter)

--------------------------------------------------
-- FUNÇÕES DAS FEATURES
--------------------------------------------------
-- ANTI LAG
local function AtivarAntiLag()
    print("🧊 Anti Lag ATIVADO")
    
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    Lighting.EnvironmentDiffuseScale = 0
    Lighting.EnvironmentSpecularScale = 0

    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            obj.Material = Enum.Material.Plastic
            obj.CastShadow = false
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") then
            obj.Enabled = false
        end
    end
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Anti Lag ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

local function DesativarAntiLag()
    print("🧊 Anti Lag DESATIVADO")
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level21
    Lighting.GlobalShadows = true
    Lighting.FogEnd = 100000
    Lighting.EnvironmentDiffuseScale = 1
    Lighting.EnvironmentSpecularScale = 1
end

-- AUTO GRAB AVANÇADO
local function StartAdvancedAutoGrab()
    if AutoGrabScript then return end
    
    print("🔥 Iniciando Auto Grab Avançado...")
    
    AutoGrabScript = {
        Running = true,
        Connections = {}
    }
    
    local isTeleporting = false
    
    local function onPromptAdded(prompt)
        if not AutoGrabScript or not AutoGrabScript.Running then return end
        
        local combinedText = (prompt.ActionText .. prompt.ObjectText):lower()
        
        if combinedText:find("steal") or combinedText:find("take") or combinedText:find("grab") then
            task.spawn(function()
                local isHolding = false
                
                while prompt and prompt.Parent and AutoGrabScript and AutoGrabScript.Running do
                    task.wait(0.1)
                    
                    if isTeleporting then 
                        if isHolding then 
                            pcall(function() prompt:InputHoldEnd() end)
                            isHolding = false 
                        end
                        continue 
                    end

                    local function getPromptLocation(promptObj)
                        if not promptObj or not promptObj.Parent then return nil end
                        if promptObj.Parent:IsA("Attachment") then
                            return promptObj.Parent.WorldPosition
                        elseif promptObj.Parent:IsA("BasePart") then
                            return promptObj.Parent.Position
                        elseif promptObj.Parent:IsA("Model") then
                            return promptObj.Parent:GetPivot().Position
                        end
                        return nil
                    end

                    local promptPos = getPromptLocation(prompt)
                    if not promptPos then continue end
                    
                    local dist = player:DistanceFromCharacter(promptPos)
                    local maxDist = prompt.MaxActivationDistance or 10
                    
                    if dist <= maxDist then
                        if not isHolding then
                            local function equipToolNow()
                                local char = player.Character
                                if not char then return false end
                                
                                if char:FindFirstChild(TARGET_TOOL) then 
                                    return true 
                                end
                                
                                local backpack = player:FindFirstChild("Backpack")
                                if backpack then
                                    local tool = backpack:FindFirstChild(TARGET_TOOL)
                                    if tool then 
                                        tool.Parent = char
                                        task.wait(0.1)
                                        return true 
                                    end
                                end
                                return false
                            end
                            
                            equipToolNow()
                            task.wait(0.1)
                            pcall(function() 
                                prompt:InputHoldBegin() 
                                isHolding = true
                            end)
                        end
                    else
                        if isHolding then
                            pcall(function() prompt:InputHoldEnd() end)
                            isHolding = false
                        end
                    end
                end
            end)
        end
    end

    for _, p in pairs(Workspace:GetDescendants()) do
        if p:IsA("ProximityPrompt") then 
            onPromptAdded(p) 
        end
    end

    local promptConnection = ProximityPromptService.PromptShown:Connect(function(prompt)
        onPromptAdded(prompt)
    end)
    
    table.insert(AutoGrabScript.Connections, promptConnection)
    
    local workspaceConnection = Workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("ProximityPrompt") then
            onPromptAdded(descendant)
        end
    end)
    
    table.insert(AutoGrabScript.Connections, workspaceConnection)
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Auto Grab ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

local function StopAdvancedAutoGrab()
    if not AutoGrabScript then return end
    
    print("🛑 Parando Auto Grab Avançado...")
    
    for _, connection in ipairs(AutoGrabScript.Connections) do
        pcall(function() connection:Disconnect() end)
    end
    
    AutoGrabScript.Running = false
    AutoGrabScript = nil
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Auto Grab DESATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

-- X-RAY (BASES TRANSPARENTES)
local function AplicarTransparenciaXRay()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name:lower():find(XRAY_BASE_NAME:lower()) then
            v.LocalTransparencyModifier = XRAY_TRANSPARENCY
        end
    end
end

local function RemoverTransparenciaXRay()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name:lower():find(XRAY_BASE_NAME:lower()) then
            v.LocalTransparencyModifier = 0
        end
    end
end

local function AtivarXRay()
    if XRayEnabled then return end
    
    print("👁️ X-Ray ATIVADO")
    XRayEnabled = true
    
    AplicarTransparenciaXRay()
    
    XRayConnection = Workspace.DescendantAdded:Connect(function(obj)
        task.wait(0.1)
        if XRayEnabled and obj:IsA("BasePart") and obj.Name:lower():find(XRAY_BASE_NAME:lower()) then
            obj.LocalTransparencyModifier = XRAY_TRANSPARENCY
        end
    end)
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "X-Ray ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
    
    -- Reconectar transparência quando voltar ao jogo
    UserInputService.WindowFocused:Connect(function()
        if not XRayEnabled then return end
        task.wait(0.5)
        AplicarTransparenciaXRay()
    end)
end

local function DesativarXRay()
    if not XRayEnabled then return end
    
    print("👁️ X-Ray DESATIVADO")
    XRayEnabled = false
    
    RemoverTransparenciaXRay()
    
    if XRayConnection then
        XRayConnection:Disconnect()
        XRayConnection = nil
    end
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "X-Ray DESATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

--------------------------------------------------
-- ANTI RAGDOLL SISTEMA
--------------------------------------------------
local PlayerModule = require(player:WaitForChild("PlayerScripts"):WaitForChild("PlayerModule"))
local Controls = PlayerModule:GetControls()

local ENABLE_ANTI_RAGDOLL = false
local Frozen = false
local active = false
local connections = {}

local character, humanoid, hrp

local BlockedStates = {
    [Enum.HumanoidStateType.Ragdoll] = true,
    [Enum.HumanoidStateType.FallingDown] = true,
    [Enum.HumanoidStateType.Physics] = true,
    [Enum.HumanoidStateType.Dead] = true
}

local function updateCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoid = character:FindFirstChildOfClass("Humanoid")
    hrp = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("RootPart") or character:FindFirstChild("HumanoidRoot")
end

local function isValid()
    return humanoid and humanoid.Parent ~= nil
end

local function blocklist()
    if not humanoid then return false end
    local state = humanoid:GetState()
    if state == Enum.HumanoidStateType.Jumping or state == Enum.HumanoidStateType.Freefall or state == Enum.HumanoidStateType.Landed then
        return false
    end
    if state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown or state == Enum.HumanoidStateType.GettingUp then
        return true
    end
    return false
end

local function enableControls()
    pcall(function()
        local pm = player:WaitForChild("PlayerScripts"):WaitForChild("PlayerModule")
        require(pm):GetControls():Enable()
    end)
end

local function cleanCharacter()
    if not blocklist() then return end
    if not character then return end
    for _, desc in pairs(character:GetDescendants()) do
        if desc:IsA("BallSocketConstraint") or desc:IsA("NoCollisionConstraint") or desc:IsA("HingeConstraint") or (desc:IsA("Attachment") and (desc.Name == "A" or desc.Name == "B")) then
            pcall(function() desc:Destroy() end)
        elseif desc:IsA("BodyVelocity") or desc:IsA("BodyPosition") or desc:IsA("BodyGyro") then
            pcall(function() desc:Destroy() end)
        elseif desc:IsA("Motor6D") then
            pcall(function() desc.Enabled = true end)
        end
    end
end

local function forceNormal(char)
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    hum.Health = hum.MaxHealth
    hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
    if not Frozen then
        Frozen = true
        root.Anchored = true
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        root.CFrame += Vector3.new(0, 1.5, 0)
    end
end

local function release(char)
    local root = char:FindFirstChild("HumanoidRootPart")
    if root and Frozen then
        root.Anchored = false
        Frozen = false
    end
end

local function restoreMotors(char)
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("Motor6D") then
            v.Enabled = true
        elseif v:IsA("Constraint") then
            v.Enabled = false
        end
    end
end

local function initAntiRagdoll(char)
    local hum = char:WaitForChild("Humanoid", 10)
    if not hum then return end
    for state in pairs(BlockedStates) do
        hum:SetStateEnabled(state, false)
    end
    hum.StateChanged:Connect(function(_, new)
        if ENABLE_ANTI_RAGDOLL and BlockedStates[new] then
            forceNormal(char)
            restoreMotors(char)
        end
    end)
    RunService.Stepped:Connect(function()
        if not ENABLE_ANTI_RAGDOLL then
            release(char)
            return
        end
        if BlockedStates[hum:GetState()] then
            forceNormal(char)
        else
            release(char)
        end
        hum.Health = hum.MaxHealth
    end)
end

local function enableAntiKB()
    if active then return end
    active = true
    updateCharacter()
    if not isValid() then return end
    if humanoid then
        table.insert(connections, humanoid.StateChanged:Connect(function()
            if blocklist() then
                pcall(function() humanoid:ChangeState(Enum.HumanoidStateType.Running) end)
                cleanCharacter()
                pcall(function() Workspace.CurrentCamera.CameraSubject = humanoid end)
                enableControls()
            end
        end))
    end
    local pkg = ReplicatedStorage:FindFirstChild("Packages")
    if pkg then
        local net = pkg:FindFirstChild("Net")
        if net then
            local ev = net:FindFirstChild("RE/CombatService/ApplyImpulse") or net:FindFirstChild("ApplyImpulse")
            if ev and ev:IsA("RemoteEvent") then
                table.insert(connections, ev.OnClientEvent:Connect(function()
                    if blocklist() and hrp then
                        pcall(function() hrp.AssemblyLinearVelocity = Vector3.new(0,0,0) end)
                    end
                end))
            end
        end
    end
    if character then
        table.insert(connections, character.DescendantAdded:Connect(function()
            if blocklist() then cleanCharacter() end
        end))
    end
    local lastVelocity = Vector3.new(0,0,0)
    table.insert(connections, RunService.Heartbeat:Connect(function()
        if blocklist() and hrp then
            cleanCharacter()
            local currentVel = hrp.AssemblyLinearVelocity
            if (currentVel - lastVelocity).Magnitude > 40 and currentVel.Magnitude > 25 then
                pcall(function() hrp.AssemblyLinearVelocity = currentVel.Unit * math.min(currentVel.Magnitude, 15) end)
            end
            lastVelocity = currentVel
        else
            if hrp then lastVelocity = hrp.AssemblyLinearVelocity end
        end
    end))
    table.insert(connections, player.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        updateCharacter()
        enableControls()
        cleanCharacter()
    end))
    enableControls()
    cleanCharacter()
end

local function disableAntiKB()
    if not active then return end
    active = false
    for _, conn in ipairs(connections) do
        if conn then pcall(function() conn:Disconnect() end) end
    end
    connections = {}
end

local function enableRagdoll()
    enableAntiKB()
    ENABLE_ANTI_RAGDOLL = true
end

local function disableRagdoll()
    disableAntiKB()
    ENABLE_ANTI_RAGDOLL = false
end

player.CharacterAdded:Connect(function(char)
    task.wait(0.4)
    initAntiRagdoll(char)
end)

if player.Character then
    initAntiRagdoll(player.Character)
end

--------------------------------------------------
-- INTERFACE DO HUB
--------------------------------------------------
local Gui = Instance.new("ScreenGui")
Gui.Name = "FluxoHubSmall"
Gui.ResetOnSpawn = false
Gui.Parent = PlayerGui

-- Hub Principal
local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.fromOffset(200, 350)  -- Aumentado para acomodar mais um botão
Main.Position = UDim2.fromScale(0.03, 0.3)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Main.Active = true
Main.Draggable = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0,14)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(160, 0, 255)

local WhiteStroke = Instance.new("UIStroke", Main)
WhiteStroke.Thickness = 1
WhiteStroke.Color = Color3.fromRGB(255,255,255)
WhiteStroke.Transparency = 0.4

-- Título
local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1,0,0,35)
Title.BackgroundTransparency = 1
Title.Text = "FLUXO HUB"
Title.Font = Enum.Font.GothamBlack
Title.TextScaled = true
Title.TextColor3 = Color3.fromRGB(200, 100, 255)

--------------------------------------------------
-- FUNÇÃO PARA CRIAR BOTÕES (SEM ANIMAÇÕES)
--------------------------------------------------
local function CreateButton(text, y, isToggle, inputValue)
    local btn = Instance.new("TextButton", Main)
    btn.Size = UDim2.new(0.85, 0, 0, 32)
    btn.Position = UDim2.new(0.075, 0, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.AutoButtonColor = false
    
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Text = text
    btn.BorderSizePixel = 0

    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,10)

    local valueBox
    if isToggle then
        valueBox = Instance.new("TextBox")
        valueBox.Parent = Main
        valueBox.Size = UDim2.new(0.3, 0, 0, 25)
        valueBox.Position = UDim2.new(0.65, 0, 0, y + 3)
        valueBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        valueBox.TextColor3 = Color3.fromRGB(255, 255, 255)
        valueBox.Font = Enum.Font.Gotham
        valueBox.TextSize = 12
        valueBox.Text = tostring(inputValue or 0)
        valueBox.ClearTextOnFocus = false
        valueBox.TextXAlignment = Enum.TextXAlignment.Center
        Instance.new("UICorner", valueBox).CornerRadius = UDim.new(0, 5)
        
        valueBox.FocusLost:Connect(function()
            local num = tonumber(valueBox.Text)
            if num then
                if text == "SPEED" then
                    SpeedValue = num
                elseif text == "SPEED" then  -- Ambos são "SPEED" agora
                    SpeedValue2 = num
                elseif text == "JUMP" then  -- Alterado de "INF JUMP" para "JUMP"
                    JumpValue = num
                end
            else
                if text == "SPEED" then
                    valueBox.Text = tostring(SpeedValue)
                elseif text == "SPEED" then  -- Ambos são "SPEED" agora
                    valueBox.Text = tostring(SpeedValue2)
                elseif text == "JUMP" then  -- Alterado de "INF JUMP" para "JUMP"
                    valueBox.Text = tostring(JumpValue)
                end
            end
        end)
    end

    local ligado = false
    
    btn.MouseButton1Click:Connect(function()
        ligado = not ligado
        
        if ligado then
            btn.BackgroundColor3 = Color3.fromRGB(0, 150, 50)
        else
            btn.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
        end
        
        -- Identificar qual botão SPEED foi clicado pela posição Y
        if text == "SPEED" then
            -- Primeiro botão SPEED (posição Y = 45)
            if y == 45 then
                if ligado then
                    SpeedEnabled = true
                    SpeedEnabled2 = false
                    -- Desativar visualmente o segundo botão SPEED
                    for _, btn2 in pairs(Main:GetChildren()) do
                        if btn2:IsA("TextButton") and btn2.Text == "SPEED" and btn2.Position.Y.Offset == 82 then
                            btn2.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
                        end
                    end
                else
                    SpeedEnabled = false
                end
            -- Segundo botão SPEED (posição Y = 82)
            elseif y == 82 then
                if ligado then
                    SpeedEnabled2 = true
                    SpeedEnabled = false
                    -- Desativar visualmente o primeiro botão SPEED
                    for _, btn2 in pairs(Main:GetChildren()) do
                        if btn2:IsA("TextButton") and btn2.Text == "SPEED" and btn2.Position.Y.Offset == 45 then
                            btn2.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
                        end
                    end
                else
                    SpeedEnabled2 = false
                end
            end
        elseif text == "JUMP" then  -- Alterado de "INF JUMP" para "JUMP"
            JumpEnabled = ligado
        elseif text == "ANTI LAG" then
            AntiLagEnabled = ligado
            if ligado then
                AtivarAntiLag()
            else
                DesativarAntiLag()
            end
        elseif text == "AUTO GRAB" then
            AutoGrabEnabled = ligado
            if ligado then
                StartAdvancedAutoGrab()
            else
                StopAdvancedAutoGrab()
            end
        elseif text == "X-RAY" then
            if ligado then
                AtivarXRay()
            else
                DesativarXRay()
            end
        elseif text == "AUTO BAT" then
            AutoBatEnabled = ligado
            if ligado then
                pcall(function()
                    StarterGui:SetCore("SendNotification", {
                        Title = "Fluxo Hub",
                        Text = "Auto Bat ATIVADO",
                        Duration = 3,
                        Icon = "rbxassetid://6723928013"
                    })
                end)
                print("⚾ Auto Bat ATIVADO")
            else
                pcall(function()
                    StarterGui:SetCore("SendNotification", {
                        Title = "Fluxo Hub",
                        Text = "Auto Bat DESATIVADO",
                        Duration = 3,
                        Icon = "rbxassetid://6723928013"
                    })
                end)
                print("⚾ Auto Bat DESATIVADO")
            end
        elseif text == "ANTI RAGDOLL" then
            AntiRagdollEnabled = ligado
            if ligado then
                enableRagdoll()
                pcall(function()
                    StarterGui:SetCore("SendNotification", {
                        Title = "Fluxo Hub",
                        Text = "Anti Ragdoll ATIVADO",
                        Duration = 3,
                        Icon = "rbxassetid://6723928013"
                    })
                end)
                print("🛡️ Anti Ragdoll ATIVADO")
            else
                disableRagdoll()
                pcall(function()
                    StarterGui:SetCore("SendNotification", {
                        Title = "Fluxo Hub",
                        Text = "Anti Ragdoll DESATIVADO",
                        Duration = 3,
                        Icon = "rbxassetid://6723928013"
                    })
                end)
                print("🛡️ Anti Ragdoll DESATIVADO")
            end
        end
    end)

    return btn, valueBox
end

--------------------------------------------------
-- CRIAR BOTÕES COM POSIÇÕES AJUSTADAS
--------------------------------------------------
local speedBtn, speedBox = CreateButton("SPEED", 45, true, 30)
local speedBtn2, speedBox2 = CreateButton("SPEED", 82, true, 60)  -- AGORA AMBOS SÃO "SPEED"
local jumpBtn, jumpBox = CreateButton("JUMP", 119, true, 50)  -- Alterado de "INF JUMP" para "JUMP"
CreateButton("AUTO GRAB", 156, false, nil)  -- Posição ajustada
CreateButton("ANTI LAG", 193, false, nil)  -- Posição ajustada
CreateButton("X-RAY", 230, false, nil)  -- Posição ajustada
CreateButton("AUTO BAT", 267, false, nil)  -- Posição ajustada
CreateButton("ANTI RAGDOLL", 304, false, nil)  -- NOVO BOTÃO

-- Adicionar padding à esquerda para todos os botões
for _, btn in pairs(Main:GetChildren()) do
    if btn:IsA("TextButton") and btn.Name == "" then
        local padding = Instance.new("UIPadding")
        padding.Parent = btn
        padding.PaddingLeft = UDim.new(0, 12)
    end
end

if speedBox then speedBox.Text = "30" end
if speedBox2 then speedBox2.Text = "60" end  -- AGORA É O SEGUNDO SPEED
if jumpBox then jumpBox.Text = "50" end

--------------------------------------------------
-- LOOPS DE FUNCIONALIDADES
--------------------------------------------------
RunService.Heartbeat:Connect(function()
    -- Aplicar SPEED se qualquer um dos dois estiver ativado
    local currentSpeed = 0
    if SpeedEnabled then currentSpeed = currentSpeed + SpeedValue end
    if SpeedEnabled2 then currentSpeed = currentSpeed + SpeedValue2 end
    
    if currentSpeed > 0 and HRP and Humanoid then
        local d = Humanoid.MoveDirection
        HRP.AssemblyLinearVelocity = Vector3.new(d.X * currentSpeed, HRP.AssemblyLinearVelocity.Y, d.Z * currentSpeed)
    end
    
    if not player.Character or not HRP then
        SpeedEnabled = false
        SpeedEnabled2 = false
        JumpEnabled = false
    end
end)

UserInputService.JumpRequest:Connect(function()
    if JumpEnabled and HRP and player.Character then
        HRP.AssemblyLinearVelocity = Vector3.new(HRP.AssemblyLinearVelocity.X, JumpValue, HRP.AssemblyLinearVelocity.Z)
    end
end)

--------------------------------------------------
-- AUTO BAT - EXATAMENTE COMO VOCÊ QUER
--------------------------------------------------
task.spawn(function()
    while true do
        if AutoBatEnabled then
            local player = game.Players.LocalPlayer
            local char = player.Character
            if char then
                local tool = char:FindFirstChild("Bat") or char:FindFirstChildWhichIsA("Tool")
                if not tool then
                    local bpItem = player.Backpack:FindFirstChild("Bat") or player.Backpack:FindFirstChildWhichIsA("Tool")
                    if bpItem then 
                        bpItem.Parent = char 
                        tool = bpItem 
                    end
                end
                if tool then 
                    tool:Activate() 
                end
            end
        end
        task.wait(0.1)
    end
end)

--------------------------------------------------
-- LIMPEZA
--------------------------------------------------
Gui.Destroying:Connect(function()
    if AutoGrabScript then
        StopAdvancedAutoGrab()
    end
    if XRayEnabled then
        DesativarXRay()
    end
    if AntiRagdollEnabled then
        disableRagdoll()
    end
    SpeedEnabled = false
    SpeedEnabled2 = false
    JumpEnabled = false
    AntiLagEnabled = false
    AutoBatEnabled = false
end)

--------------------------------------------------
-- NOTIFICAÇÃO INICIAL
--------------------------------------------------
task.wait(1)
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "Fluxo Hub",
        Text = "Carregado com sucesso!",
        Duration = 5,
        Icon = "rbxassetid://132564087673178"
    })
end)

print("✅ FLUXO HUB - CARREGADO COM SUCESSO")
print("📊 CONFIGURAÇÕES: Speed=30, Speed=60, Jump=50")
print("🎮 FUNCIONALIDADES DISPONÍVEIS:")
print("   • SPEED (30 padrão) - Texto alinhado à esquerda")
print("   • SPEED (60 padrão) - Texto alinhado à esquerda")
print("   • JUMP (50 padrão) - Texto alinhado à esquerda")  -- Alterado de "INF JUMP" para "JUMP"
print("   • AUTO GRAB AVANÇADO")
print("   • ANTI LAG (Otimização gráfica)")
print("   • X-RAY (Bases Transparentes - Transparência: 0.7)")
print("   • AUTO BAT (Ativa ferramentas automaticamente a cada 0.1s)")
print("   • ANTI RAGDOLL (Previne queda e ragdoll)")
print("   • Botões SEM ANIMAÇÕES - apenas mudam de cor instantaneamente")
--------------------------------------------------
-- TOGGLE SPEED PELO TECLADO (TECLA F)
--------------------------------------------------
local SpeedToggleIndex = 1 -- 1 = SPEED 1 | 2 = SPEED 2

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.F then
        
        -- Alternar índice
        SpeedToggleIndex = SpeedToggleIndex == 1 and 2 or 1

        if SpeedToggleIndex == 1 then
            SpeedEnabled = true
            SpeedEnabled2 = false

            -- Atualizar visual dos botões
            if speedBtn then
                speedBtn.BackgroundColor3 = Color3.fromRGB(0,150,50)
            end
            if speedBtn2 then
                speedBtn2.BackgroundColor3 = Color3.fromRGB(40,0,70)
            end

            print("⚡ SPEED 1 ATIVADO (", SpeedValue, ")")

        else
            SpeedEnabled = false
            SpeedEnabled2 = true

            -- Atualizar visual dos botões
            if speedBtn then
                speedBtn.BackgroundColor3 = Color3.fromRGB(40,0,70)
            end
            if speedBtn2 then
                speedBtn2.BackgroundColor3 = Color3.fromRGB(0,150,50)
            end

            print("⚡ SPEED 2 ATIVADO (", SpeedValue2, ")")
        end
    end
end)
