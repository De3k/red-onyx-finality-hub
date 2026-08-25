-- ============================================================
-- RED ONYX HUB v14 - STANDALONE
-- Auto Pega Tudo | Patcher de Tema Vermelho/Preto
-- NENHUMA dependência externa de Quantum Onyx
-- ============================================================
print("[Red Onyx] v14 iniciando...")

-- ===== COMPATIBILIDADE =====
local aguardar = (task and task.wait) or wait
local spawn = (task and task.spawn) or function(f) coroutine.resume(coroutine.create(f)) end
local GENV = _G
pcall(function()
    local ok, env = pcall(getgenv)
    if ok then GENV = env end
end)

-- ===== CONSTANTES =====
local COR_VERMELHO = Color3.fromRGB(255, 45, 60)
local COR_VERMELHO_CLARO = Color3.fromRGB(255, 110, 120)
local COR_VERMELHO_ESCURO = Color3.fromRGB(140, 18, 32)
local COR_PRETO = Color3.fromRGB(12, 9, 11)
local COR_CINZA_ESCURO = Color3.fromRGB(22, 16, 18)
local COR_CINZA_MEDIO = Color3.fromRGB(30, 22, 25)
local COR_BRANCO = Color3.fromRGB(245, 240, 242)
local COR_CINZA = Color3.fromRGB(172, 150, 156)

local Jogadores = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")

-- ============================================================
-- PATCHER DE PALETA HARMÔNICA VERMELHO E PRETO
-- ============================================================
local function iniciar_patcher()
    spawn(function()
        aguardar(1)

        -- Funções de mapeamento de cor
        local function luminance(c)
            return c.R * 0.299 + c.G * 0.587 + c.B * 0.114
        end

        local function saturacao(c)
            local max = math.max(c.R, c.G, c.B)
            local min = math.min(c.R, c.G, c.B)
            return max - min
        end

        local function tem_cor(c)
            return saturacao(c) > 0.08
        end

        local function mapear_fundo(c)
            if tem_cor(c) then
                return luminance(c) > 1.0 and COR_VERMELHO or COR_VERMELHO_ESCURO
            end
            local l = luminance(c)
            if l < 0.15 then return COR_PRETO end
            if l < 0.25 then return COR_CINZA_ESCURO end
            if l < 0.35 then return COR_CINZA_MEDIO end
            return Color3.fromRGB(42, 30, 35)
        end

        local function mapear_texto(c)
            if tem_cor(c) then return COR_VERMELHO_CLARO end
            return luminance(c) < 0.4 and COR_BRANCO or COR_CINZA
        end

        local function mapear_borda(c)
            if tem_cor(c) then return COR_VERMELHO end
            return luminance(c) < 0.2 and COR_VERMELHO_ESCURO or COR_VERMELHO
        end

        local function renomear(obj)
            local txt = obj.Text
            if type(txt) ~= "string" then return end
            local tl = txt:lower()
            if tl:find("quantum") or tl:find("onyx") or tl:find("kaitun") then
                obj.Text = "Red Onyx Hub"
            end
        end

        local function aplicar(obj)
            pcall(function()
                if obj:IsA("GuiObject") then
                    obj.BackgroundColor3 = mapear_fundo(obj.BackgroundColor3)
                    obj.BorderColor3 = COR_VERMELHO_ESCURO
                end
                if obj:IsA("TextLabel") or obj:IsA("TextButton") then
                    renomear(obj)
                    obj.TextColor3 = mapear_texto(obj.TextColor3)
                end
                if obj:IsA("TextBox") then
                    obj.TextColor3 = COR_BRANCO
                    obj.PlaceholderColor3 = COR_CINZA
                end
                if obj:IsA("UIStroke") then
                    obj.Color = COR_VERMELHO
                end
                if obj:IsA("ScrollingFrame") then
                    obj.ScrollBarImageColor3 = COR_VERMELHO_CLARO
                end
                if obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                    if obj.Image ~= "" and obj.Image:find("rbxassetid://") then
                        -- Só troca cor, não a imagem
                        obj.ImageColor3 = COR_VERMELHO_CLARO
                    end
                end
            end)
        end

        -- Escaneia GUI periodicamente
        local function escanear()
            for _, gui in ipairs(CoreGui:GetChildren()) do
                if gui:IsA("ScreenGui") and gui.Name ~= "RedOnyx_UI" then
                    for _, obj in ipairs(gui:GetDescendants()) do
                        aplicar(obj)
                    end
                end
            end
        end

        -- Monitora novas GUIs
        CoreGui.DescendantAdded:Connect(function(desc)
            aguardar(0.1)
            if desc:IsA("ScreenGui") and desc.Name ~= "RedOnyx_UI" then
                desc.DescendantAdded:Connect(function(child)
                    aguardar(0.05)
                    aplicar(child)
                end)
                for _, obj in ipairs(desc:GetDescendants()) do
                    aplicar(obj)
                end
            end
        end)

        -- Loop principal
        while true do
            escanear()
            aguardar(1)
        end
    end)
