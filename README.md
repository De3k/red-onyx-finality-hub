-- ============================================================
-- RED ONYX HUB v16 - COMPLETO + ANTI-TRAVAMENTO
-- Patcher vermelho/preto funcional | Auto Pega Tudo | Key salva
-- ============================================================
print("[Red Onyx Hub] v16 iniciando...")

-- ===== COMPAT =====
local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end
local GENV = _G
pcall(function() local ok, e = pcall(getgenv); if ok then GENV = e end end)

-- ===== PALETA (vermelho e preto) =====
local VERM   = Color3.fromRGB(200, 30, 45)
local VERM_C = Color3.fromRGB(240, 80, 90)
local VERM_E = Color3.fromRGB(130, 15, 28)
local ESC    = Color3.fromRGB(15, 12, 14)
local ESC_M  = Color3.fromRGB(22, 16, 18)
local CIN_M  = Color3.fromRGB(30, 22, 25)
local BRA    = Color3.fromRGB(240, 235, 237)
local CIN    = Color3.fromRGB(150, 140, 142)

-- ===== CONSTANTES =====
local SCRIPT_ID = "0ae9fe4cf963e3a13d25eed0e2ce5940"
local URL_FREE = "https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua"
local URL_PREMIUM = "https://api.luarmor.net/files/v4/loaders/" .. SCRIPT_ID .. ".lua"
local LINK_KEY = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"
local ARQ_KEY = "RedOnyxKey.txt"
local TAG = "ROXThemed" -- atributo que marca objetos já tematizados (ANTI-LOOP)

local Core = game:GetService("CoreGui")
local Jogadores = game:GetService("Players")
local Jogador = Jogadores.LocalPlayer

-- ============================================================
-- PATCHER VERMELHO/PRETO — FUNCIONAL E SEM TRAVAR
-- Estratégia: marca cada objeto com atributo. Objeto marcado
-- NUNCA é processado de novo. Isso elimina o loop de feedback
-- com o sistema de tema do hub (causa real do freeze).
-- ============================================================
local patcher_habilitado = true

local function aplicar_tema(obj)
    if not patcher_habilitado then return end
    -- ANTI-LOOP: se já foi tematizado, ignora. Sem isso = repaint infinito = freeze
    local ok, ja_feito = pcall(function() return obj:GetAttribute(TAG) end)
    if ok and ja_feito then return end

    pcall(function()
        obj:SetAttribute(TAG, true)

        if obj:IsA("GuiObject") then
            local bg = obj.BackgroundColor3
            local lum = bg.R * 0.299 + bg.G * 0.587 + bg.B * 0.114
            if lum < 0.15 then obj.BackgroundColor3 = ESC
            elseif lum < 0.25 then obj.BackgroundColor3 = ESC_M
            elseif lum < 0.35 then obj.BackgroundColor3 = CIN_M
            else obj.BackgroundColor3 = Color3.fromRGB(42, 30, 35) end
            obj.BorderColor3 = VERM_E
        end
        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
            obj.TextColor3 = BRA
            local t = obj.Text
            if type(t) == "string" and t ~= "" then
                local tl = t:lower()
                if tl:find("quantum") or tl:find("kaitun") or tl:find("k https") then
                    obj.Text = t:gsub("[Qq]uantum%s*[Oo]nyx", "Red Onyx Hub"):gsub("[Kk]aitun", "Red Onyx Hub")
                end
            end
        end
        if obj:IsA("TextBox") then
            obj.TextColor3 = BRA
            obj.PlaceholderColor3 = CIN
        end
        if obj:IsA("UIStroke") then
            obj.Color = VERM
        end
        if obj:IsA("ScrollingFrame") then
            obj.ScrollBarImageColor3 = VERM_C
        end
        if obj:IsA("UIGradient") then
            obj.Color = ColorSequence.new(ESC_M, VERM_E)
        end
        -- Imagens: NÃO mexemos em ImageColor3 para não escurecer logos
    end)
end

-- GUIs que o patcher DEVE ignorar (nossa UI e GUIs do jogo)
local gui_ignorada = {}

