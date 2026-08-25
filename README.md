-- ============================================================
-- RED ONYX HUB LITE v2 - 100% STANDALONE
-- NADA é baixado | Tudo escrito na mão | Super leve
-- ============================================================
print("[Red Onyx Lite] Iniciando...")

-- ===== COMPATIBILIDADE =====
local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== CORES FIXAS (sem cálculo) =====
local VERM = Color3.fromRGB(200, 30, 45)
local VERM_C = Color3.fromRGB(240, 80, 90)
local ESC = Color3.fromRGB(15, 12, 14)
local ESC_M = Color3.fromRGB(22, 16, 18)
local BRA = Color3.fromRGB(240, 235, 237)
local CIN = Color3.fromRGB(150, 140, 142)

-- ===== VARIÁVEIS GLOBAIS =====
local Jogadores = game:GetService("Players")
local RunService = game:GetService("RunService")
local Jogador = Jogadores.LocalPlayer
local Core = game:GetService("CoreGui")

-- Flags de controle (desliga as funções)
local Ativo = {
    Farm = false,
    Bau = false,
    Coleta = false,
    Quest = false,
    Codigos = false,
}

-- ============================================================
-- PATCHER MÍNIMO (só troca texto, SEM conexões de evento)
-- ============================================================
local function patcher_minimo()
    spawn(function()
        aguardar(2)
        while true do
            pcall(function()
                for _, gui in ipairs(Core:GetChildren()) do
                    if gui:IsA("ScreenGui") then
                        for _, obj in ipairs(gui:GetDescendants()) do
                            if (obj:IsA("TextLabel") or obj:IsA("TextButton")) and not obj:IsA("TextBox") then
                                local txt = obj.Text
                                if type(txt) == "string" and txt ~= "" then
                                    local tl = txt:lower()
                                    if tl:find("quantum") or tl:find("onyx") or tl:find("kaitun") then
                                        obj.Text = "Red Onyx Hub"
                                    end
                                end
                                -- Aplica cor vermelha em textos
                                obj.TextColor3 = VERM_C
                            end
                            if obj:IsA("GuiObject") then
                                obj.BackgroundColor3 = ESC_M
                                obj.BorderColor3 = VERM
                            end
                            if obj:IsA("UIStroke") then
                                obj.Color = VERM
                            end
                        end
                    end
                end
            end)
            aguardar(5) -- Só verifica a cada 5 segundos
        end
    end)
end

-- ============================================================
-- FUNÇÕES DE AUTO FARM (leves, sem dependência externa)
-- ============================================================

-- Auto Farm Nível: anda até mobs e ataca
local function iniciar_farm()
    Ativo.Farm = true
    spawn(function()
        while Ativo.Farm do
            aguardar(0.3)
            pcall(function()
                if not Jogador or not Jogador.Character then return end
                local char = Jogador.Character
                local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
                if not hrp then return end
                local hum = char:FindFirstChildOfClass("Humanoid")
                if not hum or hum.Health <= 0 then return end

                -- Procura mob mais próximo
                local alvo = nil
                local distMin = math.huge
                for _, obj in ipairs(workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj ~= char then
                        local mobHum = obj:FindFirstChildOfClass("Humanoid")
                        local mobHrp = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
                        if mobHum and mobHrp and mobHum.Health > 0 then
                            local dist = (hrp.Position - mobHrp.Position).Magnitude
                            if dist < distMin and dist < 150 then -- Só considera até 150 studs
                                distMin = dist
                                alvo = mobHrp
                            end
                        end
                    end
                end

                if alvo and distMin < 150 then
                    -- Teleporta pra perto do mob
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 0, 4 + math.random() * 2)

                    -- Usa ferramenta se tiver
                    local tool = char:FindFirstChildOfClass("Tool") or char:FindFirstChildOfClass("HopperBin")
                    if tool then
                        pcall(function() tool:Activate() end)
                    end

                    -- Pula pra crítico
                    if hum:GetState() ~= Enum.HumanoidStateType.Jumping then
                        pcall(function() hum.Jump = true end)
                    end
                end
            end)
        end
    end)
end