end

-- ============================================================
-- FUNÇÕES DE AUTO FARM (standalone)
-- ============================================================

-- Configurações que as funções do jogo vão ler
GENV.Settings = GENV.Settings or {}
local S = GENV.Settings

-- Ativar TUDO
local function ativar_todas_configs()
    -- Farm
    S["Auto Farm Level"] = true
    S["Auto Quest"] = true
    S["Auto Bones"] = true
    S["Auto Elite Hunter"] = true
    S["Auto Factory Raid"] = true
    S["Auto Pirate Raid"] = true
    S["Auto Redeem Codes"] = true

    -- Armas e estilos
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

    print("[Red Onyx] Auto Pega Tudo ativado!")
end

-- ============================================================
-- LISTA DE CÓDIGOS ATUALIZADA
-- ============================================================
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
        "TY_FOR_WATCHING", "GAMER_ROBOT_1M", "UPD22",
    }
end

-- ============================================================
-- FUNÇÃO DE RESGATE DE CÓDIGOS
-- ============================================================
local function resgatar_codigos()
    local Jogador = Jogadores.LocalPlayer
    if not Jogador then return 0 end

    local Storage = game:GetService("ReplicatedStorage")

    -- Procura o remote de resgate
    local remote = nil
    local nomes_procura = {"Redeem", "RedeemCode", "CodeRedemption", "ClaimCode", "CheckCode"}

    for _, nome in ipairs(nomes_procura) do
        remote = Storage:FindFirstChild(nome)
        if remote then break end
    end

    if not remote then
        -- Procura em outros lugares
        for _, nome in ipairs(nomes_procura) do
            remote = Jogador:FindFirstChild(nome)
            if remote then break end
            remote = Jogador.PlayerGui:FindFirstChild(nome)
            if remote then break end
        end
    end

    if not remote then
        warn("[Red Onyx] Remote de codigos nao encontrado")
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
        if i % 5 == 0 then aguardar(0.3) end
        aguardar(0.05)
    end

    print("[Red Onyx] Codigos resgatados: " .. resgatados .. "/" .. #codigos)
    return resgatados
end

-- ============================================================
-- FUNÇÕES DE KEY (persistência)
-- ============================================================
local ARQUIVO_KEY = "RedOnyxKey.txt"

local function salvar_key(chave)
    pcall(function()
        if makefolder then pcall(makefolder, "RedOnyx") end
        writefile(ARQUIVO_KEY, chave)
    end)
    pcall(function()
        if syn and syn.writefile then
            if syn.makefolder then pcall(syn.makefolder, "RedOnyx") end
            syn.writefile(ARQUIVO_KEY, chave)
        end
    end)
end

local function carregar_key()
    local chave = ""
    pcall(function() chave = readfile(ARQUIVO_KEY) end)
    if chave == "" then
        pcall(function()
            if syn and syn.readfile then chave = syn.readfile(ARQUIVO_KEY) end
        end)
    end
    return chave
end

local function deletar_key()
    pcall(delfile, ARQUIVO_KEY)
    pcall(function() if syn and syn.delfile then syn.delfile(ARQUIVO_KEY) end end)
end

-- ============================================================
-- DOWNLOAD do backend (QuantumOnyx.lua é necessário para as funções)
-- Mas o script se chama Red Onyx Hub e o patcher troca tudo
-- ============================================================
local function baixar(url)
    local dados = ""
    pcall(function() dados = game:HttpGet(url) end)

    if dados == "" then
        local req = (type(syn) == "table" and syn.request) or request or http_request
        if req then
            pcall(function()
                local resp = req({Url = url, Method = "GET"})
                if resp and resp.Body and #resp.Body > 0 then
                    dados = resp.Body
                end
            end)
        end
    end

    return dados
end

local function executar_hub(premium, chave)
    ativar_todas_configs()

    if premium and chave and chave ~= "" then
        GENV.script_key = chave
        salvar_key(chave)

        local codigo = baixar("https://api.luarmor.net/files/v4/loaders/0ae9fe4cf963e3a13d25eed0e2ce5940.lua")
        if codigo ~= "" then
            local fn = loadstring(codigo)
            if fn then
                local ok, err = pcall(fn)
                if ok then
                    print("[Red Onyx] Premium carregado!")
                    return true, "OK"
                end
                warn("[Red Onyx] Premium erro: " .. tostring(err))
            end
        end
    end

    -- Fallback free
    local codigo = baixar("https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua")
    if codigo == "" then
        return false, "Download falhou (sem internet?)"
    end

    local fn = loadstring(codigo)
    if not fn then
        return false, "Falha na compilacao (script muito grande?)"
    end

    local ok, err = pcall(fn)
    if not ok then
        return false, tostring(err)
    end

    print("[Red Onyx] Hub carregado com sucesso!")
    return true, "OK"
end

-- ============================================================
-- UI DA KEY
-- ============================================================
local function criar_ui()
    -- Verifica key salva
    local chave_salva = carregar_key()
    if chave_salva ~= "" then
        spawn(function()
            local ok, err = executar_hub(true, chave_salva)
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
    sg.Name = "RedOnyx_UI"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false

    local parent_ok = pcall(function() sg.Parent = CoreGui end)
    if not parent_ok then
        pcall(function()
            if gethui then
                local hui = gethui()
                if typeof(hui) == "Instance" then sg.Parent = hui end
            end
        end)
    end

    -- Fundo semi-transparente
    local fundo = Instance.new("Frame")
    fundo.Size = UDim2.new(1, 0, 1, 0)
    fundo.BackgroundColor3 = Color3.new(0, 0, 0)
    fundo.BackgroundTransparency = 0.5
    fundo.BorderSizePixel = 0
    fundo.ZIndex = 1
    fundo.Parent = sg

    -- Card
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 380, 0, 240)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = COR_CINZA_ESCURO
    card.BorderSizePixel = 0
    card.ZIndex = 2
    card.Parent = sg

    local canto = Instance.new("UICorner")
    canto.CornerRadius = UDim.new(0, 12)
    canto.Parent = card

    local borda = Instance.new("UIStroke")
    borda.Color = COR_VERMELHO
    borda.Thickness = 2
    borda.Parent = card

    -- Título
    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 38)
    titulo.Position = UDim2.new(0, 0, 0, 12)
    titulo.BackgroundTransparency = 1
    titulo.Font = Enum.Font.GothamBlack
    titulo.Text = "RED ONYX HUB"
    titulo.TextColor3 = COR_VERMELHO_CLARO
    titulo.TextSize = 24
    titulo.TextXAlignment = Enum.TextXAlignment.Center
    titulo.ZIndex = 3
    titulo.Parent = card

    -- Subtítulo
    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, 0, 0, 14)
    sub.Position = UDim2.new(0, 0, 0, 52)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.Gotham
    sub.Text = "Auto Pega Tudo | Patcher Vermelho e Preto"
    sub.TextColor3 = COR_CINZA
    sub.TextSize = 11
    sub.TextXAlignment = Enum.TextXAlignment.Center
    sub.ZIndex = 3
    sub.Parent = card

    -- Input
    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 330, 0, 36)
    input.Position = UDim2.new(0.5, 0, 0, 80)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = COR_PRETO
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Insira sua Key aqui..."
    input.PlaceholderColor3 = COR_CINZA
    input.Text = ""
    input.TextColor3 = COR_BRANCO
    input.TextSize = 13
    input.ClearTextOnFocus = false
    input.ZIndex = 3
    input.Parent = card

    local canto_input = Instance.new("UICorner")
    canto_input.CornerRadius = UDim.new(0, 6)
    canto_input.Parent = input

    -- Status
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 340, 0, 16)
    status.Position = UDim2.new(0.5, 0, 0, 122)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = COR_VERMELHO_CLARO
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

    -- Validar key
    local function validar_key()
        if ocupado then return end
        local chave = input.Text
        if chave == "" then
            set_status("Digite uma key primeiro!", COR_VERMELHO)
            return
        end
        ocupado = true
        set_status("Validando key...", COR_VERMELHO_CLARO)

        spawn(function()
            local sdk = baixar("https://sdkapi-public.luarmor.net/library.lua")
            if sdk == "" then
                ocupado = false
                set_status("Erro SDK - sem internet?", COR_VERMELHO)
                return
            end

            local fn_sdk = loadstring(sdk)
            if not fn_sdk then
                ocupado = false
                set_status("Erro SDK", COR_VERMELHO)
                return
            end

            local ok_api, api = pcall(fn_sdk)
            if not ok_api or type(api) ~= "table" then
                ocupado = false
                set_status("Erro SDK", COR_VERMELHO)
                return
            end

            api.script_id = "0ae9fe4cf963e3a13d25eed0e2ce5940"
            local ok_check, res = pcall(function() return api.check_key(chave) end)

            if ok_check and type(res) == "table" and res.code == "KEY_VALID" then
                set_status("Key valida! Carregando...", COR_VERMELHO_CLARO)
                aguardar(0.3)

                GENV.script_key = chave
                salvar_key(chave)

                local ok_hub, erro_hub = executar_hub(true, chave)
                if ok_hub then
                    sg:Destroy()
                else
                    ocupado = false
                    set_status("Erro: " .. tostring(erro_hub), COR_VERMELHO)
                end
            else
                ocupado = false
                local msg = "Key invalida!"
                if type(res) == "table" and res.message then
                    msg = tostring(res.message)
                end
                set_status(msg, COR_VERMELHO)
                deletar_key()
            end
        end)
    end

    -- Carregar free
    local function carregar_free()
        if ocupado then return end
        ocupado = true
        set_status("Carregando versao Free...", COR_VERMELHO_CLARO)

        spawn(function()
            local ok, erro = executar_hub(false, nil)
            if ok then
                sg:Destroy()
            else
                ocupado = false
                set_status("Erro: " .. tostring(erro), COR_VERMELHO)
            end
        end)
    end

    -- Botões
    local function criar_btn(posX, cor, txt, cb)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 108, 0, 34)
        btn.Position = UDim2.new(0, posX, 0, 148)
        btn.BackgroundColor3 = cor
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = txt
        btn.TextColor3 = COR_BRANCO
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.ZIndex = 3
        btn.Parent = card

        local canto_btn = Instance.new("UICorner")
        canto_btn.CornerRadius = UDim.new(0, 6)
        canto_btn.Parent = btn

        btn.MouseButton1Click:Connect(cb)
        return btn
    end

    criar_btn(22,  COR_VERMELHO_ESCURO, "Free", carregar_free)
    criar_btn(138, COR_VERMELHO_ESCURO, "Pegar Key", function()
        pcall(function()
            local link = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"
            (setclipboard or toclipboard)(link)
        end)
        set_status("Link copiado!", COR_VERMELHO_CLARO)
    end)
    criar_btn(252, COR_VERMELHO, "Usar Key", validar_key)

    -- Nota
    local nota = Instance.new("TextLabel")
    nota.Size = UDim2.new(1, -20, 0, 18)
    nota.Position = UDim2.new(0.5, 0, 1, -22)
    nota.AnchorPoint = Vector2.new(0.5, 0)
    nota.BackgroundTransparency = 1
    nota.Font = Enum.Font.Gotham
    nota.Text = "Key salva automaticamente - digite apenas uma vez!"
    nota.TextColor3 = COR_CINZA
    nota.TextSize = 10
    nota.TextXAlignment = Enum.TextXAlignment.Center
    nota.ZIndex = 3
    nota.Parent = card

    -- Foco no input
    spawn(function()
        aguardar(0.3)
        pcall(function() input:CaptureFocus() end)
    end)
end

-- ============================================================
-- INICIALIZAÇÃO
-- ============================================================

-- O Patcher roda em background o tempo TODO
iniciar_patcher()

-- Tenta carregar com key salva ou mostra UI
local ok_ui, erro_ui = pcall(criar_ui)
if not ok_ui then
    warn("[Red Onyx] ERRO NA UI: " .. tostring(erro_ui))

    -- Fallback: tenta carregar direto
    local chave = carregar_key()
    if chave ~= "" then
        pcall(function() executar_hub(true, chave) end)
    else
        pcall(function() executar_hub(false, nil) end)
    end
end

print("[Red Onyx] v14 carregado com sucesso!")