local function nome_deve_patchear(gui)
    local n = gui.Name:lower()
    if gui_ignorada[gui] then return false end
    -- Só tematiza GUIs que parecem ser do hub
    return n:find("hub") or n:find("quantum") or n:find("onyx") or n:find("kaitun")
        or n:find("ui", 1, true) or n:find("lib", 1, true) or n:find("red")
end

-- Detecção de novas GUIs por evento (leve) + pass único com debounce
local function iniciar_patcher()
    spawn(function()
        aguardar(2)
        -- Pass inicial: tematiza GUIs do hub que já existem
        pcall(function()
            for _, gui in ipairs(Core:GetChildren()) do
                if gui:IsA("ScreenGui") and nome_deve_patchear(gui) then
                    for _, obj in ipairs(gui:GetDescendants()) do
                        aplicar_tema(obj)
                    end
                end
            end
        end)

        -- Evento: nova GUI/objeto aparece → tematiza (com debounce leve)
        pcall(function()
            Core.ChildAdded:Connect(function(child)
                aguardar(1) -- espera a GUI terminar de construir
                if child:IsA("ScreenGui") and nome_deve_patchear(child) then
                    for _, obj in ipairs(child:GetDescendants()) do
                        aplicar_tema(obj)
                    end
                    -- Novos objetos adicionados DENTRO da GUI do hub
                    child.DescendantAdded:Connect(function(novo)
                        aguardar(0.2)
                        aplicar_tema(novo)
                    end)
                end
            end)
        end)

        -- Rede de segurança: pass completo a cada 30s (só objetos NÃO marcados)
        while true do
            aguardar(30)
            pcall(function()
                for _, gui in ipairs(Core:GetChildren()) do
                    if gui:IsA("ScreenGui") and gui.Parent and nome_deve_patchear(gui) then
                        for _, obj in ipairs(gui:GetDescendants()) do
                            aplicar_tema(obj) -- objetos marcados são ignorados instantaneamente
                        end
                    end
                end
            end)
        end
    end)
end

-- ============================================================
-- KEY: PERSISTÊNCIA EM DISCO
-- ============================================================
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

-- ============================================================
-- DOWNLOAD (com fallback)
-- ============================================================
local function baixar(url)
    local dados = ""
    pcall(function() dados = game:HttpGet(url) end)
    if type(dados) ~= "string" or dados == "" then
        local req = (type(syn) == "table" and syn.request) or request or http_request
        if req then
            pcall(function()
                local resp = req({ Url = url, Method = "GET" })
                if resp and resp.Body and #resp.Body > 0 then dados = resp.Body end
            end)
        end
    end
    return dados
end

-- ============================================================
-- AUTO PEGA TUDO: ATIVAR TODAS AS CONFIGS
-- (o QuantumOnyx lê essas flags ao iniciar)
-- ============================================================
GENV.Settings = GENV.Settings or {}
local S = GENV.Settings

local function ativar_tudo()
    S["Auto Farm Level"]     = true
    S["Auto Quest"]          = true
    S["Auto Bones"]          = true
    S["Auto Elite Hunter"]   = true
    S["Auto Factory Raid"]   = true
    S["Auto Pirate Raid"]    = true
    S["Auto Redeem Codes"]   = true
    S["Auto Yama"]           = true
    S["Auto TTK"]            = true
    S["Auto Tushita"]        = true
    S["Auto Ghoul"]          = true
    S["Auto Get Ghoul"]      = true
    S["Auto CDK"]            = true
    S["Auto Rengoku"]        = true
    S["Auto Soul Guitar"]    = true
    S["Auto Shark Anchor"]   = true
    S["Auto Rainbow Haki"]   = true
    S["Auto Swords"]         = true
    S["Auto Guns"]           = true
    S["Auto Fighting Style"] = true
    S["Auto Mastery"]        = true
    S["Auto Fragment"]       = true
    S["Auto Ectoplasm"]      = true
    S["Auto Candy"]          = true
    S["Auto Chest"]          = true
    S["Auto Sea Beast"]      = true
    S["Auto Observation"]    = true
    S["Auto Buso Haki"]      = true
    S["Farm Distance"]       = 2500
    S["Team"]                = "Pirates"
    print("[Red Onyx Hub] Auto Pega Tudo ativado!")
end