-- Auto Coleta: pega fragmentos, ectoplasma, doces, ossos
local function iniciar_coleta()
    Ativo.Coleta = true
    spawn(function()
        while Ativo.Coleta do
            aguardar(0.5)
            pcall(function()
                if not Jogador or not Jogador.Character then return end
                local hrp = Jogador.Character:FindFirstChild("HumanoidRootPart") or Jogador.Character:FindFirstChild("Torso")
                if not hrp then return end

                for _, obj in ipairs(workspace:GetDescendants()) do
                    if obj:IsA("BasePart") and obj.Transparency < 1 and obj.CanCollide == false then
                        local nome = obj.Name:lower()
                        local pegar = false

                        if nome:find("fragment") or nome:find("fragmento") then pegar = true end
                        if nome:find("ectoplasm") or nome:find("ectoplasma") then pegar = true end
                        if nome:find("candy") or nome:find("doce") or nome:find("bone") or nome:find("osso") then pegar = true end
                        if nome:find("chest") or nome:find("bau") or nome:find("baú") then pegar = true end
                        -- Pega qualquer item coletável (frutas, etc)
                        if obj:FindFirstChild("TouchInterest") or obj:FindFirstChild("ClickDetector") then pegar = true end

                        if pegar then
                            local dist = (hrp.Position - obj.Position).Magnitude
                            if dist < 30 then
                                hrp.CFrame = CFrame.new(obj.Position + Vector3.new(0, 2, 0))
                                aguardar(0.05)
                                firetouchinterest(hrp, obj, 0)
                                aguardar(0.05)
                                firetouchinterest(hrp, obj, 1)
                            end
                        end
                    end
                end
            end)
        end
    end)
end

-- Auto Quest: procura NPCs com quest
local function iniciar_quest()
    Ativo.Quest = true
    spawn(function()
        while Ativo.Quest do
            aguardar(2)
            pcall(function()
                if not Jogador or not Jogador.Character then return end
                local hrp = Jogador.Character:FindFirstChild("HumanoidRootPart")
                if not hrp then return end

                for _, obj in ipairs(workspace:GetDescendants()) do
                    local nome = obj.Name:lower()
                    if obj:IsA("Model") and (nome:find("npc") or nome:find("quest") or nome:find("merchant") or nome:find("vendedor")) then
                        local head = obj:FindFirstChild("Head") or obj:FindFirstChild("HumanoidRootPart")
                        if head then
                            local dist = (hrp.Position - head.Position).Magnitude
                            if dist < 15 then
                                -- Interage com o NPC
                                local pp = head:FindFirstChildWhichIsA("ProximityPrompt")
                                if pp then
                                    fireproximityprompt(pp)
                                else
                                    -- Tenta clicar
                                    local cd = head:FindFirstChildWhichIsA("ClickDetector")
                                    if cd then
                                        fireclickdetector(cd)
                                    end
                                end
                            end
                        end
                    end
                end
            end)
        end
    end)
end

