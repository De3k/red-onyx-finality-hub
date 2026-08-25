-- ============================================================
-- RED ONYX HUB v15 - PATCHER FUNCIONAL + AUTO PEGA TUDO
-- ============================================================
print("[Red Onyx] Iniciando...")

-- ===== COMPAT =====
local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end

-- ===== ENV =====
local GENV = _G
pcall(function() local ok, e = pcall(getgenv); if ok then GENV = e end end)

-- ===== CORES =====
local VERM = Color3.fromRGB(200, 30, 45)
local VERM_C = Color3.fromRGB(240, 80, 90)
local VERM_E = Color3.fromRGB(130, 15, 28)
local ESC = Color3.fromRGB(15, 12, 14)
local ESC_M = Color3.fromRGB(22, 16, 18)
local CIN_M = Color3.fromRGB(30, 22, 25)
local BRA = Color3.fromRGB(240, 235, 237)
local CIN = Color3.fromRGB(150, 140, 142)

-- ===== CONSTANTES =====
local SCRIPT_ID = "0ae9fe4cf963e3a13d25eed0e2ce5940"
local URL_FREE = "https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua"
local URL_PREMIUM = "https://api.luarmor.net/files/v4/loaders/" .. SCRIPT_ID .. ".lua"
local LINK_KEY = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"
local ARQ_KEY = "RedOnyxKey.txt"

-- ============================================================
-- PATCHER VERMELHO/PRETO LEVE (mas funcional)
-- ============================================================
local function patcher_funcional()
    spawn(function()
        aguardar(2)

        -- Funções de cor
        local function lum(c) return c.R * 0.299 + c.G * 0.587 + c.B * 0.114 end

        local function cor_fundo(c)
            local l = lum(c)
            if l < 0.15 then return ESC
            elseif l < 0.25 then return ESC_M
            elseif l < 0.35 then return CIN_M
            else return Color3.fromRGB(42, 30, 35) end
        end

        local function cor_texto(c)
            return lum(c) < 0.4 and BRA or CIN
        end

        local function cor_borda(c)
            return lum(c) < 0.2 and VERM_E or VERM
        end

        -- Aplica cores num objeto
        local function aplicar(objeto)
            pcall(function()
                if objeto:IsA("GuiObject") then
                    objeto.BackgroundColor3 = cor_fundo(objeto.BackgroundColor3)
                    objeto.BorderColor3 = VERM_E
                end
                if objeto:IsA("TextLabel") or objeto:IsA("TextButton") then
                    objeto.TextColor3 = cor_texto(objeto.TextColor3)
                    -- Renomeia
                    local txt = objeto.Text
                    if type(txt) == "string" and txt ~= "" then
                        local tl = txt:lower()
                        if tl:find("quantum") or tl:find("onyx") or tl:find("kaitun") then
                            objeto.Text = "Red Onyx Hub"
                        end
                    end
                end
                if objeto:IsA("TextBox") then
                    objeto.TextColor3 = BRA
                    objeto.PlaceholderColor3 = CIN
                end
                if objeto:IsA("UIStroke") then
                    objeto.Color = VERM
                end
                if objeto:IsA("ScrollingFrame") then
                    objeto.ScrollBarImageColor3 = VERM_C
                end
                if (objeto:IsA("ImageLabel") or objeto:IsA("ImageButton")) and objeto.Image ~= "" then
                    pcall(function() objeto.ImageColor3 = VERM_C end)
                end
                if objeto:IsA("UIGradient") then
                    local ks = objeto.Color.Keypoints
                    local novos = {}
                    for i = 1, #ks do
                        novos[i] = ColorSequenceKeypoint.new(ks[i].Time, cor_fundo(ks[i].Value))
                    end
                    objeto.Color = ColorSequence.new(novos)
                end
            end)
        end

        -- Escaneia tudo de uma GUI
        local function escanear_gui(gui)
            for _, obj in ipairs(gui:GetDescendants()) do
                aplicar(obj)
            end
        end

        -- Loop principal (a cada 2s para não pesar)
        local function loop_patcher()
            local Core = game:GetService("CoreGui")
            local pcall_ok = pcall(function()
                for _, gui in ipairs(Core:GetChildren()) do
                    if gui:IsA("ScreenGui") then escanear_gui(gui) end
                end
                if GENV.HIDEUI and typeof(GENV.HIDEUI) == "Instance" then
                    escanear_gui(GENV.HIDEUI)
                end
                local plr = game:GetService("Players").LocalPlayer
                if plr and plr.PlayerGui then
                    for _, gui in ipairs(plr.PlayerGui:GetChildren()) do
                        if gui:IsA("ScreenGui") then escanear_gui(gui) end
                    end
                end
            end)
        end

        -- Monitora novas GUIs (leve, sem conexões infinitas)
        local Core = game:GetService("CoreGui")
        local function monitorar()
            pcall(function()
                for _, gui in ipairs(Core:GetChildren()) do
                    if gui:IsA("ScreenGui") then
                        -- Aplica nos descendentes atuais
                        for _, obj in ipairs(gui:GetDescendants()) do
                            aplicar(obj)
                        end
                    end
                end
            end)
        end

        -- Roda o patcher
        while true do
            loop_patcher()
            aguardar(2)
            -- A cada 10s (5 ciclos), forca uma varredura completa
            -- O loop acima já faz isso a cada 2s
        end
    end)
