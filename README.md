-- Red Onyx Hub v12 - COMPLETO em PT-BR
print("[Red Onyx Hub] v12 iniciando...")

-- ===== CONFIG =====
local LOGO_RBX = "rbxassetid://240328436758150"
local KEY_FILE = "RedOnyxHub_Key.txt"

-- ===== COMPAT =====
local aguardar = task and task.wait or wait
local spawn = task and task.spawn or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== SEGURANÇA =====
local function pegar_env()
    local ok, env = pcall(getgenv)
    return ok and env or _G
end

local function definir_chave_env(chave)
    local env = pegar_env()
    env.script_key = chave
    pcall(function() _G.script_key = chave end)
end

-- ===== PERSISTÊNCIA DA KEY =====
local function salvar_chave(arquivo, chave)
    local ok = pcall(function()
        if makefolder then pcall(makefolder, "RedOnyxHub") end
        writefile(arquivo, chave)
    end)
    if not ok then
        ok = pcall(function()
            if syn and syn.writefile then
                if syn.makefolder then pcall(syn.makefolder, "RedOnyxHub") end
                syn.writefile(arquivo, chave)
            end
        end)
    end
    return ok
end

local function carregar_chave(arquivo)
    local chave = ""
    pcall(function() chave = readfile(arquivo) end)
    if chave == "" then
        pcall(function()
            if syn and syn.readfile then chave = syn.readfile(arquivo) end
        end)
    end
    return chave
end

local function deletar_chave(arquivo)
    pcall(function() delfile(arquivo) end)
    pcall(function() if syn and syn.delfile then syn.delfile(arquivo) end end)
end

-- ===== CONSTANTES =====
local Jogadores = game:GetService("Players")
local SCRIPT_ID = "0ae9fe4cf963e3a13d25eed0e2ce5940"
local URL_FREE = "https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua"
local URL_PREMIUM = "https://api.luarmor.net/files/v4/loaders/" .. SCRIPT_ID .. ".lua"
local LINK_KEY = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"

-- ===== CORES =====
local VERM   = Color3.fromRGB(255, 45, 60)
local VERM_T = Color3.fromRGB(255, 110, 120)
local VERM_E = Color3.fromRGB(140, 18, 32)
local ESC    = Color3.fromRGB(12, 9, 11)
local ESC_M  = Color3.fromRGB(22, 16, 18)
local CIN_E  = Color3.fromRGB(30, 22, 25)
local CIN_M  = Color3.fromRGB(42, 30, 35)
local BRANCO = Color3.fromRGB(245, 240, 242)
local CINZA  = Color3.fromRGB(172, 150, 156)

-- ===== HTTP COM TIMEOUT =====
local function baixar(url, timeout)
    timeout = timeout or 15
    local resultado = nil
    local pronto = false
    local inicio = tick()

    spawn(function()
        local ok, dados = pcall(function() return game:HttpGet(url) end)
        if ok and type(dados) == "string" and #dados > 0 then
            resultado = dados
        end
        pronto = true
    end)

    while not pronto and tick() - inicio < timeout do
        aguardar(0.05)
    end

    if resultado then return resultado end

    -- Fallback: syn.request / request
    local req = (type(syn) == "table" and syn.request) or request or http_request
    if req then
        local ok, resp = pcall(req, { Url = url, Method = "GET" })
        if ok and resp and resp.StatusCode == 200 and type(resp.Body) == "string" and #resp.Body > 0 then
            return resp.Body
        end
    end

    return ""
end

