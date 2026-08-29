-- ============================================================
-- RED ONYX PROJECT - BIBLIOTECA + HUB COMPLETO
-- 100% autônomo | Zero download | Design futurista
-- ============================================================
print("[Red Onyx Project] Iniciando...")

-- ===== SERVIÇOS =====
local Jogadores      = game:GetService("Players")
local UserInput      = game:GetService("UserInputService")
local TweenService   = game:GetService("TweenService")
local RunService     = game:GetService("RunService")
local ContextAction  = game:GetService("ContextActionService")

local Jogador = Jogadores.LocalPlayer

-- ===== COMPAT =====
local aguardar = (task and task.wait) or wait
local spawn    = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== PALETA =====
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
-- BIBLIOTECA RED ONYX (RedOnyxLib)
-- ============================================================
local RedOnyxLib = {}
RedOnyxLib.__index = RedOnyxLib

-- Config central (os loops de farm leem daqui)
RedOnyxLib.Config = {
    AutoFarm      = false,
    AutoColeta    = false,
    AutoBaus      = false,
    AutoQuest     = false,
    DistanciaFarm = 200,
}

-- ---------- helpers visuais ----------
local function novo(classe, props, pai)
    local obj = Instance.new(classe)
    for k, v in pairs(props) do
        obj[k] = v
    end
    obj.Parent = props.Parent
    return obj
end

local function canto(pai, raio)
    return novo("UICorner", { CornerRadius = UDim.new(0, raio or 8), Parent = pai })
end

local function borda(pai, cor, espessura, transp)
    return novo("UIStroke", {
        Color = cor or VERM,
        Thickness = espessura or 1,
        Transparency = transparencia or 0,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Parent = pai,
    })
end