-- ============================================================
-- CARREGAR HUB — SEM MODIFICAR O CÓDIGO (gsub quebrava tudo)
-- ============================================================
local function carregar_hub(premium, chave)
    ativar_tudo()

    if premium and chave and chave ~= "" then
        GENV.script_key = chave
        salvar_key(chave)

        local codigo = baixar(URL_PREMIUM)
        if codigo ~= "" then
            local fn = loadstring(codigo)
            if fn then
                local ok, err = pcall(fn)
                if ok then
                    print("[Red Onyx Hub] Premium carregado!")
                    return true, "OK"
                end
                warn("[Red Onyx] Premium erro: " .. tostring(err))
            end
        end
        print("[Red Onyx Hub] Premium falhou, caindo para Free...")
    end

    local codigo = baixar(URL_FREE)
    if codigo == "" then return false, "Download falhou (sem internet ou executor bloqueou)" end

    print("[Red Onyx Hub] Baixado " .. #codigo .. " bytes")

    local fn = loadstring(codigo)
    if not fn then return false, "Falha na compilacao" end

    local ok, err = pcall(fn)
    if not ok then return false, "Erro na execucao: " .. tostring(err) end

    print("[Red Onyx Hub] Hub carregado!")
    return true, "OK"
end

-- ============================================================
-- VALIDAR KEY (Luarmor SDK)
-- ============================================================
local function validar_key(chave, cb_ok, cb_erro)
    spawn(function()
        local sdk = baixar("https://sdkapi-public.luarmor.net/library.lua")
        if sdk == "" then
            if cb_erro then cb_erro("SDK offline - sem internet") end
            return
        end
        local fn = loadstring(sdk)
        if not fn then
            if cb_erro then cb_erro("SDK invalido") end
            return
        end
        local ok_api, api = pcall(fn)
        if not ok_api or type(api) ~= "table" then
            if cb_erro then cb_erro("Erro no SDK") end
            return
        end
        api.script_id = SCRIPT_ID
        local ok_check, res = pcall(function() return api.check_key(chave) end)
        if ok_check and type(res) == "table" and res.code == "KEY_VALID" then
            if cb_ok then cb_ok() end
        else
            local msg = "Key invalida"
            if type(res) == "table" and res.message then msg = tostring(res.message) end
            if cb_erro then cb_erro(msg) end
        end
    end)
end

-- ============================================================
-- UI DE KEY
-- ============================================================
local sg_key_ui = nil -- referência para o patcher ignorar nossa UI

local function criar_ui_real()
    sg_key_ui = Instance.new("ScreenGui")
    sg_key_ui.Name = "RedOnyxHubKey"
    sg_key_ui.IgnoreGuiInset = true
    sg_key_ui.ResetOnSpawn = false

    local pai_ok = pcall(function() sg_key_ui.Parent = Core end)
    if not pai_ok then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg_key_ui.Parent = hui end
            end
        end)
    end

    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 380, 0, 210)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = ESC_M
    card.BorderSizePixel = 0
    card.Parent = sg_key_ui

    local canto = Instance.new("UICorner")
    canto.CornerRadius = UDim.new(0, 10)
    canto.Parent = card

    local borda = Instance.new("UIStroke")
    borda.Color = VERM
    borda.Thickness = 2
    borda.Parent = card

    local tit = Instance.new("TextLabel")
    tit.Size = UDim2.new(1, 0, 0, 34)
    tit.Position = UDim2.new(0, 0, 0, 8)
    tit.BackgroundTransparency = 1
    tit.Font = Enum.Font.GothamBlack
    tit.Text = "RED ONYX HUB"
    tit.TextColor3 = VERM_C
    tit.TextSize = 22
    tit.TextXAlignment = Enum.TextXAlignment.Center
    tit.Parent = card

    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, 0, 0, 12)
    sub.Position = UDim2.new(0, 0, 0, 44)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.Gotham
    sub.Text = "Auto Pega Tudo | Patcher Vermelho e Preto"
    sub.TextColor3 = CIN
    sub.TextSize = 10
    sub.TextXAlignment = Enum.TextXAlignment.Center
    sub.Parent = card

    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 340, 0, 34)
    input.Position = UDim2.new(0.5, 0, 0, 68)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = ESC
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Key Premium (opcional)"
    input.PlaceholderColor3 = CIN
    input.Text = ""
    input.TextColor3 = BRA
    input.TextSize = 12
    input.ClearTextOnFocus = false
    input.Parent = card

    local ci = Instance.new("UICorner")
    ci.CornerRadius = UDim.new(0, 6)
    ci.Parent = input

    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(1, -20, 0, 16)
    status.Position = UDim2.new(0.5, 0, 0, 108)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = VERM_C
    status.TextSize = 10
    status.TextWrapped = true
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.Parent = card

    local function set_status(txt, cor)
        status.Text = txt
        if cor then status.TextColor3 = cor end
    end

    local ocupado = false

    local function criar_btn(posX, cor, txt, cb)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 170, 0, 34)
        btn.Position = UDim2.new(0, posX, 0, 130)
        btn.BackgroundColor3 = cor
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = txt
        btn.TextColor3 = BRA
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.Parent = card

        local cb2 = Instance.new("UICorner")
        cb2.CornerRadius = UDim.new(0, 6)
        cb2.Parent = btn

        btn.MouseButton1Click:Connect(cb)
        return btn
    end

    criar_btn(15, VERM_E, "INICIAR FREE", function()
        if ocupado then return end
        ocupado = true
        set_status("Carregando versao Free...", VERM_C)
        spawn(function()
            local ok, err = carregar_hub(false, nil)
            if ok then
                sg_key_ui:Destroy()
            else
                ocupado = false
                set_status("Erro: " .. tostring(err), VERM)
            end
        end)
    end)

    criar_btn(195, VERM, "USAR KEY", function()
        if ocupado then return end
        local chave = input.Text
        if chave == "" then
            set_status("Digite a key ou use o Free!", VERM)
            return
        end
        ocupado = true
        set_status("Validando key...", VERM_C)

        validar_key(chave, function()
            set_status("Key valida! Salvando e carregando...", VERM_C)
            aguardar(0.3)
            local ok, err = carregar_hub(true, chave)
            if ok then
                sg_key_ui:Destroy()
            else
                ocupado = false
                set_status("Erro: " .. tostring(err), VERM)
            end
        end, function(msg)
            ocupado = false
            set_status(msg, VERM)
            deletar_key() -- key inválida não fica salva
        end)
    end)

    -- Botão copiar link da key
    local btn_link = Instance.new("TextButton")
    btn_link.Size = UDim2.new(0, 340, 0, 24)
    btn_link.Position = UDim2.new(0.5, 0, 0, 172)
    btn_link.AnchorPoint = Vector2.new(0.5, 0)
    btn_link.BackgroundColor3 = Color3.fromRGB(60, 20, 28)
    btn_link.BorderSizePixel = 0
    btn_link.Font = Enum.Font.Gotham
    btn_link.Text = "Pegar Key (copia o link)"
    btn_link.TextColor3 = CIN
    btn_link.TextSize = 10
    btn_link.AutoButtonColor = false
    btn_link.Parent = card

    local cl = Instance.new("UICorner")
    cl.CornerRadius = UDim.new(0, 5)
    cl.Parent = btn_link

    btn_link.MouseButton1Click:Connect(function()
        pcall(function() (setclipboard or toclipboard)(LINK_KEY) end)
        set_status("Link copiado! Cole no navegador.", VERM_C)
    end)
end

local function iniciar()
    -- Key salva? Tenta carregar direto (sem UI)
    local chave = carregar_key()
    if chave ~= "" then
        print("[Red Onyx Hub] Key salva encontrada! Carregando automaticamente...")
        spawn(function()
            local ok = carregar_hub(true, chave)
            if ok then return end
            -- Key expirou/inválida
            print("[Red Onyx Hub] Key salva invalida - mostrando UI")
            deletar_key()
            criar_ui_real()
        end)
    else
        criar_ui_real()
    end
end

-- ============================================================
-- INICIALIZAÇÃO
-- ============================================================
iniciar_patcher()

local ok, err = pcall(iniciar)
if not ok then
    warn("[Red Onyx Hub] ERRO CRITICO: " .. tostring(err))
    -- Fallback: carrega free direto
    spawn(function()
        aguardar(2)
        pcall(function() carregar_hub(false, nil) end)
    end)
end

print("[Red Onyx Hub] v16 pronto!")
