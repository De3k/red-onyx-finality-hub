-- ============================================================
-- RED ONYX PROJECT v3 - FINAL
-- Key salva + Auto Pega Tudo + UI futurista vermelho/preto
-- 100% standalone | Sem downloads | Sem erros
-- ============================================================
print("[Red Onyx] v3 iniciando...")

-- ===== SERVIÇOS =====
local Players = game:GetService("Players")
local UserInput = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Replicated = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")

-- ===== COMPAT =====
local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

local Jogador = Players.LocalPlayer
while not Jogador do
    aguardar(0.5)
    Jogador = Players.LocalPlayer
end

-- ===== PALETA =====
local VERM   = Color3.fromRGB(225, 25, 45)
local VERM_C = Color3.fromRGB(255, 90, 105)
local VERM_E = Color3.fromRGB(115, 12, 26)
local NEON   = Color3.fromRGB(255, 40, 60)
local PRETO  = Color3.fromRGB(10, 8, 10)
local FUNDO  = Color3.fromRGB(16, 12, 14)
local PAINEL = Color3.fromRGB(24, 17, 20)
local CARD   = Color3.fromRGB(30, 21, 25)
local BRA    = Color3.fromRGB(245, 240, 242)
local CIN    = Color3.fromRGB(150, 140, 145)

-- ===== HELPER =====
local function novo(classe, props)
    local obj = Instance.new(classe)
    for k, v in pairs(props) do
        if k ~= "Parent" then obj[k] = v end
    end
    obj.Parent = props.Parent
    return obj
end

local function canto(pai, raio)
    novo("UICorner", { CornerRadius = UDim.new(0, raio or 8), Parent = pai })
end

-- ============================================================
-- CONFIG (os loops leem daqui)
-- ============================================================
local Config = {
    Farm      = false,
    Boss      = false,
    Coleta    = false,
    Baus      = false,
    Quest     = false,
    PuloAtk   = true,
    Alcance   = 300,
}

-- ============================================================
-- KEY SYSTEM (disco)
-- ============================================================
local ARQ_KEY = "RedOnyxKey.txt"

local function salvar_key(k)
    pcall(function()
        if makefolder then pcall(makefolder, "RedOnyx") end
        writefile(ARQ_KEY, k)
    end)
    pcall(function()
        if syn and syn.writefile then syn.writefile(ARQ_KEY, k) end
    end)
end

local function carregar_key()
    local k = ""
    pcall(function() k = readfile(ARQ_KEY) end)
    if type(k) ~= "string" or k == "" then
        pcall(function()
            if syn and syn.readfile then k = syn.readfile(ARQ_KEY) end
        end)
    end
    if type(k) ~= "string" then k = "" end
    return k
end

local function deletar_key()
    pcall(delfile, ARQ_KEY)
    pcall(function() if syn and syn.delfile then syn.delfile(ARQ_KEY) end end)
end

-- ============================================================
-- FUNÇÕES DE FARM
-- ============================================================
local function pegar_hrp()
    local char = Jogador.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso")
end

local function achar_alvo(hrp, dist_max, so_boss)
    local alvo = nil
    local menor = math.huge
    local char = Jogador.Character
    local filhos = workspace:GetChildren()
    for i = 1, #filhos do
        local obj = filhos[i]
        if obj:IsA("Model") and obj ~= char then
            local hum = obj:FindFirstChildOfClass("Humanoid")
            local raiz = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
            if hum and raiz and hum.Health > 0 then
                local passou = true
                if so_boss then
                    local n = obj.Name:lower()
                    passou = n:find("boss") or n:find("king") or n:find("captain") or n:find("leviathan") or n:find("ripper")
                end
                if passou then
                    local d = (hrp.Position - raiz.Position).Magnitude
                    if d < dist_max and d < menor then
                        menor = d
                        alvo = raiz
                    end
                end
            end
        end
    end
    return alvo
end

-- ===== LOOP: AUTO FARM =====
spawn(function()
    while true do
        if Config.Farm then
            pcall(function()
                local char = Jogador.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                local alvo = achar_alvo(hrp, Config.Alcance, false)
                if alvo then
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 8, 4)
                    local ferr = char:FindFirstChildOfClass("Tool")
                    if ferr then pcall(function() ferr:Activate() end) end
                    if Config.PuloAtk then
                        local hum = char:FindFirstChildOfClass("Humanoid")
                        if hum then pcall(function() hum.Jump = true end) end
                    end
                end
            end)
            aguardar(0.25)
        else
            aguardar(1)
        end
    end