-- ===== CÓDIGOS ATUALIZADOS (Agosto 2026) =====
local function obter_codigos()
    return {
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
end

-- ===== RESGATAR CÓDIGOS =====
local function resgatar_codigos()
    local Storage = game:GetService("ReplicatedStorage")
    local Jogador = Jogadores.LocalPlayer
    if not Jogador then return 0 end

    -- Procura o Remote de resgate
    local remote = nil
    local nomes = {"Redeem", "RedeemCode", "CodeRedemption", "ClaimCode", "CheckCode"}

    local function procurar(pasta, nomes)
        if not pasta then return nil end
        for _, nome in ipairs(nomes) do
            local obj = pasta:FindFirstChild(nome)
            if obj then return obj end
        end
        return nil
    end

    remote = procurar(Storage, nomes)
    if not remote then remote = procurar(Jogador, nomes) end

    if not remote then
        warn("[Red Onyx] Remote de resgate nao encontrado")
        return 0
    end

    local codigos = obter_codigos()
    local resgatados = 0

    for i, codigo in ipairs(codigos) do
        local ok = pcall(function()
            if remote:IsA("RemoteFunction") then
                remote:InvokeServer(codigo)
            elseif remote:IsA("RemoteEvent") then
                remote:FireServer(codigo)
            end
        end)
        if ok then resgatados = resgatados + 1 end
        if i % 5 == 0 then aguardar(0.5) end
        aguardar(0.1)
    end

    print("[Red Onyx] Resgatados " .. resgatados .. "/" .. #codigos .. " codigos")
    return resgatados
end

-- ===== ATIVAR AUTO PEGA TUDO =====
local function ativar_auto_tudo()
    local env = pegar_env()
    env.Settings = env.Settings or {}
    local S = env.Settings

    -- Armas e estilos
    S["Auto Yama"]        = true
    S["Auto TTK"]         = true
    S["Auto Tushita"]     = true
    S["Auto Ghoul"]       = true
    S["Auto Get Ghoul"]   = true
    S["Auto CDK"]         = true
    S["Auto Rengoku"]     = true
    S["Auto Soul Guitar"] = true
    S["Auto Shark Anchor"]= true
    S["Auto Rainbow Haki"]= true
    S["Auto Swords"]      = true
    S["Auto Guns"]        = true
    S["Auto Fighting Style"] = true

    -- Farm
    S["Auto Farm Level"]  = true
    S["Auto Quest"]       = true
    S["Auto Bones"]       = true
    S["Auto Elite Hunter"]= true
    S["Auto Factory Raid"]= true
    S["Auto Pirate Raid"] = true
    S["Auto Redeem Codes"]= true

    -- Config
    S["Farm Distance"]    = S["Farm Distance"] or 2500
    S["Team"]             = S["Team"] or "Pirates"

    -- Pega TUDO (inclui frutas, maestria, etc)
    S["Auto All"]         = true
    S["Auto All Swords"]  = true
    S["Auto All Guns"]    = true
    S["Auto All Fruits"]  = true
    S["Auto All Fighting"]= true
    S["Auto All Accessories"] = true
    S["Auto All Items"]   = true
    S["Auto Mastery"]     = true
    S["Auto Bone"]        = true
    S["Auto Fragment"]    = true
    S["Auto Ectoplasm"]   = true
    S["Auto Candy"]       = true
    S["Auto Chest"]       = true
    S["Auto Sea Beast"]   = true
    S["Auto Ship"]        = true
    S["Auto Dungeon"]     = true
    S["Auto Race V2"]     = true
    S["Auto Observation"] = true
    S["Auto Buso Haki"]   = true
    S["Auto Soru"]        = true
    S["Auto Geppo"]       = true
    S["Auto Flask"]       = true
    S["Auto Leviathan"]   = true
    S["Auto Cursed"]["Cursed Captain"] = true

    print("[Red Onyx] Auto Pega Tudo ativado!")
end

-- ===== CARREGADOR =====
local function carregar_hub(url, callback_status)
    if callback_status then callback_status("Baixando hub...") end
    local codigo = baixar(url)
    if codigo == "" then return false, "Falha no download (sem internet?)" end

    print("[Red Onyx] Baixados " .. #codigo .. " bytes")
    if callback_status then callback_status("Compilando...") end

    local fn = (loadstring or load)(codigo)
    if type(fn) ~= "function" then return false, "Falha na compilacao" end

    if callback_status then callback_status("Iniciando hub...") end
    local ok, erro = pcall(fn)
    if not ok then return false, tostring(erro) end
    return true, "OK"
end

-- ===== EXECUTAR HUB =====
local function executar_hub(premium, chave, ui_para_fechar)
    ativar_auto_tudo()

    if premium and chave and chave ~= "" then
        definir_chave_env(chave)
        salvar_chave(KEY_FILE, chave)

        local ok, erro = carregar_hub(URL_PREMIUM)
        if ok then
            print("[Red Onyx] Premium carregado!")
            if ui_para_fechar then aguardar(0.5); pcall(function() ui_para_fechar:Destroy() end) end
            return true
        end
        warn("[Red Onyx] Premium falhou: " .. tostring(erro) .. " - tentando Free...")
    end

    local ok, erro = carregar_hub(URL_FREE)
    if ok then
        print("[Red Onyx] Hub carregado com sucesso!")
        if ui_para_fechar then aguardar(0.5); pcall(function() ui_para_fechar:Destroy() end) end
        return true
    end

    warn("[Red Onyx] Erro final: " .. tostring(erro))
    return false, erro
end

-- ===== CRIAR UI DE KEY =====
local function criar_ui_key()
    local sg = Instance.new("ScreenGui")
    sg.Name = "RedOnyxHub_UI"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false

    local parent_ok = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not parent_ok then
        local ok, hui = pcall(gethui)
        if ok and typeof(hui) == "Instance" then sg.Parent = hui end
    end

    -- Verifica se tem key salva
    local chave_salva = carregar_chave(KEY_FILE)
    if chave_salva ~= "" then
        print("[Red Onyx] Key salva encontrada! Validando automaticamente...")
        spawn(function()
            local ok = executar_hub(true, chave_salva, sg)
            if not ok then
                warn("[Red Onyx] Key salva invalida - mostrando UI")
                deletar_chave(KEY_FILE)
                spawn(function() construir_ui(sg, "") end)
            end
        end)
        return
    end

    construir_ui(sg, "")
end

local function construir_ui(sg, chave_inicial)
    -- Fundo escuro
    local fundo = Instance.new("Frame")
    fundo.Size = UDim2.new(1, 0, 1, 0)
    fundo.BackgroundColor3 = Color3.new(0, 0, 0)
    fundo.BackgroundTransparency = 0.5
    fundo.BorderSizePixel = 0
    fundo.ZIndex = 1
    fundo.Parent = sg

    -- Card principal
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 420, 0, 260)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = ESC_M
    card.BorderSizePixel = 0
    card.ZIndex = 2
    card.Parent = sg

    local canto = Instance.new("UICorner")
    canto.CornerRadius = UDim.new(0, 12)
    canto.Parent = card

    local borda = Instance.new("UIStroke")
    borda.Color = VERM
    borda.Thickness = 2
    borda.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    borda.Parent = card

    -- Título
    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 40)
    titulo.Position = UDim2.new(0, 0, 0, 14)
    titulo.BackgroundTransparency = 1
    titulo.Font = Enum.Font.GothamBlack
    titulo.Text = "RED ONYX HUB"
    titulo.TextColor3 = VERM_T
    titulo.TextSize = 26
    titulo.TextXAlignment = Enum.TextXAlignment.Center
    titulo.ZIndex = 3
    titulo.Parent = card

    -- Subtítulo
    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, 0, 0, 16)
    sub.Position = UDim2.new(0, 0, 0, 56)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.Gotham
    sub.Text = "Versao Free + Premium | Auto Pega Tudo"
    sub.TextColor3 = CINZA
    sub.TextSize = 11
    sub.TextXAlignment = Enum.TextXAlignment.Center
    sub.ZIndex = 3
    sub.Parent = card

    -- Input da Key
    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 360, 0, 36)
    input.Position = UDim2.new(0.5, 0, 0, 86)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = ESC
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Insira sua key aqui..."
    input.PlaceholderColor3 = CINZA
    input.TextColor3 = BRANCO
    input.TextSize = 13
    input.ClearTextOnFocus = false
    input.ZIndex = 3
    input.Parent = card
    if chave_inicial and chave_inicial ~= "" then input.Text = chave_inicial end

    local canto_input = Instance.new("UICorner")
    canto_input.CornerRadius = UDim.new(0, 6)
    canto_input.Parent = input

    local borda_input = Instance.new("UIStroke")
    borda_input.Color = VERM
    borda_input.Transparency = 0.5
    borda_input.Parent = input

    -- Status
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 380, 0, 18)
    status.Position = UDim2.new(0.5, 0, 0, 128)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = VERM_T
    status.TextSize = 11
    status.TextWrapped = true
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.ZIndex = 3
    status.Parent = card

    local function set_status(txt, cor)
        status.Text = txt
        if cor then status.TextColor3 = cor end
    end

    local ocupado = false

    -- Validar Key Premium
    local function validar_key(chave)
        if ocupado then return end
        if not chave or chave == "" then
            set_status("Digite uma key primeiro!", VERM_T)
            return
        end
        ocupado = true
        set_status("Validando key...", VERM_T)

        spawn(function()
            local codigo = baixar("https://sdkapi-public.luarmor.net/library.lua")
            if codigo == "" then
                ocupado = false
                set_status("Erro SDK - sem internet?", VERM_T)
                return
            end

            local fn = (loadstring or load)(codigo)
            if type(fn) ~= "function" then
                ocupado = false
                set_status("Erro SDK", VERM_T)
                return
            end

            local ok_api, api = pcall(fn)
            if not ok_api or type(api) ~= "table" then
                ocupado = false
                set_status("Erro SDK - tente novamente", VERM_T)
                return
            end

            api.script_id = SCRIPT_ID
            local ok_check, resultado = pcall(function() return api.check_key(chave) end)

            if ok_check and type(resultado) == "table" and resultado.code == "KEY_VALID" then
                set_status("Key valida! Carregando...", VERM_T)
                aguardar(0.3)

                local ok_hub = executar_hub(true, chave, sg)
                if not ok_hub then
                    ocupado = false
                    set_status("Erro ao carregar hub", VERM_T)
                end
            else
                ocupado = false
                local msg = "Key invalida!"
                if type(resultado) == "table" then msg = tostring(resultado.message or resultado.code) end
                set_status(msg, VERM_T)
                deletar_chave(KEY_FILE)
            end
        end)
    end

    -- Carregar Free
    local function carregar_free()
        if ocupado then return end
        ocupado = true
        set_status("Carregando versao Free...", VERM_T)

        spawn(function()
            local ok, erro = executar_hub(false, nil, sg)
            if not ok then
                ocupado = false
                set_status("Erro: " .. tostring(erro), VERM_T)
            end
        end)
    end

    -- Função para criar botões
    local function criar_botao(pos_x, cor, texto, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 115, 0, 34)
        btn.Position = UDim2.new(0, pos_x, 0, 158)
        btn.BackgroundColor3 = cor
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = texto
        btn.TextColor3 = BRANCO
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.ZIndex = 3
        btn.Parent = card

        local canto_btn = Instance.new("UICorner")
        canto_btn.CornerRadius = UDim.new(0, 6)
        canto_btn.Parent = btn

        local borda_btn = Instance.new("UIStroke")
        borda_btn.Color = VERM
        borda_btn.Transparency = 0.55
        borda_btn.Parent = btn

        btn.MouseButton1Click:Connect(function()
            local ok, err = pcall(callback)
            if not ok then warn("[Red Onyx] Erro no botao: " .. tostring(err)) end
        end)

        -- Efeito hover
        btn.MouseEnter:Connect(function() btn.BackgroundColor3 = cor:Lerp(BRANCO, 0.2) end)
        btn.MouseLeave:Connect(function() btn.BackgroundColor3 = cor end)

        return btn
    end

    -- Botões
    criar_botao(32,  VERM_E, "Versao Free", carregar_free)
    criar_botao(152, VERM_E, "Pegar Key", function()
        pcall(function() (setclipboard or toclipboard)(LINK_KEY) end)
        set_status("Link copiado para area de transferencia!", VERM_T)
    end)
    criar_botao(272, VERM,   "Usar Key", function() validar_key(input.Text) end)

    -- Nota informativa no rodapé
    local nota = Instance.new("TextLabel")
    nota.Size = UDim2.new(1, -20, 0, 20)
    nota.Position = UDim2.new(0.5, 0, 1, -24)
    nota.AnchorPoint = Vector2.new(0.5, 0)
    nota.BackgroundTransparency = 1
    nota.Font = Enum.Font.Gotham
    nota.Text = "Key salva automaticamente - so digita uma vez!"
    nota.TextColor3 = CINZA
    nota.TextSize = 10
    nota.TextXAlignment = Enum.TextXAlignment.Center
    nota.ZIndex = 3
    nota.Parent = card