end

-- ============================================================
-- LIMPAR PATCHER PESADO DO QUANTUM ONYX
-- ============================================================
local function limpar_codigo(codigo)
    -- Remove O PATCHER INTEIRO do QuantumOnyx (que causa crash)
    -- Padrão: procura por blocos de código que conectam eventos de GUI

    -- Remove TODAS as conexões de evento que causam memory leak
    codigo = codigo:gsub("(.-)DescendantAdded:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)ChildAdded:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)BackgroundColor3Changed:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)TextColor3Changed:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)ImageColor3Changed:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)BorderColor3Changed:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)PlaceholderColor3Changed:Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)ImageChanged:Connect(.-)\n", "-- patcher desativado\n")

    -- Remove PropertyChangedSignal do patcher
    codigo = codigo:gsub("(.-)GetPropertyChangedSignal%(\"Text\"%):Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)GetPropertyChangedSignal%(\"Color\"%):Connect(.-)\n", "-- patcher desativado\n")
    codigo = codigo:gsub("(.-)GetPropertyChangedSignal%(\"Image\"%):Connect(.-)\n", "-- patcher desativado\n")

    -- Renomeia Quantum Onyx -> Red Onyx no código fonte
    codigo = codigo:gsub("Quantum Onyx Hub", "Red Onyx Hub")
    codigo = codigo:gsub("Quantum Onyx", "Red Onyx Hub")
    codigo = codigo:gsub("QUANTUM ONYX", "RED ONYX HUB")
    codigo = codigo:gsub("Kaitun", "Red Onyx Hub")
    codigo = codigo:gsub("kaitun", "Red Onyx Hub")

    return codigo
end

-- ============================================================
-- KEY PERSISTÊNCIA
-- ============================================================
local function salvar_key(chave)
    pcall(function()
        if makefolder then pcall(makefolder, "RedOnyx") end
        writefile(ARQ_KEY, chave)
    end)
    pcall(function()
        if syn and syn.writefile then
            if syn.makefolder then pcall(syn.makefolder, "RedOnyx") end
            syn.writefile(ARQ_KEY, chave)
        end
    end)
end

local function carregar_key()
    local chave = ""
    pcall(function() chave = readfile(ARQ_KEY) end)
    if chave == "" then
        pcall(function() if syn and syn.readfile then chave = syn.readfile(ARQ_KEY) end end)
    end
    return chave
end

local function deletar_key()
    pcall(delfile, ARQ_KEY)
    pcall(function() if syn and syn.delfile then syn.delfile(ARQ_KEY) end end)
end