local function gradiente(pai, c1, c2)
    return novo("UIGradient", {
        Color = ColorSequence.new(c1, c2),
        Rotation = 90,
        Parent = pai,
    })
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

    local ok_pai = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not ok_pai then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg.Parent = hui end
            end
        end)
    end

    -- Janela
    local janela = novo("Frame", {
        Name = "Janela",
        Size = UDim2.new(0, 460, 0, 320),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = FUNDO,
        BorderSizePixel = 0,
        Parent = sg,
    })
    canto(janela, 12)
    local stroke_janela = novo("UIStroke", { Color = VERM_E, Thickness = 1.5, Parent = janela })

    -- Glow futurista (stroke animado)
    spawn(function()
        while janela.Parent do
            local tw = TweenService:Create(stroke_janela, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
                Color = NEON, Transparency = 0.15
            })
            tw:Play()
            tw.Completed:Wait()
            if not janela.Parent then break end
            local tw2 = TweenService:Create(stroke_janela, TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
                Color = VERM_E, Transparency = 0.4
            })
            tw2:Play()
            tw2.Completed:Wait()
        end
    end)

    -- Header com gradiente
    local header = novo("Frame", {
        Name = "Header",
        Size = UDim2.new(1, 0, 0, 46),
        BackgroundColor3 = PAINEL,
        BorderSizePixel = 0,
        Parent = janela,
    })
    canto(header, 12)
    gradiente(header, Color3.fromRGB(45, 12, 18), FUNDO)

    -- Linha neon sob o header
    novo("Frame", {
        Name = "NeonLine",
        Size = UDim2.new(1, -24, 0, 1),
        Position = UDim2.new(0, 12, 0, 46),
        BackgroundColor3 = VERM,
        BorderSizePixel = 0,
        Parent = janela,
    })

    -- Logo/título
    novo("TextLabel", {
        Size = UDim2.new(0, 200, 0, 26),
        Position = UDim2.new(0, 14, 0, 7),
        BackgroundTransparency = 1,
        Font = Enum.Font.GothamBlack,
        Text = "RED ONYX",
        TextColor3 = VERM_C,
        TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header,
    })
    novo("TextLabel", {
        Size = UDim2.new(0, 240, 0, 12),
        Position = UDim2.new(0, 15, 0, 28),
        BackgroundTransparency = 1,
        Font = Enum.Font.Gotham,
        Text = cfg.subtitulo or "PROJECT | v1.0",
        TextColor3 = CIN,
        TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header,
    })

    -- Botão fechar
    local btn_fechar = novo("TextButton", {
        Size = UDim2.new(0, 26, 0, 26),
        Position = UDim2.new(1, -34, 0, 10),
        BackgroundColor3 = VERM_E,
        Font = Enum.Font.GothamBold,
        Text = "X",
        TextColor3 = BRA,
        TextSize = 12,
        AutoButtonColor = false,
        Parent = header,
    })
    canto(btn_fechar, 6)

    -- Arrastar janela
    do
        local arrastando, pos_inicial, ini_janela = false, nil, nil
        header.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                arrastando = true
                pos_inicial = input.Position
                ini_janela = janela.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        arrastando = false
                    end
                end)
            end
        end)
        UserInput.InputChanged:Connect(function(input)
            if arrastando and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local delta = input.Position - pos_inicial
                janela.Position = UDim2.new(ini_janela.X.Scale, ini_janela.X.Offset + delta.X, ini_janela.Y.Scale, ini_janela.Y.Offset + delta.Y)
            end
        end)
    end

    -- Barra lateral de tabs
    local coluna_tabs = novo("Frame", {
        Name = "Tabs",
        Size = UDim2.new(0, 110, 1, -58),
        Position = UDim2.new(0, 10, 0, 56),
        BackgroundTransparency = 1,
        Parent = janela,
    })
    local layout_tabs = novo("UIListLayout", {
        Padding = UDim.new(0, 6),
        Parent = coluna_tabs,
    })

    -- Área de conteúdo
    local conteudo = novo("Frame", {
        Name = "Conteudo",
        Size = UDim2.new(1, -132, 1, -66),
        Position = UDim2.new(0, 122, 0, 56),
        BackgroundColor3 = PRETO,
        BackgroundTransparency = 0.35,
        BorderSizePixel = 0,
        Parent = janela,
    })
    canto(conteudo, 8)

    -- API da janela
    local api = {}
    local abas = {}

    function api.criar_aba(nome)
        local btn_tab = novo("TextButton", {
            Size = UDim2.new(1, 0, 0, 28),
            BackgroundColor3 = CARD,
            Font = Enum.Font.GothamBold,
            Text = nome,
            TextColor3 = CIN,
            TextSize = 12,
            AutoButtonColor = false,
            Parent = coluna_tabs,
        })
        canto(btn_tab, 6)

        local pagina = novo("ScrollingFrame", {
            Name = "Pag_" .. nome,
            Size = UDim2.new(1, -8, 1, -8),
            Position = UDim2.new(0, 4, 0, 4),
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = VERM,
            CanvasSize = UDim2.new(0, 0, 0, 0),
            AutomaticCanvasSize = Enum.AutomaticSize.Y,
            Visible = false,
            Parent = conteudo,
        })
        local lay = novo("UIListLayout", { Padding = UDim.new(0, 6), Parent = pagina })

        local aba = { btn = btn_tab, pagina = pagina, nome = nome }
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

        -- ---------- componentes da aba ----------
        local componentes = {}

        function componentes.botao(txt, callback)
            local btn = novo("TextButton", {
                Size = UDim2.new(1, -8, 0, 30),
                BackgroundColor3 = CARD,
                Font = Enum.Font.GothamBold,
                Text = txt,
                TextColor3 = BRA,
                TextSize = 11,
                AutoButtonColor = false,
                Parent = pagina,
            })
            canto(btn, 6)
            novo("UIStroke", { Color = VERM_E, Thickness = 1, Parent = btn })
            btn.MouseButton1Click:Connect(function()
                spawn(callback)
            end)
            return btn
        end

        function componentes.toggle(txt, chave_config, padrao)
            RedOnyxLib.Config[chave_config] = padrao and true or false

            local row = novo("TextButton", {
                Size = UDim2.new(1, -8, 0, 30),
                BackgroundColor3 = CARD,
                Text = "",
                AutoButtonColor = false,
                Parent = pagina,
            })
            canto(row, 6)

            novo("TextLabel", {
                Size = UDim2.new(1, -60, 1, 0),
                Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1,
                Font = Enum.Font.Gotham,
                Text = txt,
                TextColor3 = BRA,
                TextSize = 11,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = row,
            })

            -- Switch futurista
            local trilho = novo("Frame", {
                Size = UDim2.new(0, 36, 0, 16),
                Position = UDim2.new(1, -46, 0.5, 0),
                AnchorPoint = Vector2.new(0, 0.5),
                BackgroundColor3 = PAINEL,
                BorderSizePixel = 0,
                Parent = row,
            })
            canto(trilho, 8)
            local bolinha = novo("Frame", {
                Size = UDim2.new(0, 12, 0, 12),
                Position = UDim2.new(0, 2, 0.5, 0),
                AnchorPoint = Vector2.new(0, 0.5),
                BackgroundColor3 = CIN,
                BorderSizePixel = 0,
                Parent = trilho,
            })
            canto(trilho, 8)
            canto(bolinha, 8)

            local ligado = RedOnyxLib.Config[chave_config]

            local function atualizar(instante)
                if ligado then
                    trilho.BackgroundColor3 = VERM_E
                    bolinha.BackgroundColor3 = VERM_C
                    bolinha:TweenPosition(UDim2.new(1, -16, 0.5, 0), "Out", "Quad", 0.15, true)
                else
                    trilho.BackgroundColor3 = PAINEL
                    bolinha.BackgroundColor3 = CIN
                    bolinha:TweenPosition(UDim2.new(0, 2, 0.5, 0), "Out", "Quad", 0.15, true)
                end
            end

            atualizar(true)

            row.MouseButton1Click:Connect(function()
                ligado = not ligado
                RedOnyxLib.Config[chave_config] = ligado
                atualizar()
                print("[Red Onyx] " .. txt .. ": " .. (ligado and "ON" or "OFF"))
            end)

            return row
        end

        function componentes.label(txt)
            local l = novo("TextLabel", {
                Size = UDim2.new(1, -8, 0, 22),
                BackgroundTransparency = 1,
                Font = Enum.Font.Gotham,
                Text = txt,
                TextColor3 = CIN,
                TextSize = 10,
                TextXAlignment = Enum.TextXAlignment.Left,
                TextWrapped = true,
                Parent = pagina,
            })
            return l
        end

        return componentes
    end

    btn_fechar.MouseButton1Click:Connect(function()
        janela.Visible = not janela.Visible
    end)

    -- Atalho: RightShift mostra/esconde
    spawn(function()
        UserInput.InputBegan:Connect(function(input, processado)
            if processado then return end
            if input.KeyCode == Enum.KeyCode.RightShift then
                janela.Visible = not janela.Visible
            end
        end)
    end)

    api.janela = janela
    return api