end

-- ===== PATCHER DE TEMA SIMPLIFICADO =====
-- Apenas troca logos, sem interferir no funcionamento do hub
spawn(function()
    aguardar(2)

    local function trocar_logo(obj)
        if LOGO_RBX == "" then return false end
        if not (obj:IsA("ImageLabel") or obj:IsA("ImageButton")) then return false end
        if obj.Image == LOGO_RBX then return true end
        local img = tostring(obj.Image)
        if not img:find("rbxassetid://") then return false end
        pcall(function() obj.Image = LOGO_RBX end)
        return true
    end

    local function renomear_texto(obj)
        local t = obj.Text
        if type(t) ~= "string" or t == "" then return end
        if t:lower():find("quantum") or t:lower():find("onyx") or t:lower():find("kaitun") then
            obj.Text = "Red Onyx Hub"
        end
    end

    local function varrer_gui(gui)
        for _, obj in ipairs(gui:GetDescendants()) do
            pcall(function()
                if obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                    trocar_logo(obj)
                end
                if (obj:IsA("TextLabel") or obj:IsA("TextButton")) and not obj:IsA("TextBox") then
                    renomear_texto(obj)
                end
            end)
        end
    end

    local CoreGui = game:GetService("CoreGui")

    for i = 1, 30 do -- 30 tentativas (15 segundos)
        for _, gui in ipairs(CoreGui:GetChildren()) do
            if gui:IsA("ScreenGui") then
                pcall(function() varrer_gui(gui) end)
            end
        end
        aguardar(0.5)
    end
    print("[Red Onyx] Patcher finalizado")
end)