-- ============================================================
-- DOWNLOAD
-- ============================================================
local function baixar(url)
    local dados = ""
    pcall(function() dados = game:HttpGet(url) end)
    if dados == "" then
        local req = (type(syn) == "table" and syn.request) or request or http_request
        if req then
            pcall(function()
                local resp = req({Url = url, Method = "GET"})
                if resp and resp.Body and #resp.Body > 0 then dados = resp.Body end
            end)
        end
    end
    return dados
end

-- ============================================================
-- ATIVAR CONFIGURAÇÕES (lido pelo QuantumOnyx)
-- ============================================================
GENV.Settings = GENV.Settings or {}
local S = GENV.Settings

local function ativar_tudo()
    S["Auto Farm Level"] = true
    S["Auto Quest"] = true
    S["Auto Bones"] = true
    S["Auto Elite Hunter"] = true
    S["Auto Factory Raid"] = true
    S["Auto Pirate Raid"] = true
    S["Auto Redeem Codes"] = true
    S["Auto Yama"] = true
    S["Auto TTK"] = true
    S["Auto Tushita"] = true
    S["Auto Ghoul"] = true
    S["Auto Get Ghoul"] = true
    S["Auto CDK"] = true
    S["Auto Rengoku"] = true
    S["Auto Soul Guitar"] = true
    S["Auto Shark Anchor"] = true
    S["Auto Rainbow Haki"] = true
    S["Auto Swords"] = true
    S["Auto Guns"] = true
    S["Auto Fighting Style"] = true
    S["Auto Mastery"] = true
    S["Auto Fragment"] = true
    S["Auto Ectoplasm"] = true
    S["Auto Candy"] = true
    S["Auto Chest"] = true
    S["Auto Sea Beast"] = true
    S["Auto Ship"] = true
    S["Auto Dungeon"] = true
    S["Auto Observation"] = true
    S["Auto Buso Haki"] = true
    S["Auto Soru"] = true
    S["Auto Geppo"] = true
    S["Auto Race V2"] = true
    S["Farm Distance"] = 2500
    S["Team"] = "Pirates"
    S["Melee"] = true
    S["Blox Fruit"] = true
    S["Sword"] = true
    S["Gun"] = true
    print("[Red Onyx] Auto Pega Tudo ATIVADO!")
end

-- ============================================================
-- CARREGAR HUB
-- ============================================================
local function carregar_hub(premium, chave)
    ativar_tudo()

    if premium and chave and chave ~= "" then
        GENV.script_key = chave
        salvar_key(chave)

        local codigo = baixar(URL_PREMIUM)
        if codigo ~= "" then
            codigo = limpar_codigo(codigo)
            local fn = loadstring(codigo)
            if fn then
                local ok, err = pcall(fn)
                if ok then
                    print("[Red Onyx] Premium carregado!")
                    return true, "OK"
                end
                warn("[Red Onyx Premium erro]: " .. tostring(err))
            end
        end
    end

    -- Free
    local codigo = baixar(URL_FREE)
    if codigo == "" then return false, "Download falhou (sem internet?)" end

    codigo = limpar_codigo(codigo)
    local fn = loadstring(codigo)
    if not fn then return false, "Falha na compilacao" end

    local ok, err = pcall(fn)
    if not ok then return false, tostring(err) end

    print("[Red Onyx] Hub carregado com sucesso!")
    return true, "OK"
end