end

-- ============================================================
-- MÓDULO DE FARM (usa RedOnyxLib.Config)
-- ============================================================
local function pegar_hrp()
    local char = Jogador and Jogador.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart")
end

local function mob_mais_proximo(hrp, distancia_max)
    local alvo, dist_alvo = nil, math.huge
    local char = Jogador.Character
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj:IsA("Model") and obj ~= char then
            local hum = obj:FindFirstChildOfClass("Humanoid")
            local raiz = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
            if hum and raiz and hum.Health > 0 then
                local d = (hrp.Position - raiz.Position).Magnitude
                if d < dist_alvo then
                    dist_alvo = d
                    alvo = raiz
                end
            end
        end
    end
    if dist_alvo <= (distancia_max or 300) then
        return alvo, dist_alvo
    end
    return nil, nil
end

-- ===== AUTO FARM =====
spawn(function()
    while true do
        if RedOnyxLib.Config.AutoFarm then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                local alvo = mob_mais_proximo(hrp, RedOnyxLib.Config.DistanciaFarm)
                if alvo then
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 8, 4)
                    local char = Jogador.Character
                    local ferr = char and char:FindFirstChildOfClass("Tool")
                    if ferr then pcall(function() ferr:Activate() end) end
                end
            end)
        end
        aguardar(RedOnyxLib.Config.AutoFarm and 0.25 or 1)
    end