-- ===== INICIAR =====
local ok_inicio, erro_inicio = pcall(criar_ui_key)
if not ok_inicio then
    warn("[Red Onyx] Erro: " .. tostring(erro_inicio))

    -- UI de erro
    local eg = Instance.new("ScreenGui")
    eg.Name = "RedOnyxHub_Erro"
    eg.IgnoreGuiInset = true
    local parent_ok = pcall(function() eg.Parent = game:GetService("CoreGui") end)
    if not parent_ok then
        local ok, hui = pcall(gethui)
        if ok and typeof(hui) == "Instance" then eg.Parent = hui end
    end

    local el = Instance.new("TextLabel")
    el.Size = UDim2.new(1, -60, 0, 100)
    el.Position = UDim2.new(0.5, 0, 0.85, 0)
    el.AnchorPoint = Vector2.new(0.5, 0)
    el.BackgroundColor3 = ESC
    el.BackgroundTransparency = 0.15
    el.BorderSizePixel = 0
    el.Font = Enum.Font.Gotham
    el.Text = "RED ONYX HUB - ERRO:\n" .. tostring(erro_inicio) .. "\n\nTente executar novamente ou use outro executor."
    el.TextColor3 = VERM_T
    el.TextSize = 13
    el.TextWrapped = true
    el.TextXAlignment = Enum.TextXAlignment.Center
    el.Parent = eg

    local canto_el = Instance.new("UICorner")
    canto_el.CornerRadius = UDim.new(0, 8)
    canto_el.Parent = el

    local borda_el = Instance.new("UIStroke")
    borda_el.Color = VERM
    borda_el.Thickness = 1.5
    borda_el.Parent = el
end

print("[Red Onyx Hub] v12 carregado com sucesso!")