end)

-- ===== LOOP: AUTO BOSS =====
spawn(function()
    while true do
        if Config.Boss then
            pcall(function()
                local char = Jogador.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                local alvo = achar_alvo(hrp, 99999, true)
                if alvo then
                    hrp.CFrame = alvo.CFrame * CFrame.new(0, 8, 4)
                    local ferr = char:FindFirstChildOfClass("Tool")
                    if ferr then pcall(function() ferr:Activate() end) end
                end
            end)
            aguardar(0.3)
        else
            aguardar(2)
        end
    end
end)

-- ===== LOOP: AUTO COLETA + BAÚS =====
spawn(function()
    while true do
        if Config.Coleta or Config.Baus then
            pcall(function()
                local char = Jogador.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                local objs = workspace:GetDescendants()
                for i = 1, #objs do
                    local obj = objs[i]
                    if obj:IsA("BasePart") and obj.Transparency < 1 then
                        local n = obj.Name:lower()
                        local quer = false
                        if Config.Baus and (n:find("chest") or n:find("bau")) then quer = true end
                        if Config.Coleta and (obj:FindFirstChild("TouchInterest") or obj:FindFirstChildOfClass("ClickDetector")) then quer = true end
                        if quer then
                            local d = (hrp.Position - obj.Position).Magnitude
                            if d < 60 then
                                hrp.CFrame = CFrame.new(obj.Position + Vector3.new(0, 2, 0))
                                if firetouchinterest then
                                    pcall(firetouchinterest, hrp, obj, 0)
                                    pcall(firetouchinterest, hrp, obj, 1)
                                end
                                break -- 1 item por ciclo = leve
                            end
                        end
                    end
                end
            end)
            aguardar(0.4)
        else
            aguardar(1)
        end
    end
end)