end)

-- ===== AUTO COLETA + BAÚS =====
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
                                break -- 1 item por ciclo (leve)
                            end
                        end
                    end
                end
            end)
        end
        aguardar(0.4)
    end
end)

-- ===== AUTO QUEST =====
spawn(function()
    while true do
        if RedOnyxLib.Config.AutoQuest then
            pcall(function()
                local hrp = pegar_hrp()
                if not hrp then return end
                for _, obj in ipairs(workspace:GetDescendants()) do
                    if obj:IsA("ProximityPrompt") and obj.Enabled then
                        local parte = obj.Parent
                        if parte and (parte.Position - hrp.Position).Magnitude < 12 then
                            pcall(fireproximityprompt, obj)
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

-- ===== RESGATAR CÓDIGOS =====
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
    local Storage = game:GetService("ReplicatedStorage")
    local remote = Storage:FindFirstChild("Redeem") or Storage:FindFirstChild("RedeemCode")
    if not remote then
        warn("[Red Onyx] Remote de codigos nao encontrado")
        return 0, false
    end
    local ok_count = 0
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
-- MONTAR A INTERFACE
-- ============================================================
spawn(function()
    -- Espera o jogador existir (servidor demora às vezes)
    while not Jogador do
        aguardar(0.5)
        Jogador = Jogadores.LocalPlayer
    end

    local janela = RedOnyxLib.criar_janela({
        titulo = "RED ONYX HUB",
        subtitulo = "PROJECT | Auto Pega Tudo",
    })

    -- ===== ABA PRINCIPAL =====
    local principal = janela.criar_aba("Principal")

    principal.label("Funcoes automaticas — ligue o que quiser:")

    principal.toggle("Auto Farm (mobs)", "AutoFarm", false)
    principal.toggle("Auto Coleta (itens)", "AutoColeta", false)
    principal.toggle("Auto Baús", "AutoBaus", false)
    principal.toggle("Auto Quest", "AutoQuest", false)

    principal.botao("RESGATAR TODOS OS CÓDIGOS", function()
        print("[Red Onyx] Resgatando codigos...")
        local total, ok = resgatar_codigos()
        print("[Red Onyx] " .. (ok and ("Codigos enviados: " .. total) or "Remote nao encontrado"))
    end)

    -- ===== ABA VELOCIDADE =====
    local config_aba = janela.criar_aba("Config")

    config_aba.label("Alcance do farm (studs):")
    config_aba.botao("Alcance: 100 (curto)", function()
        RedOnyxLib.Config.DistanciaFarm = 100
        print("[Red Onyx] Alcance: 100")
    end)
    config_aba.botao("Alcance: 300 (médio)", function()
        RedOnyxLib.Config.DistanciaFarm = 300
        print("[Red Onyx] Alcance: 300")
    end)
    config_aba.botao("Alcance: 999 (tudo)", function()
        RedOnyxLib.Config.DistanciaFarm = 999
        print("[Red Onyx] Alcance: 999")
    end)

    config_aba.label("")
    config_aba.label("Atalho: RightShift mostra/esconde a janela.")

    config_aba.botao("DESATIVAR TUDO", function()
        RedOnyxLib.Config.AutoFarm = false
        RedOnyxLib.Config.AutoColeta = false
        RedOnyxLib.Config.AutoBaus = false
        RedOnyxLib.Config.AutoQuest = false
        print("[Red Onyx] Tudo desativado")
    end)

    print("[Red Onyx Project] Pronto! Use RightShift para abrir/fechar.")
end)

print("[Red Onyx Project] Carregado!")
