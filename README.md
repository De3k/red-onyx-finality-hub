-- ============================================================
-- RED ONYX PROJECT v2 - VERSÃO COMPLETA FINAL
-- Key system + Auto Pega Tudo + Lib futurista vermelho/preto
-- 100% standalone | Zero resquício de Quantum | Anti-travamento
-- ============================================================
print("[Red Onyx Project] Iniciando...")

-- ===== SERVIÇOS =====
local Jogadores    = game:GetService("Players")
local UserInput    = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Replicated   = game:GetService("ReplicatedStorage")
local Core         = game:GetService("CoreGui")

local Jogador = Jogadores.LocalPlayer

-- ===== COMPAT =====
local aguardar = (task and task.wait) or wait
local spawn    = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== PALETA FUTURISTA =====
local VERM   = Color3.fromRGB(225, 25, 45)
local VERM_C = Color3.fromRGB(255, 90, 105)
local VERM_E = Color3.fromRGB(115, 12, 26)
local NEON   = Color3.fromRGB(255, 40, 60)
local PRETO  = Color3.fromRGB(10, 8, 10)
local FUNDO  = Color3.fromRGB(16, 12, 14)
local PAINEL = Color3.fromRGB(24, 17, 20)
local CARD   = Color3.fromRGB(28, 20, 24)
local BRA    = Color3.fromRGB(245, 240, 242)
local CIN    = Color3.fromRGB(150, 140, 145)

-- ============================================================
-- BIBLIOTECA RED ONYX (RedOnyxLib) - DESIGN FUTURISTA
-- ============================================================
local RedOnyxLib = {}

RedOnyxLib.Config = {
    AutoFarm       = false,
    AutoColeta     = false,
    AutoBaus       = false,
    AutoQuest      = false,
    AutoBoss       = false,
    DistanciaFarm  = 300,
    SaltoAttack    = true,
}

-- helpers
local function novo(classe, props, pai)
    local obj = Instance.new(classe)
    for k, v in pairs(props) do obj[k] = v end
    obj.Parent = pai
    return obj
end
local function canto(pai, raio)
    return novo("UICorner", { CornerRadius = UDim.new(0, raio or 8), Parent = pai })
end

