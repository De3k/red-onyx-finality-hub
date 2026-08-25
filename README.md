-- Red Onyx Hub v13 - ULTRA SIMPLIFICADO e FUNCIONAL
print("[Red Onyx] Iniciando...")

-- ===== COMPAT MÍNIMA =====
local wait = task and task.wait or wait
local spawn = task and task.spawn or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== TENTA PEGAR O GETGENV (seguro) =====
local GENV = _G
pcall(function()
    local ok, env = pcall(getgenv)
    if ok then GENV = env end
end)

-- ===== VARIÁVEIS GLOBAIS =====
GENV.script_key = GENV.script_key or ""

-- ===== CONSTANTES =====
local SCRIPT_ID = "0ae9fe4cf963e3a13d25eed0e2ce5940"
local URL_FREE = "https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua"
local URL_PREMIUM = "https://api.luarmor.net/files/v4/loaders/" .. SCRIPT_ID .. ".lua"
local LINK_KEY = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"
local ARQUIVO_KEY = "RedOnyxKey.txt"

-- ===== CORES =====
local VERM = Color3.fromRGB(255, 45, 60)
local VERM_CLARO = Color3.fromRGB(255, 110, 120)
local VERM_ESC = Color3.fromRGB(140, 18, 32)
local ESCURO = Color3.fromRGB(12, 9, 11)
local CINZA_ESC = Color3.fromRGB(30, 22, 25)
local BRANCO = Color3.fromRGB(245, 240, 242)
local CINZA = Color3.fromRGB(172, 150, 156)

-- ===== FUNÇÕES DE ARQUIVO (100% seguras) =====
local function salvar_chave_disco(chave)
    local ok = pcall(function()
        if makefolder then pcall(makefolder, "RedOnyx") end
        writefile(ARQUIVO_KEY, chave)
    end)
    if not ok then
        pcall(function()
            if syn and syn.writefile then
                if syn.makefolder then pcall(syn.makefolder, "RedOnyx") end
                syn.writefile(ARQUIVO_KEY, chave)
            end
        end)
    end
end

local function carregar_chave_disco()
    local chave = ""
    pcall(function() chave = readfile(ARQUIVO_KEY) end)
    if chave == "" then
        pcall(function()
            if syn and syn.readfile then chave = syn.readfile(ARQUIVO_KEY) end
        end)
    end
    return chave
end

-- ===== DOWNLOAD SIMPLES =====
local function baixar(url)
    local dados = ""
    local ok = pcall(function() dados = game:HttpGet(url) end)

    if not ok or dados == "" then
        local req = (type(syn) == "table" and syn.request) or request or http_request
        if req then
            local ok2, resp = pcall(req, {Url = url, Method = "GET"})
            if ok2 and resp and resp.Body and #resp.Body > 0 then
                dados = resp.Body
            end
        end
    end

    return dados
end

-- ===== ATIVAR TUDO (Settings que o Quantum Onyx lê) =====
local function ativar_tudo()
    GENV.Settings = GENV.Settings or {}
    local S = GENV.Settings

    -- Farm básico
    S["Auto Farm Level"] = true
    S["Auto Quest"] = true
    S["Auto Bones"] = true
    S["Auto Elite Hunter"] = true
    S["Auto Factory Raid"] = true
    S["Auto Pirate Raid"] = true
    S["Auto Redeem Codes"] = true

    -- Armas lendárias
    S["Auto Yama"] = true
    S["Auto TTK"] = true
    S["Auto Tushita"] = true
    S["Auto Ghoul"] = true
    S["Auto CDK"] = true
    S["Auto Rengoku"] = true
    S["Auto Soul Guitar"] = true
    S["Auto Shark Anchor"] = true
    S["Auto Rainbow Haki"] = true

    -- Mastery e coleta
    S["Auto Swords"] = true
    S["Auto Guns"] = true
    S["Auto Fighting Style"] = true
    S["Auto Mastery"] = true
    S["Auto Fragment"] = true
    S["Auto Ectoplasm"] = true
    S["Auto Candy"] = true
    S["Auto Chest"] = true
    S["Auto Sea Beast"] = true
    S["Auto Observation"] = true
    S["Auto Buso Haki"] = true

    -- Config
    S["Farm Distance"] = 2500
    S["Team"] = "Pirates"

    print("[Red Onyx] Auto Pega Tudo ATIVADO!")
end

