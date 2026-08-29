-- ============================================================
-- RED ONYX PROJECT v4 - AUTODIAGNÓSTICO
-- Se der erro, ele aparece NA TELA (não precisa do F9)
-- ============================================================

local Players = game:GetService("Players")
local UserInput = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Replicated = game:GetService("ReplicatedStorage")

local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

local Jogador = Players.LocalPlayer
while not Jogador do
    aguardar(0.5)
    Jogador = Players.LocalPlayer
end

-- ===== HELPER DE INSTÂNCIA =====
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

-- ===== MOSTRAR ERRO NA TELA (autodiagnóstico) =====
local function mostrar_erro(msg)
    pcall(function()
        local sg = Instance.new("ScreenGui")
        sg.Name = "RedOnyx_Erro"
        local ok = pcall(function() sg.Parent = game:GetService("CoreGui") end)
        if not ok then sg.Parent = Jogador:WaitForChild("PlayerGui") end

        local box = novo("TextLabel", {
            Size = UDim2.new(0, 420, 0, 120),
            Position = UDim2.new(0.5, 0, 0.85, 0),
            AnchorPoint = Vector2.new(0.5, 0),
            BackgroundColor3 = Color3.fromRGB(15, 12, 14),
            TextColor3 = Color3.fromRGB(255, 90, 105),
            TextSize = 12,
            Font = Enum.Font.Gotham,
            TextWrapped = true,
            Text = "RED ONYX - ERRO:\n" .. tostring(msg),
            BorderSizePixel = 0,
            Parent = sg,
        })
        canto(box, 8)
        novo("UIStroke", { Color = Color3.fromRGB(255, 40, 60), Thickness = 1.5, Parent = box })
    end)
end

-- ===== CONFIG =====
local Config = {
    Farm    = false,
    Boss    = false,
    Coleta  = false,
    Baus    = false,
    Quest   = false,
    PuloAtk = true,
    Alcance = 300,
}

-- ===== KEY (disco) =====
local ARQ_KEY = "RedOnyxKey.txt"

local function salvar_key(k)
    local ok = pcall(function()
        if makefolder then makefolder("RedOnyx") end
        writefile(ARQ_KEY, k)
    end)
    if not ok then
        pcall(function()
            if syn and syn.writefile then syn.writefile(ARQ_KEY, k) end
        end)
    end
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
    pcall(function() delfile(ARQ_KEY) end)
    pcall(function()
        if syn and syn.delfile then syn.delfile(ARQ_KEY) end
    end)
end

-- ===== CORE DO FARM =====
local function achar_alvo(hrp, dist_max, so_boss)
    local alvo = nil
    local menor = math.huge
    local char = Jogador.Character
    local lista = workspace:GetChildren()
    for i = 1, #lista do
        local obj = lista[i]
        if obj:IsA("Model") and obj ~= char then
            local hum = obj:FindFirstChildOfClass("Humanoid")
            local raiz = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
            if hum and raiz and hum.Health > 0 then
                local passou = true
                if so_boss then
                    local n = obj.Name:lower()
                    passou = (n:find("boss") ~= nil) or (n:find("king") ~= nil)
                        or (n:find("captain") ~= nil) or (n:find("soul") ~= nil)
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

-- LOOP: AUTO FARM
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

-- LOOP: AUTO BOSS
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

-- LOOP: AUTO COLETA + BAÚS
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
                                break
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

-- LOOP: AUTO QUEST
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
                        local parte = nil
                        if pai and pai:IsA("BasePart") then
                            parte = pai
                        elseif pai then
                            parte = pai:FindFirstChildWhichIsA("BasePart")
                        end
                        if parte and (parte.Position - hrp.Position).Magnitude < 12 then
                            if fireproximityprompt then
                                pcall(fireproximityprompt, pp)
                            end
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
    return true
end