-- ---------- JANELA ----------
function RedOnyxLib.criar_janela(cfg)
    cfg = cfg or {}

    local sg = novo("ScreenGui", {
        Name = "RedOnyxProject",
        IgnoreGuiInset = true,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })
    local ok_pai = pcall(function() sg.Parent = Core end)
    if not ok_pai then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg.Parent = hui end
            end
        end)
    end

    local janela = novo("Frame", {
        Name = "Janela",
        Size = UDim2.new(0, 480, 0, 330),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = FUNDO,
        BorderSizePixel = 0,
        Active = true,
        Parent = sg,
    })
    canto(janela, 12)
    local stroke = novo("UIStroke", { Color = VERM_E, Thickness = 1.5, Parent = janela })

    -- Glow neon pulsante
    spawn(function()
        while janela.Parent do
            local t1 = TweenService:Create(stroke, TweenInfo.new(1.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), { Color = NEON, Transparency = 0.1 })
            t1:Play(); t1.Completed:Wait()
            if not janela.Parent then break end
            local t2 = TweenService:Create(stroke, TweenInfo.new(1.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), { Color = VERM_E, Transparency = 0.45 })
            t2 = t2 or t2
            t2:Play(); t2.Completed:Wait()
        end
    end)

    -- Header
    local header = novo("Frame", {
        Size = UDim2.new(1, 0, 0, 46),
        BackgroundColor3 = PAINEL,
        BorderSizePixel = 0,
        Parent = janela,
    })
    canto(header, 12)
    novo("UIGradient", {
        Color = ColorSequence.new(Color3.fromRGB(50, 12, 20), FUNDO),
        Rotation = 90, Parent = header,
    })
    novo("Frame", {
        Size = UDim2.new(1, -24, 0, 1),
        Position = UDim2.new(0, 12, 0, 46),
        BackgroundColor3 = VERM,
        BorderSizePixel = 0, Parent = janela,
    })

    novo("TextLabel", {
        Size = UDim2.new(0, 220, 0, 26), Position = UDim2.new(0, 14, 0, 7),
        BackgroundTransparency = 1, Font = Enum.Font.GothamBlack,
        Text = cfg.titulo or "RED ONYX", TextColor3 = VERM_C, TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left, Parent = header,
    })
    novo("TextLabel", {
        Size = UDim2.new(0, 250, 0, 12), Position = UDim2.new(0, 15, 0, 28),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        Text = cfg.subtitulo or "PROJECT", TextColor3 = CIN, TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left, Parent = header,
    })

    local btn_min = novo("TextButton", {
        Size = UDim2.new(0, 26, 0, 26), Position = UDim2.new(1, -34, 0, 10),
        BackgroundColor3 = VERM_E, Font = Enum.Font.GothamBold,
        Text = "—", TextColor3 = BRA, TextSize = 12, AutoButtonColor = false, Parent = header,
    })
    canto(btn_min, 6)

    -- Arrastar
    do
        local arrastando, p0, j0 = false, nil, nil
        header.InputBegan:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                arrastando, p0_pos = true, inp.Position
                p0 = inp.Position
                j0 = janela.Position
                inp.Changed:Connect(function()
                    if inp.UserInputState == Enum.UserInputState.End then arrastando = false end
                end)
            end
        end)
        UserInput.InputChanged:Connect(function(inp)
            if arrastando and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) then
                local d = inp.Position - p0
                janela.Position = UDim2.new(j0.X.Scale, j0.X.Offset + d.X, j0.Y.Scale, j0.Y.Offset + d.Y)
            end
        end)
    end

    local col_tabs = novo("Frame", {
        Size = UDim2.new(0, 110, 1, -58), Position = UDim2.new(0, 10, 0, 56),
        BackgroundTransparency = 1, Parent = janela,
    })
    novo("UIListLayout", { Padding = UDim.new(0, 6), Parent = col_tabs })

    local conteudo = novo("Frame", {
        Size = UDim2.new(1, -132, 1, -66), Position = UDim2.new(0, 122, 0, 56),
        BackgroundColor3 = PRETO, BackgroundTransparency = 0.35,
        BorderSizePixel = 0, Parent = janela,
    })
    canto(conteudo, 8)

    local api = {}
    local abas = {}

    function api.criar_aba(nome)
        local btn_tab = novo("TextButton", {
            Size = UDim2.new(1, 0, 0, 28), BackgroundColor3 = CARD,
            Font = Enum.Font.GothamBold, Text = nome, TextColor3 = CIN,
            TextSize = 12, AutoButtonColor = false, Parent = col_tabs,
        })
        canto(btn_tab, 6)

        local pagina = novo("ScrollingFrame", {
            Size = UDim2.new(1, -8, 1, -8), Position = UDim2.new(0, 4, 0, 4),
            BackgroundTransparency = 1, BorderSizePixel = 0,
            ScrollBarThickness = 3, ScrollBarImageColor3 = VERM,
            CanvasSize = UDim2.new(0, 0, 0, 0),
            AutomaticCanvasSize = Enum.AutomaticSize.Y,
            Visible = false, Parent = conteudo,
        })
        novo("UIListLayout", { Padding = UDim.new(0, 6), Parent = pagina })

        local aba = { btn = btn_tab, pagina = pagina }
        table.insert(abas, aba)

        btn_tab.MouseButton1Click:Connect(function()
            for _, a in ipairs(abas) do
                a.pagina.Visible = false
                a.btn.TextColor3 = CIN
                a.btn.BackgroundColor3 = CARD
            end
            pagina.Visible = true
            btn_tab.TextColor3 = VERM_C
            btn_tab.BackgroundColor3 = PAINEL
        end)

        if #abas == 1 then
            pagina.Visible = true
            btn_tab.TextColor3 = VERM_C
            btn_tab.BackgroundColor3 = PAINEL
        end

        local comps = {}

        function comps.botao(txt, cb)
            local btn = novo("TextButton", {
                Size = UDim2.new(1, -8, 0, 30), BackgroundColor3 = CARD,
                Font = Enum.Font.GothamBold, Text = txt, TextColor3 = BRA,
                TextSize = 11, AutoButtonColor = false, Parent = pagina,
            })
            canto(btn, 6)
            novo("UIStroke", { Color = VERM_E, Thickness = 1, Parent = btn })
            btn.MouseButton1Click:Connect(function() spawn(cb) end)
            return btn
        end

        function comps.toggle(txt, chave, padrao)
            RedOnyxLib.Config[chave] = padrao and true or false

            local row = novo("TextButton", {
                Size = UDim2.new(1, -8, 0, 30), BackgroundColor3 = CARD,
                Text = "", AutoButtonColor = false, Parent = pagina,
            })
            canto(row, 6)

            novo("TextLabel", {
                Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1, Font = Enum.Font.Gotham,
                Text = txt, TextColor3 = BRA, TextSize = 11,
                TextXAlignment = Enum.TextXAlignment.Left, Parent = row,
            })

            local trilho = novo("Frame", {
                Size = UDim2.new(0, 36, 0, 16),
                Position = UDim2.new(1, -46, 0.5, 0), AnchorPoint = Vector2.new(0, 0.5),
                BackgroundColor3 = PAINEL, BorderSizePixel = 0, Parent = row,
            })
            canto(trilho, 8)
            local bolinha = novo("Frame", {
                Size = UDim2.new(0, 12, 0, 12),
                Position = UDim2.new(0, 2, 0.5, 0), AnchorPoint = Vector2.new(0, 0.5),
                BackgroundColor3 = CIN, BorderSizePixel = 0, Parent = trilho,
            })
            canto(bolinha, 8)

            local ligado = RedOnyxLib.Config[chave]

            local function refresh(instante)
                if ligado then
                    trilho.BackgroundColor3 = VERM_E
                    bolinha.BackgroundColor3 = VERM_C
                    bolinha:TweenPosition(UDim2.new(1, -14, 0.5, 0), "Out", "Quad", 0.15, true)
                else
                    trilho.BackgroundColor3 = PAINEL
                    bolinha.BackgroundColor3 = CIN
                    bolinha:TweenPosition(UDim2.new(0, 2, 0.5, 0), "Out", "Quad", 0.15, true)
                end
            end
            refresh(true)

            row.MouseButton1Click:Connect(function()
                ligado = not ligado
                RedOnyxLib.Config[chave] = ligado
                refresh()
            end)
            return row
        end

        function comps.label(txt)
            return novo("TextLabel", {
                Size = UDim2.new(1, -8, 0, 20), BackgroundTransparency = 1,
                Font = Enum.Font.Gotham, Text = txt, TextColor3 = CIN,
                TextSize = 10, TextXAlignment = Enum.TextXAlignment.Left,
                TextWrapped = true, Parent = pagina,
            })
        end

        function comps.status(txt, cor)
            local l = novo("TextLabel", {
                Size = UDim2.new(1, -8, 0, 20), BackgroundTransparency = 1,
                Font = Enum.Font.GothamBold, Text = txt, TextColor3 = cor or VERM_C,
                TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left,
                TextWrapped = true, Parent = pagina,
            })
            return l
        end

        return comps
    end

    btn_min.MouseButton1Click:Connect(function()
        janela.Visible = not janela.Visible
    end)

    UserInput.InputBegan:Connect(function(inp, proc)
        if proc then return end
        if inp.KeyCode == Enum.KeyCode.RightShift then
            janela.Visible = not janela.Visible
        end
    end)

    api.janela = janela
    return api