-- ===== EXECUTAR HUB =====
local function executar(premium, chave)
    ativar_tudo()

    if premium and chave and chave ~= "" then
        GENV.script_key = chave
        salvar_chave_disco(chave)

        print("[Red Onyx] Tentando Premium...")
        local codigo = baixar(URL_PREMIUM)
        if codigo ~= "" then
            local fn, err = loadstring(codigo)
            if fn then
                local ok, erro = pcall(fn)
                if ok then
                    print("[Red Onyx] Premium carregado!")
                    return true
                end
                warn("[Red Onyx] Premium erro: " .. tostring(erro))
            end
        end
        print("[Red Onyx] Premium falhou, tentando Free...")
    end

    print("[Red Onyx] Carregando Free...")
    local codigo = baixar(URL_FREE)
    if codigo == "" then
        warn("[Red Onyx] Download falhou (sem internet ou URL bloqueada)")
        return false, "Download falhou"
    end

    local fn, err = loadstring(codigo)
    if not fn then
        warn("[Red Onyx] Compilacao falhou: " .. tostring(err))
        return false, "Compilacao falhou"
    end

    local ok, erro = pcall(fn)
    if not ok then
        warn("[Red Onyx] Execucao falhou: " .. tostring(erro))
        return false, tostring(erro)
    end

    print("[Red Onyx] Hub carregado com sucesso!")
    return true, "OK"
end