-- Auto Códigos: resgata códigos
local function iniciar_codigos()
    Ativo.Codigos = true
    spawn(function()
        aguardar(3) -- Aguarda o jogo carregar
        pcall(function()
            if not Jogador then return end

            local Storage = game:GetService("ReplicatedStorage")
            local remote = Storage:FindFirstChild("Redeem") or Storage:FindFirstChild("RedeemCode")

            if not remote then
                -- Procura em outros lugares
                remote = Jogador:FindFirstChild("Redeem") or Jogador:FindFirstChild("RedeemCode")
            end

            if not remote then
                print("[Red Onyx Lite] Remote de codigos não encontrado")
                Ativo.Codigos = false
                return
            end

            local codigos = {
                "KITT_RESET", "SUB2GAMERROBOT_RESET1", "Sub2UncleKizaru",
                "SUB2GAMERROBOT_EXP1", "15B_BESTBROTHERS",
                "EASTEREXP", "LIGHTNINGABUSE", "Axiore", "Bluxxy",
                "Enyu_is_Pro", "JCWK", "Kittgaming", "Magicbus",
                "Starcodeheo", "StrawHatMaine", "Sub2CaptainMaui",
                "Sub2Fer999", "Sub2OfficialNoobie", "1lostadmin ",
                "Sub2Daigrock", "Sub2NoobMaster123", "TantaiGaming",
                "Fudd10", "Fudd10_v2", "Bignews", "Chandler",
                "TY_FOR_WATCHING", "GAMER_ROBOT_1M",
            }

            local resgatados = 0
            for i, cod in ipairs(codigos) do
                local ok = pcall(function()
                    if remote:IsA("RemoteFunction") then
                        remote:InvokeServer(cod)
                    elseif remote:IsA("RemoteEvent") then
                        remote:FireServer(cod)
                    end
                end)
                if ok then resgatados = resgatados + 1 end
                if i % 5 == 0 then aguardar(0.3) end
                aguardar(0.05)
            end
            print("[Red Onyx Lite] Codigos resgatados: " .. resgatados .. "/" .. #codigos)
            Ativo.Codigos = false
        end)
    end)
end

-- Parar tudo
local function parar_tudo()
    Ativo.Farm = false
    Ativo.Bau = false
    Ativo.Coleta = false
    Ativo.Quest = false
    Ativo.Codigos = false
    print("[Red Onyx Lite] Todas as funcoes paradas")
end

-- ============================================================
-- UI SUPER LEVE
-- ============================================================
local function criar_ui()
    local sg = Instance.new("ScreenGui")
    sg.Name = "RedOnyxLite"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false

    local pai_ok = pcall(function() sg.Parent = Core end)
    if not pai_ok then
        pcall(function()
            local hui = gethui()
            if typeof(hui) == "Instance" then sg.Parent = hui end
        end)
    end

    -- Card
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 280, 0, 260)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = ESC_M
    card.BorderSizePixel = 0
    card.Parent = sg

    local canto = Instance.new("UICorner")
    canto.CornerRadius = UDim.new(0, 8)
    canto.Parent = card

    local borda = Instance.new("UIStroke")
    borda.Color = VERM
    borda.Thickness = 2
    borda.Parent = card

    -- Título
    local tit = Instance.new("TextLabel")
    tit.Size = UDim2.new(1, 0, 0, 30)
    tit.Position = UDim2.new(0, 0, 0, 6)
    tit.BackgroundTransparency = 1
    tit.Font = Enum.Font.GothamBlack
    tit.Text = "RED ONYX LITE"
    tit.TextColor3 = VERM_C
    tit.TextSize = 18
    tit.TextXAlignment = Enum.TextXAlignment.Center
    tit.Parent = card

    -- Status geral
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(1, -10, 0, 14)
    status.Position = UDim2.new(0.5, 0, 0, 36)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = "Clique para iniciar"
    status.TextColor3 = CIN
    status.TextSize = 10
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.Parent = card

    local function set_status(txt)
        status.Text = txt
    end

    -- Botões
    local function criar_btn(posY, cor, txt, cb)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 240, 0, 28)
        btn.Position = UDim2.new(0.5, 0, 0, posY)
        btn.AnchorPoint = Vector2.new(0.5, 0)
        btn.BackgroundColor3 = cor
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = txt
        btn.TextColor3 = BRA
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.Parent = card

        local cb2 = Instance.new("UICorner")
        cb2.CornerRadius = UDim.new(0, 5)
        cb2.Parent = btn

        btn.MouseButton1Click:Connect(function()
            pcall(cb)
        end)
        return btn
    end

    local rodando = false

    criar_btn(60, VERM, "INICIAR TUDO", function()
        if rodando then
            set_status("Já está rodando!")
            return
        end
        rodando = true
        set_status("Iniciando todas as funções...")

        iniciar_farm()
        aguardar(0.1)
        iniciar_coleta()
        aguardar(0.1)
        iniciar_quest()
        aguardar(0.1)
        iniciar_codigos()

        set_status("Tudo ativo! Farm + Coleta + Quest + Códigos")
        print("[Red Onyx Lite] Todas as funções iniciadas!")
    end)

    criar_btn(95, Color3.fromRGB(80, 20, 28), "PARAR TUDO", function()
        parar_tudo()
        rodando = false
        set_status("Parado. Clique em Iniciar para voltar")
    end)

    criar_btn(145, VERM, "RESGATAR CÓDIGOS", function()
        set_status("Resgatando códigos...")
        spawn(function()
            iniciar_codigos()
            aguardar(10)
            set_status("Códigos processados!")
        end)
    end)

    criar_btn(180, Color3.fromRGB(60, 20, 28), "FECHAR UI", function()
        parar_tudo()
        sg:Destroy()
    end)

    -- Info
    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(1, -10, 0, 14)
    info.Position = UDim2.new(0.5, 0, 1, -18)
    info.AnchorPoint = Vector2.new(0.5, 0)
    info.BackgroundTransparency = 1
    info.Font = Enum.Font.Gotham
    info.Text = "Zero download | Ultra leve | ESC para fechar"
    info.TextColor3 = Color3.fromRGB(90, 85, 87)
    info.TextSize = 9
    info.TextXAlignment = Enum.TextXAlignment.Center
    info.Parent = card

    -- Fechar com ESC
    spawn(function()
        aguardar(0.5)
        local cInp = game:GetService("ContextActionService")
        cInp:BindActionAt("FecharRedOnyxLite", function()
            parar_tudo()
            sg:Destroy()
        end, false, Enum.KeyCode.Escape)
    end)
end

-- ============================================================
-- INICIAR
-- ============================================================

-- Patcher mínimo (só texto, a cada 5s)
patcher_minimo()

-- UI
local ok, err = pcall(criar_ui)
if not ok then
    warn("[Red Onyx Lite] ERRO NA UI: " .. tostring(err))

    -- Fallback: inicia direto sem UI
    spawn(function()
        aguardar(3)
        iniciar_farm()
        iniciar_coleta()
        iniciar_quest()
        aguardar(5)
        iniciar_codigos()
    end)
end

print("[Red Onyx Lite] Versao LITE - sem travamentos!")