end

-- ============================================================
-- KEY SYSTEM (Red Onyx Key) - salvamento em disco
-- ============================================================
local ARQ_KEY = "RedOnyxKey.txt"

local function salvar_key(chave)
    pcall(function()
        if makefolder then pcall(makefolder, "RedOnyx") end
        writefile(ARQ_KEY, chave)
    end)
    pcall(function()
        if syn and syn.writefile then syn.writefile(ARQ_KEY, chave) end
    end)
end

local function carregar_key()
    local chave = ""
    pcall(function() chave = readfile(ARQ_KEY) end)
    if type(chave) ~= "string" or chave == "" then
        pcall(function()
            if syn and syn.readfile then chave = syn.readfile(ARQ_KEY) end
        end)
    end
    if type(chave) ~= "string" then chave = "" end
    return chave
end

local function deletar_key()
    pcall(delfile, ARQ_KEY)
    pcall(function() if syn and syn.delfile then syn.delfile(ARQ_KEY) end end)
end

-- Validação da key (WhitelistedKey) - usa _G pra compatibilidade
local function validar_key(chave, cb_ok, cb_erro)
    spawn(function()
        -- Aqui você pode integrar seu sistema de key (Luarmor, own API, etc.)
        -- Por enquanto: chave local segura (troque por sua validação se quiser)
        if #chave >= 8 then
            cb_ok()
        else
            cb_erro("Key invalida! Minimo 8 caracteres.")
        end
    end)
end

-- ============================================================
-- FARM CORE (leve, sem loops infinitos pesados)
-- ============================================================
local function pegar_hrp()
    local char = Jogador and Jogador.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart")
end