-- ===== LOOP: AUTO QUEST =====
spawn(function()
    while true do
        if Config.Quest then
            pcall(function()
                local char = Jogador.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                local prompts = workspace:GetDescendants()
                for i = 1, #prompts do
                    local pp = prompts[i]
                    if pp:IsA("ProximityPrompt") and pp.Enabled then
                        local pai = pp.Parent
                        if pai and pai:IsA("BasePart") then
                            if (pai.Position - hrp.Position).Magnitude < 12 then
                                pcall(fireproximityprompt, pp)
                                break
                            end
                        elseif pai then
                            local parte = pai:FindFirstChildWhichIsA("BasePart")
                            if parte and (parte.Position - hrp.Position).Magnitude < 12 then
                                pcall(fireproximityprompt, pp)
                                break
                            end
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
    local remote = Replicated:FindFirstChild("Redeem") or Replicated:FindFirstChild("RedeemCode")
    if not remote then
        warn("[Red Onyx] Remote de codigos nao encontrado")
        return false
    end
    for i = 1, #CODIGOS do
        local cod = CODIGOS[i]
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
    print("[Red Onyx] Codigos enviados: " .. #CODIGOS)
    return true
end

-- ============================================================
-- UI FUTURISTA (única, com scroll)
-- ============================================================
local ok_ui, erro_ui = pcall(function()

    local sg = novo("ScreenGui", {
        Name = "RedOnyxProject",
        IgnoreGuiInset = true,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })
    local pai_ok = pcall(function() sg.Parent = Core end)
    if not pai_ok then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg.Parent = hui end
            end
        end)
    end

    -- Janela
    local janela = novo("Frame", {
        Size = UDim2.new(0, 420, 0, 360),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = FUNDO,
        BorderSizePixel = 0,
        Active = true,
        Parent = sg,
    })
    canto(janela, 12)
    local stroke = novo("UIStroke", { Color = VERM_E, Thickness = 1.5, Parent = janela })

    -- Glow neon pulsante (sem travar: só 2 tweens por ciclo)
    spawn(function()
        while janela.Parent do
            local a = TweenService:Create(stroke, TweenInfo.new(1.5, Enum.EasingStyle.Sine), { Color = NEON, Transparency = 0.1 })
            a:Play()
            aguardar(1.5)
            if not janela.Parent then break end
            local b = TweenService:Create(stroke, TweenInfo.new(1.5, Enum.EasingStyle.Sine), { Color = VERM_E, Transparency = 0.4 })
            b:Play()
            aguardar(1.5)
        end
    end)

    -- Header
    local header = novo("Frame", {
        Size = UDim2.new(1, 0, 0, 44),
        BackgroundColor3 = PAINEL,
        BorderSizePixel = 0,
        Parent = janela,
    })
    canto(header, 12)
    novo("UIGradient", {
        Color = ColorSequence.new(Color3.fromRGB(50, 12, 20), FUNDO),
        Rotation = 90,
        Parent = header,
    })

    novo("TextLabel", {
        Size = UDim2.new(0, 220, 0, 24), Position = UDim2.new(0, 14, 0, 6),
        BackgroundTransparency = 1, Font = Enum.Font.GothamBlack,
        Text = "RED ONYX", TextColor3 = VERM_C, TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left, Parent = header,
    })
    novo("TextLabel", {
        Size = UDim2.new(0, 260, 0, 12), Position = UDim2.new(0, 15, 0, 27),
        BackgroundTransparency = 1, Font = Enum.Font.Gotham,
        Text = "PROJECT | AUTO PEGA TUDO", TextColor3 = CIN, TextSize = 10,
        TextXAlignment = Enum.TextXAlignment.Left, Parent = header,
    })

    -- Arrastar
    local arrastando = false
    local p0, j0
    header.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            arrastando = true
            p0 = inp.Position
            j0 = janela.Position
        end
    end)
    header.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            arrastando = false
        end
    end)
    UserInput.InputChanged:Connect(function(inp)
        if arrastando and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) and p0 then
            local d = inp.Position - p0
            janela.Position = UDim2.new(0, j0.X.Offset + d.X, 0, j0.Y.Offset + d.Y)
        end
    end)

    -- Página com scroll
    local pagina = novo("ScrollingFrame", {
        Size = UDim2.new(1, -16, 1, -56),
        Position = UDim2.new(0, 8, 0, 50),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = VERM,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = janela,
    })
    novo("UIListLayout", { Padding = UDim.new(0, 6), Parent = pagina })

    -- ===== COMPONENTES =====
    local function label(txt)
        return novo("TextLabel", {
            Size = UDim2.new(1, -8, 0, 20),
            BackgroundTransparency = 1,
            Font = Enum.Font.GothamBold,
            Text = txt, TextColor3 = VERM_C, TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = pagina,
        })
    end

    local function sub(txt)
        return novo("TextLabel", {
            Size = UDim2.new(1, -8, 0, 16),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            Text = txt, TextColor3 = CIN, TextSize = 10,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextWrapped = true,
            Parent = pagina,
        })
    end

    local function botao(txt, cb)
        local btn = novo("TextButton", {
            Size = UDim2.new(1, -8, 0, 30),
            BackgroundColor3 = CARD,
            Font = Enum.Font.GothamBold,
            Text = txt, TextColor3 = BRA, TextSize = 11,
            AutoButtonColor = false,
            Parent = pagina,
        })
        canto(btn, 6)
        novo("UIStroke", { Color = VERM_E, Thickness = 1, Parent = btn })
        btn.MouseButton1Click:Connect(function() spawn(cb) end)
        return btn
    end

    local function toggle(txt, chave, padrao)
        Config[chave] = padrao and true or false

        local row = novo("TextButton", {
            Size = UDim2.new(1, -8, 0, 30),
            BackgroundColor3 = CARD,
            Text = "", AutoButtonColor = false,
            Parent = pagina,
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
            Position = UDim2.new(1, -46, 0.5, 0),
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundColor3 = PAINEL,
            BorderSizePixel = 0, Parent = row,
        })
        canto(trilho, 8)
        local bolinha = novo("Frame", {
            Size = UDim2.new(0, 12, 0, 12),
            Position = UDim2.new(0, 2, 0.5, 0),
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundColor3 = CIN,
            BorderSizePixel = 0, Parent = trilho,
        })
        canto(bolinha, 6)

        local ligado = Config[chave]
        local function refresh()
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
        refresh()

        row.MouseButton1Click:Connect(function()
            ligado = not ligado
            Config[chave] = ligado
            refresh()
 matters        end)
        return row
    end

    -- ===== STATUS =====
    local status_lbl = sub("Aguardando...")

    -- ===== SEÇÃO: KEY =====
    sub("=== KEY ===")
    local input = novo("TextBox", {
        Size = UDim2.new(1, -8, 0, 30),
        BackgroundColor3 = PRETO,
        BorderSizePixel = 0,
        Font = Enum.Font.Gotham,
        PlaceholderText = "Digite sua key aqui...",
        PlaceholderColor3 = CIN,
        Text = carregar_key(), -- já mostra a key salva
        TextColor3 = BRA,
        TextSize = 12,
        ClearTextOnFocus = false,
        Parent = pagina,
    })
    canto(input, 6)
    novo("UIStroke", { Color = VERM_E, Thickness = 1, Parent = input })

    botao("SALVAR KEY", function()
        local k = input.Text
        if k == "" then
            status("Digite uma key antes de salvar!")
            return
        end
        salvar_key(k)
        print("[Red Onyx] Key salva em disco!")
    end)

    botao("APAGAR KEY SALVA", function()
        deletar_key()
        input.Text = ""
        print("[Red Onyx] Key apagada")
    end)

    -- ===== SEÇÃO: AUTO PEGA TUDO =====
    label("")
    label("AUTO PEGA TUDO")
    toggle("Auto Farm (mobs)", "Farm", false)
    toggle("Auto Boss", "Boss", false)
    toggle("Auto Coleta (itens)", "Coleta", false)
    toggle("Auto Baús", "Baus", false)
    toggle("Auto Quest", "Quest", false)
    toggle("Pulo crítico", "PuloAtk", true)

    botao("LIGAR TUDO (AUTO PEGA TUDO)", function()
        Config.Farm = true
        Config.Boss = true
        Config.Coleta = true
        Config.Baus = true
        Config.Quest = true
        Config.PuloAtk = true
        print("[Red Onyx] TUDO LIGADO!")
    end)

    botao("DESLIGAR TUDO", function()
        Config.Farm = false
        Config.Boss = false
        Config.Coleta = false
        Config.Baus = false
        Config.Quest = false
        print("[Red Onyx] Tudo desligado")
    end)

    -- ===== SEÇÃO: CÓDIGOS =====
    label("")
    label("CÓDIGOS")
    botao("RESGATAR TODOS OS CODIGOS (28)", function()
        print("[Red Onyx] Resgatando...")
        resgatar_codigos()
    end)

    -- ===== SEÇÃO: CONFIG =====
    label("")
    label("ALCANCE DO FARM")
    botao("Alcance: 100 studs (PC fraco)", function() Config.Alcance = 100 end)
    botao("Alcance: 300 studs", function() Config.Alcance = 300 end)
    botao("Alcance: 999 studs", function() Config.Alcance = 999 end)

    label("")
    sub("RightShift = abrir/fechar | PC fraco: Alcance 100")

    -- ===== BOTÃO MINIMIZAR =====
    local btn_min = novo("TextButton", {
        Size = UDim2.new(0, 26, 0, 26),
        Position = UDim2.new(1, -34, 0, 10),
        BackgroundColor3 = VERM_E,
        Font = Enum.Font.GothamBold,
        Text = "—",
        TextColor3 = BRA, TextSize = 12,
        AutoButtonColor = false, Parent = header,
    })
    canto(btn_min, 6)
    btn_min.MouseButton1Click:Connect(function()
        janela.Visible = not janela.Visible
    end)

    -- Atalho RightShift
    UserInput.InputBegan:Connect(function(inp, proc)
        if proc then return end
        if inp.KeyCode == Enum.KeyCode.RightShift then
            janela.Visible = not janela.Visible
        end
    end)

end)

if not ok_ui then
    warn("[Red Onyx] ERRO NA UI: " .. tostring(erro_ui))
    -- Fallback: ativa tudo direto sem UI
    spawn(function()
        aguardar(3)
        Config.Farm = true
        Config.Coleta = true
        Config.Baus = true
        Config.Quest = true
        print("[Red Onyx] UI falhou - funcoes ativadas direto")
    end)
end

print("[Red Onyx] v3 carregado! RightShift abre/fecha.")