-- ===== RESGATAR CÓDIGOS =====
local function resgatar_codigos()
    local Storage = game:GetService("ReplicatedStorage")
    local Jogador = game:GetService("Players").LocalPlayer
    if not Jogador then return end

    local remote = Storage:FindFirstChild("Redeem") or Storage:FindFirstChild("RedeemCode") or Storage:FindFirstChild("CodeRedemption")

    if not remote then
        warn("[Red Onyx] Remote de codigos nao encontrado")
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
        "TY_FOR_WATCHING", "GAMER_ROBOT_1M", "UPD22",
    }

    local resgatados = 0
    for i, cod in ipairs(codigos) do
        local ok = pcall(function()
            if remote:IsA("RemoteFunction") then
                remote:InvokeServer(cod)
            else
                remote:FireServer(cod)
            end
        end)
        if ok then resgatados = resgatados + 1 end
        if i % 5 == 0 then wait(0.5) end
        wait(0.1)
    end
    print("[Red Onyx] Codigos resgatados: " .. resgatados .. "/" .. #codigos)
end

-- ===== UI SIMPLES E FUNCIONAL =====
local function criar_ui()
    -- Verifica key salva primeiro
    local chave_salva = carregar_chave_disco()
    if chave_salva ~= "" then
        print("[Red Onyx] Key salva encontrada!")
        spawn(function()
            local ok = executar(true, chave_salva)
            if not ok then
                warn("[Red Onyx] Key salva invalida, mostrando UI...")
                pcall(function() delfile(ARQUIVO_KEY) end)
                criar_ui_real()
            end
        end)
        return
    end

    criar_ui_real()
end

local function criar_ui_real()
    local sg = Instance.new("ScreenGui")
    sg.Name = "RedOnyx_UI"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false
    sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- Tenta parent
    local parent_ok = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not parent_ok then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg.Parent = hui end
            end
        end)
    end

    -- Card
    local card = Instance.new("Frame")
    card.Name = "Card"
    card.Size = UDim2.new(0, 380, 0, 240)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = Color3.fromRGB(22, 16, 18)
    card.BorderSizePixel = 0
    card.Parent = sg

    local uc = Instance.new("UICorner")
    uc.CornerRadius = UDim.new(0, 10)
    uc.Parent = card

    local us = Instance.new("UIStroke")
    us.Color = VERM
    us.Thickness = 2
    us.Parent = card

    -- Título
    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 36)
    titulo.Position = UDim2.new(0, 0, 0, 12)
    titulo.BackgroundTransparency = 1
    titulo.Font = Enum.Font.GothamBlack
    titulo.Text = "RED ONYX HUB"
    titulo.TextColor3 = VERM_CLARO
    titulo.TextSize = 24
    titulo.TextXAlignment = Enum.TextXAlignment.Center
    titulo.Parent = card

    -- Subtítulo
    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, 0, 0, 14)
    sub.Position = UDim2.new(0, 0, 0, 50)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.Gotham
    sub.Text = "Auto Pega Tudo | Free + Premium"
    sub.TextColor3 = CINZA
    sub.TextSize = 11
    sub.TextXAlignment = Enum.TextXAlignment.Center
    sub.Parent = card

    -- Input
    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 330, 0, 34)
    input.Position = UDim2.new(0.5, 0, 0, 78)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = ESCURO
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Insira sua Key aqui..."
    input.PlaceholderColor3 = CINZA
    input.Text = ""
    input.TextColor3 = BRANCO
    input.TextSize = 13
    input.ClearTextOnFocus = false
    input.Parent = card

    local iuc = Instance.new("UICorner")
    iuc.CornerRadius = UDim.new(0, 6)
    iuc.Parent = input

    -- Status
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 340, 0, 16)
    status.Position = UDim2.new(0.5, 0, 0, 118)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = VERM_CLARO
    status.TextSize = 11
    status.TextWrapped = true
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.Parent = card

    local function set_status(txt, cor)
        status.Text = txt
        if cor then status.TextColor3 = cor end
    end

    local ocupado = false

    -- Validar key
    local function validar_key()
        if ocupado then return end
        local chave = input.Text
        if chave == "" then
            set_status("Digite uma key primeiro!", VERM)
            return
        end
        ocupado = true
        set_status("Validando key...", VERM_CLARO)

        spawn(function()
            local sdk = baixar("https://sdkapi-public.luarmor.net/library.lua")
            if sdk == "" then
                ocupado = false
                set_status("Erro SDK - sem internet?", VERM)
                return
            end

            local fn_sdk, err_sdk = loadstring(sdk)
            if not fn_sdk then
                ocupado = false
                set_status("Erro SDK", VERM)
                return
            end

            local ok_api, api = pcall(fn_sdk)
            if not ok_api or type(api) ~= "table" then
                ocupado = false
                set_status("Erro SDK", VERM)
                return
            end

            api.script_id = SCRIPT_ID
            local ok_check, res = pcall(function() return api.check_key(chave) end)

            if ok_check and type(res) == "table" and res.code == "KEY_VALID" then
                set_status("Key valida! Carregando...", VERM_CLARO)
                wait(0.3)

                local ok_hub, erro_hub = executar(true, chave)
                if ok_hub then
                    sg:Destroy()
                else
                    ocupado = false
                    set_status("Erro: " .. tostring(erro_hub), VERM)
                end
            else
                ocupado = false
                local msg = "Key invalida!"
                if type(res) == "table" and res.message then
                    msg = tostring(res.message)
                end
                set_status(msg, VERM)
            end
        end)
    end

    -- Carregar free
    local function carregar_free()
        if ocupado then return end
        ocupado = true
        set_status("Carregando versao Free...", VERM_CLARO)

        spawn(function()
            local ok, erro = executar(false, nil)
            if ok then
                sg:Destroy()
            else
                ocupado = false
                set_status("Erro: " .. tostring(erro), VERM)
            end
        end)
    end

    -- Função pra criar botão
    local function criar_btn(posX, cor, txt, cb)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 108, 0, 32)
        btn.Position = UDim2.new(0, posX, 0, 146)
        btn.BackgroundColor3 = cor
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = txt
        btn.TextColor3 = BRANCO
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.Parent = card

        local bc = Instance.new("UICorner")
        bc.CornerRadius = UDim.new(0, 6)
        bc.Parent = btn

        btn.MouseButton1Click:Connect(cb)
        return btn
    end

    criar_btn(22,  VERM_ESC, "Free", carregar_free)
    criar_btn(138, VERM_ESC, "Pegar Key", function()
        pcall(function() setclipboard(LINK_KEY) end)
        set_status("Link copiado!", VERM_CLARO)
    end)
    criar_btn(252, VERM, "Usar Key", validar_key)

    spawn(function()
        wait(0.5)
        input:CaptureFocus()
    end)
end

-- ===== INICIAR TUDO =====
local ok, erro = pcall(criar_ui)
if not ok then
    warn("[Red Onyx] ERRO CRITICO: " .. tostring(erro))

    -- Tenta carregar direto sem UI
    local chave = carregar_chave_disco()
    if chave ~= "" then
        print("[Red Onyx] Tentando carregar direto com key salva...")
        pcall(function() executar(true, chave) end)
    else
        print("[Red Onyx] Tentando carregar versao Free direto...")
        pcall(function() executar(false, nil) end)
    end
end

print("[Red Onyx] Script finalizado - aguardando...")