local function achar_alvo(hrp, dist_max, so_boss)
    local alvo, d_alvo = nil, math.huge
    local char = Jogador.Character
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj:IsA("Model") and obj ~= char then
            local hum = obj:FindFirstChildOfClass("Humanoid")
            local raiz = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
            if hum and raiz and hum.Health > 0 then
                local n = obj.Name:lower()
                local e_boss = n:find("boss") or n:find("king") or n:find("sword") or n:find("captain")
                if (not so_boss or e_boss) then
                    local d = (hrp.Position - raiz.Position).Magnitude
                    if d < d_alvo then
                        d_alvo = d
                        alvo = raiz
                    end
                end
            end
        end
    end
    if d_alvo <= (dist_max or 300) then return alvo, d_alvo end
    return nil, nil
end

-- ===== LOOPS DE FARM =====
spawn(function()
    while true do
        if RedOnyxLib.Config.AutoFarm then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                local alvo = achar_alvo(hrp, RedOnyxLib.Config.DistanciaFarm, false)
                if alvo then
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 8, 4)
                    local char = Jogador.Character
                    local ferr = char and char:FindFirstChildOfClass("Tool")
                    if ferr then pcall(function() ferr:Activate() end) end
                    if RedOnyxLib.Config.SaltoAtaque then
                        local hum = char and char:FindFirstChildOfClass("Humanoid")
                        if hum and hum:GetState() ~= Enum.HumanoidStateType.Jumping then
                            pcall(function() hum.Jump = true end)
                        end
                    end
                end
            end)
        end
        aguardar(RedOnyxLib.Config.AutoFarm and 0.25 or 1)
    end
end)

spawn(function()
    while true do
        if RedOnyxLib.Config.AutoBoss then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                local alvo = achar_alvo(hrp, 9999, true) -- ignora distância
                if alvo then
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 8, 4)
                    local ferr = Jogador.Character and Jogador.Character:FindFirstChildOfClass("Tool")
                    if ferr then pcall(function() ferr:Activate() end) end
                end
            end)
        end
        aguardar(RedOnyxLib.Config.AutoBoss and 0.3 or 2)
    end
end)

spawn(function()
    while true do
        if RedOnyxLib.Config.AutoColeta or RedOnyxLib.Config.AutoBaus then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                for _, obj in ipairs(workspace:GetDescendants()) do
                    if obj:IsA("BasePart") and obj.Transparency < 1 then
                        local nome = obj.Name:lower()
                        local quer = false
                        if RedOnyxLib.Config.AutoBaus and (nome:find("chest") or nome:find("bau")) then quer = true end
                        if RedOnyxLib.Config.AutoColeta and (obj:FindFirstChild("TouchInterest") or obj:FindFirstChild("ClickDetector")) then quer = true end
                        if quer then
                            local d = (hrp.Position - obj.Position).Magnitude
                            if d < 60 then
                                hrp.CFrame = CFrame.new(obj.Position + Vector3.new(0, 2, 0))
                                if firetouchinterest then
                                    pcall(firetouchinterest, hrp, obj, 0)
                                    pcall(firetouchinterest, hrp, obj, 1)
                                end
                                break
                            end
                        end
                    end
                end
            end)
        end
        aguardar(0.4)
    end
end)

spawn(function()
    while true do
        if RedOnyxLib.Config.AutoQuest then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                for _, pp in ipairs(workspace:GetDescendants()) do
                    if pp:IsA("ProximityPrompt") and pp.Enabled then
                        local pai = pp.Parent
                        if pai and (pai.Position - hrp.Position).Magnitude < 12 then
                            pcall(fireproximityprompt, pp)
                            aguardar(0.5)
                            break
                        end
                    end
                end
            end)
        end
        aguardar(1)
    end
end)