-- ============================================================
-- VALIDAR KEY
-- ============================================================
local function validar_key_luarmor(chave, cb_ok, cb_erro)
    spawn(function()
        local sdk = baixar("https://sdkapi-public.luarmor.net/library.lua")
        if sdk == "" then
            if cb_erro then cb_erro("SDK offline (sem internet)") end
            return
        end
        local fn = loadstring(sdk)
        if not fn then
            if cb_erro then cb_erro("SDK invalido") end
            return
        end
        local ok_api, api = pcall(fn)
        if not ok_api or type(api) ~= "table" then
            if cb_erro then cb_erro("SDK erro") end
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
-- UI
-- ============================================================
local function criar_ui()
    -- Verifica key salva
    local chave = carregar_key()
    if chave ~= "" then
        print("[Red Onyx] Key salva encontrada!")
        spawn(function()
            local ok, err = carregar_hub(true, chave)
            if ok then return end
            deletar_key()
            criar_ui_real()
        end)
        return
    end

    criar_ui_real()
end

local function criar_ui_real()
    local sg = Instance.new("ScreenGui")
    sg.Name = "RedOnyxHub"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false

    local pai_ok = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not pai_ok then
        pcall(function()
            local hui = gethui()
            if typeof(hui) == "Instance" then sg.Parent = hui end
        end)
    end

    -- Card
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 380, 0, 200)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = ESC_M
    card.BorderSizePixel = 0
    card.Parent = sg

    local canto = Instance.new("UICorner")
    canto.CornerRadius = UDim.new(0, 10)
    canto.Parent = card

    local borda = Instance.new("UIStroke")
    borda.Color = VERM
    borda.Thickness = 2
    borda.Parent = card

    -- Título
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

    -- Sub
    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, 0, 0, 12)
    sub.Position = UDim2.new(0, 0, 0, 42)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.Gotham
    sub.Text = "Auto Pega Tudo | Patcher Vermelho e Preto"
    sub.TextColor3 = CIN
    sub.TextSize = 10
    sub.TextXAlignment = Enum.TextXAlignment.Center
    sub.Parent = card

    -- Input key
    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 340, 0, 34)
    input.Position = UDim2.new(0.5, 0, 0, 65)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = ESC
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Key Premium (opcional - deixe vazio para Free)"
    input.PlaceholderColor3 = CIN
    input.Text = ""
    input.TextColor3 = BRA
    input.TextSize = 12
    input.ClearTextOnFocus = false
    input.Parent = card

    local ci = Instance.new("UICorner")
    ci.CornerRadius = UDim.new(0, 6)
    ci.Parent = input

    -- Status
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(1, -20, 0, 14)
    status.Position = UDim2.new(0.5, 0, 0, 105)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = VERM_C
    status.TextSize = 10
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.Parent = card

    local function set_status(txt, cor)
        status.Text = txt
        if cor then status.TextColor3 = cor end
    end

    local ocupado = false

    -- Botão Free
    local function criar_btn(posX, cor, txt, cb)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 170, 0, 34)
        btn.Position = UDim2.new(0, posX, 0, 125)
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
                sg:Destroy()
            else
                ocupado = false
                set_status("Erro: " .. tostring(err), VERM)
            end
        end)
    end)

    criar_btn(195, VERM, "INICIAR PREMIUM", function()
        if ocupado then return end
        local chave = input.Text
        if chave == "" then
            set_status("Digite a key premium ou use Free!", VERM)
            return
        end
        ocupado = true
        set_status("Validando key...", VERM_C)

        validar_key_luarmor(chave,
            function()
                set_status("Key valida! Carregando...", VERM_C)
                aguardar(0.3)
                local ok, err = carregar_hub(true, chave)
                if ok then
                    sg:Destroy()
                else
                    ocupado = false
                    set_status("Erro: " .. tostring(err), VERM)
                end
            end,
            function(msg)
                ocupado = false
                set_status(msg, VERM)
            end
        )
    end)

    -- Fechar com ESC
    spawn(function()
        aguardar(0.5)
        local cInp = game:GetService("ContextActionService")
        cInp:BindActionAt("FecharRedOnyx", function()
            sg:Destroy()
        end, false, Enum.KeyCode.Escape)
    end)
end

-- ============================================================
-- INICIAR TUDO
-- ============================================================

-- 1. Patcher funcional começa AGORA (em background)
print("[Red Onyx] Iniciando patcher vermelho/preto...")
patcher_funcional()

-- 2. Carrega UI ou key salva
local ok, err = pcall(criar_ui)
if not ok then
    warn("[Red Onyx] ERRO: " .. tostring(err))
    local chave = carregar_key()
    if chave ~= "" then
        pcall(function() carregar_hub(true, chave) end)
    else
        pcall(function() carregar_hub(false, nil) end)
    end
end

print("[Red Onyx] Pronto! Pressione ESC para fechar a UI")