-- ============================================================
-- UI (uma página só, com scroll — sem abas)
-- ============================================================
local ok_principal, erro_principal = xpcall(function()

    local sg = novo("ScreenGui", {
        Name = "RedOnyxProject",
        IgnoreGuiInset = true,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })

    local ok_pai = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not ok_pai then
        sg.Parent = Jogador:WaitForChild("PlayerGui")
    end

    -- Janela
    local janela = novo("Frame", {
        Size = UDim2.new(0, 400, 0, 350),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = Color3.fromRGB(16, 12, 14),
        BorderSizePixel = 0,
        Active = true,
        Parent = sg,
    })
    canto(janela, 12)
    local stroke = novo("UIStroke", { Color = Color3.fromRGB(115, 12, 26), Thickness = 1.5, Parent = janela })

    -- Glow pulsante: 1 tween com RepeatCount -1 (zero loops, zero travamento)
    local tw = TweenService:Create(
        stroke,
        TweenInfo.new(1.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        { Color = Color3.fromRGB(255, 40, 60), Transparency = 0.1 }
    )
    tw:Play()

    -- Header
    local header = novo("Frame", {
        Size = UDim2.new(1, 0, 0, 42),
        BackgroundColor3 = Color3.fromRGB(24, 17, 20),
        BorderSizePixel = 0,
        Parent = janela,
    })
    canto(header, 12)

    novo("TextLabel", {
        Size = UDim2.new(0, 200, 1, 0),
        Position = UDim2.new(0, 14, 0, 0),
        BackgroundTransparency = 1,
        Font = Enum.Font.GothamBlack,
        Text = "RED ONYX",
        TextColor3 = Color3.fromRGB(255, 90, 105),
        TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header,
    })

    -- Arrastar (guardado contra nil)
    local arrastando = false
    local p0 = nil
    local j0 = nil
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
        if arrastando and p0 and j0 then
            if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
                local d = inp.Position - p0
                janela.Position = UDim2.new(0, j0.X.Offset + d.X, 0, j0.Y.Offset + d.Y)
            end
        end
    end)

    -- Página
    local pagina = novo("ScrollingFrame", {
        Size = UDim2.new(1, -16, 1, -56),
        Position = UDim2.new(0, 8, 0, 50),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Color3.fromRGB(225, 25, 45),
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = janela,
    })
    novo("UIListLayout", { Padding = UDim.new(0, 6), Parent = pagina })

    -- Componentes
    local function sub(txt)
        return novo("TextLabel", {
            Size = UDim2.new(1, -8, 0, 18),
            BackgroundTransparency = 1,
            Font = Enum.Font.GothamBold,
            Text = txt,
            TextColor3 = Color3.fromRGB(255, 90, 105),
            TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = pagina,
        })
    end

    local function nota(txt)
        return novo("TextLabel", {
            Size = UDim2.new(1, -8, 0, 16),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            Text = txt,
            TextColor3 = Color3.fromRGB(150, 140, 145),
            TextSize = 10,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextWrapped = true,
            Parent = pagina,
        })
    end

    local function botao(txt, cb)
        local btn = novo("TextButton", {
            Size = UDim2.new(1, -8, 0, 30),
            BackgroundColor3 = Color3.fromRGB(30, 21, 25),
            Font = Enum.Font.GothamBold,
            Text = txt,
            TextColor3 = Color3.fromRGB(245, 240, 242),
            TextSize = 11,
            AutoButtonColor = false,
            Parent = pagina,
        })
        canto(btn, 6)
        novo("UIStroke", { Color = Color3.fromRGB(115, 12, 26), Thickness = 1, Parent = btn })
        btn.MouseButton1Click:Connect(function() spawn(cb) end)
        return btn
    end

    local refreshers = {}

    local function toggle(txt, chave, padrao)
        Config[chave] = padrao and true or false

        local row = novo("TextButton", {
            Size = UDim2.new(1, -8, 0, 30),
            BackgroundColor3 = Color3.fromRGB(30, 21, 25),
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
            TextColor3 = Color3.fromRGB(245, 240, 242),
            TextSize = 11,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = row,
        })

        local trilho = novo("Frame", {
            Size = UDim2.new(0, 36, 0, 16),
            Position = UDim2.new(1, -46, 0.5, 0),
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundColor3 = Color3.fromRGB(24, 17, 20),
            BorderSizePixel = 0,
            Parent = row,
        })
        canto(trilho, 8)
        local bolinha = novo("Frame", {
            Size = UDim2.new(0, 12, 0, 12),
            Position = UDim2.new(0, 2, 0.5, 0),
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundColor3 = Color3.fromRGB(150, 140, 145),
            BorderSizePixel = 0,
            Parent = trilho,
        })
        canto(bolinha, 6)

        local function refresh()
            if Config[chave] then
                trilho.BackgroundColor3 = Color3.fromRGB(115, 12, 26)
                bolinha.BackgroundColor3 = Color3.fromRGB(255, 90, 105)
                bolinha:TweenPosition(UDim2.new(1, -14, 0.5, 0), "Out", "Quad", 0.15, true)
            else
                trilho.BackgroundColor3 = Color3.fromRGB(24, 17, 20)
                bolinha.BackgroundColor3 = Color3.fromRGB(150, 140, 145)
                bolinha:TweenPosition(UDim2.new(0, 2, 0.5, 0), "Out", "Quad", 0.15, true)
            end
        end

        table.insert(refreshers, refresh)
        refresh()

        row.MouseButton1Click:Connect(function()
            Config[chave] = not Config[chave]
            refresh()
        end)
    end

    -- Botões em massa que atualizam a UI
    local function set_all(v)
        Config.Farm = v
        Config.Boss = v
        Config.Coleta = v
        Config.Baus = v
        Config.Quest = v
        for _, f in ipairs(refreshers) do
            pcall(f)
        end
    end

    -- ===== CONTEÚDO =====
    sub("KEY")
    local input = novo("TextBox", {
        Size = UDim2.new(1, -8, 0, 30),
        BackgroundColor3 = Color3.fromRGB(10, 8, 10),
        BorderSizePixel = 0,
        Font = Enum.Font.Gotham,
        PlaceholderText = "Digite sua key...",
        PlaceholderColor3 = Color3.fromRGB(120, 110, 112),
        Text = carregar_key(),
        TextColor3 = Color3.fromRGB(245, 240, 242),
        TextSize = 12,
        ClearTextOnFocus = false,
        Parent = pagina,
    })
    canto(input, 6)
    novo("UIStroke", { Color = Color3.fromRGB(115, 12, 26), Thickness = 1, Parent = input })

    novo("TextButton", {}):Destroy() -- no-op, removido

    local btn_salvar = botao("SALVAR KEY", function()
        if input.Text ~= "" then
            salvar_key(input.Text)
            input.Text = ""
            print("[Red Onyx] Key salva! Reinicie o script pra ver a key carregada.")
        end
    end)

    botao("APAGAR KEY SALVA", function()
        deletar_key()
        input.Text = ""
回家了    end)

    sub("")
    sub("AUTO PEGA TUDO")
    toggle("Auto Farm (mobs)", "Farm", false)
    toggle("Auto Boss", "Boss", false)
    toggle("Auto Coleta (itens)", "Coleta", false)
    toggle("Auto Baús", "Baus", false)
    toggle("Auto Quest", "Quest", false)
    toggle("Pulo crítico", "PuloAtk", true)

    botao("LIGAR TUDO (AUTO PEGA TUDO)", function()
        set_all(true)
每种    end)

    botao("DESLIGAR TUDO", function()
        set_all(false)
    end)

    sub("")
    sub("CÓDIGOS")
    botao("RESGATAR TODOS OS CODIGOS", function()
        local remote = Replicated:FindFirstChild("Redeem") or Replicated:FindFirstChild("RedeemCode")
        if not remote then
            warn("[Red Onyx] Remote nao encontrado")
            return
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
        print("[Red Onyx] Codigos enviados!")
    end)

    sub("")
    sub("ALCANCE DO FARM")
    botao("Alcance: 100 (PC fraco)", function() Config.Alcance = 100 end)
    botao("Alcance: 300", function() Config.Alcance = 300 end)
    botao("Alcance: 999", function() Config.Alcance = 999 end)

    sub("")
    nota("RightShift = abrir/fechar a janela")

    -- Minimizar
    local btn_min = novo("TextButton", {
        Size = UDim2.new(0, 26, 0, 26),
        Position = UDim2.new(1, -34, 0, 9),
        BackgroundColor3 = Color3.fromRGB(115, 12, 26),
        Font = Enum.Font.GothamBold,
        Text = "—",
        TextColor3 = Color3.fromRGB(245, 240, 242),
        TextSize = 12,
        AutoButtonColor = false,
        Parent = header,
    })
    canto(btn_min, 6)
    btn_min.MouseButton1Click:Connect(function()
        janela.Visible = not janela.Visible
    end)

    UserInput.InputBegan:Connect(function(inp, proc)
        if proc then return end
        if inp.KeyCode == Enum.KeyCode.RightShift then
            janela.Visible = not janela.Visible
        end
    end)

end, function(e)
    return tostring(e) .. "\n" .. (debug and debug.traceback and debug.traceback() or "")
end)

if not ok_principal then
    warn("[Red Onyx] ERRO: " .. tostring(erro_principal))
    mostrar_erro(erro_principal)
end

print("[Red Onyx] v4 carregado!")