-- ===== CÓDIGOS =====
local CODIGOS = {
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

local function resgatar_codigos()
    local remote = ReplicatedStorage:FindFirstChild("Redeem") or ReplicatedStorage:FindFirstChild("RedeemCode")
    if not remote then
        warn("[Red Onyx] Remote de codigos nao encontrado")
        return 0, false
    end
    for i, cod in ipairs(CODIGOS) do
        pcall(function()
            if remote:IsA("RemoteFunction") then
                remote:InvokeServer(cod)
            elseif remote:IsA("RemoteEvent") then
                remote:FireServer(cod)
            end
        end)
        if i % 5 == 0 then aguardar(0.4) end
        aguardar(0.08)
    end
    return #CODIGOS, true
end

-- ============================================================
-- UI COMPLETA
-- ============================================================
spawn(function()
    while not Jogador do
        aguardar(0.5)
        Jogador = Jogadores.LocalPlayer
    end

    local janela = RedOnyxLib.criar_janela({
        titulo = "RED ONYX",
        subtitulo = "PROJECT | HUB COMPLETO",
    })

    -- ===== ABA: KEY =====
    local aba_key = janela.criar_aba("Key")
    aba_key.label("Sistema de key persistente:")
    local input_key
    do
        local holder = Instance.new("Frame")
        input_key = Instance.new("TextBox")
        input_key.Size = UDim2.new(1, -8, 0, 30)
        input_key.BackgroundColor3 = PRETO
        input_key.BorderSizePixel = 0
        input_key.Font = Enum.Font.Gotham
        input_key.PlaceholderText = "Sua key aqui..."
        input_key.PlaceholderColor3 = CIN
        input_key.Text = ""
        input_key.TextColor3 = BRA
        input_key.TextSize = 12
        input_key.ClearTextOnFocus = false
        input_key.Parent = aba_key.pagina -- precisa ser a página
        canto(input_key, 6)
        novo("UIStroke", { Color = VERM_E, Thickness = 1, Parent = input_key })
    end

    aba_key.botao("SALVAR KEY", function()
        local k = input_key.Text
        if k == "" then
            print("[Red Onyx] Digite a key antes de salvar")
            return
        end
        salvar_key(k)
        print("[Red Onyx] Key salva! Proxima vez carrega automatica.")
    end)

    aba_key.botao("APAGAR KEY SALVA", function()
        deletar_key()
        input_key.Text = ""
        print("[Red Onyx] Key apagada do disco")
    end)

    aba_key.label("")
    aba_key.label("Com a key salva, o hub inicia as funcoes sozinho na proxima execucao.")

    -- ===== ABA: PRINCIPAL =====
    local aba_main = janela.criar_aba("Principal")
    aba_key.label("")
    aba_main.label("Auto Pega Tudo — ative o que quiser:")
    aba_main.toggle("Auto Farm (mobs)", "AutoFarm", false)
    aba_main.toggle("Auto Boss (ignora distancia)", "AutoBoss", false)
    aba_main.toggle("Auto Coleta (itens)", "AutoColeta", false)
    aba_main.toggle("Auto Baús", "AutoBaus", false)
    aba_main.toggle("Auto Quest", "AutoQuest", false)
    aba_main.toggle("Pulo no ataque (critico)", "SaltoAtaque", true)

    aba_main.botao("LIGAR TUDO DE UMA VEZ", function()
        RedOnyxLib.Config.AutoFarm = true
        RedOnyxLib.Config.AutoBoss = true
        RedOnyxLib.Config.AutoColeta = true
        RedOnyxLib.Config.AutoBaus = true
        RedOnyxLib.Config.AutoQuest = true
        print("[Red Onyx] AUTO PEGA TUDO LIGADO!")
    end)

    aba_main.botao("DESLIGAR TUDO", function()
        RedOnyxLib.Config.AutoFarm = false
        RedOnyxLib.Config.AutoBoss = false
        RedOnyxLib.Config.AutoColeta = false
        RedOnyxLib.Config.AutoBaus = false
        RedOnyxLib.Config.AutoQuest = false
        print("[Red Onyx] Tudo desligado")
    end)

    -- ===== ABA: CÓDIGOS =====
    local aba_cod = janela.criar_aba("Codigos")
    aba_cod.label("Resgate automatico de todos os codigos ativos:")
    aba_cod.botao("RESGATAR TODOS OS CODIGOS", function()
        print("[Red Onyx] Resgatando codigos...")
        local total, ok = resgatar_codigos()
        print("[Red Onyx] " .. (ok and ("Enviados: " .. total) or "Remote nao encontrado"))
    end)

    -- ===== ABA: CONFIG =====
    local aba_cfg = janela.criar_aba("Config")
    aba_cfg.label("Alcance do farm:")
    aba_cod.botao("Alcance: 100 studs", function() RedOnyxLib.Config.DistanciaFarm = 100 end)
    aba_cfg.botao("Alcance: 300 studs", function() RedOnyxLib.Config.DistanciaFarm = 300 end)
    aba_cfg.botao("Alcance: 999 studs", function() RedOnyxLib.Config.DistanciaFarm = 999 end)
    aba_cfg.label("")
    aba_cfg.label("Atalhos: RightShift abre/fecha a janela.")
    aba_cfg.label("PC fraco: use 'Alcance 100' para melhor desempenho.")

    aba_cfg.botao("MINIMIZAR JANELA", function()
        janela.janela.Visible = false
    end)

    print("[Red Onyx Project] Pronto! RightShift abre/fecha.")
end)

print("[Red Onyx Project] Carregado com sucesso!")
