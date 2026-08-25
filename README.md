-- Red Onyx Hub v11 - CORRIGIDO: key salva em disco, UI fecha, auto farm funcional
print("[Red Onyx Hub] v11 iniciando...")

-- ===== CONFIG =====
local LOGO_RBX = "rbxassetid://240328436758150"  -- "" para nao trocar
local KEY_FILE = "RedOnyxHub_Key.txt"  -- arquivo onde a key sera salva

-- ===== COMPAT =====
local waitF = task and task.wait or wait
local spawnF = task and task.spawn or function(f) coroutine.resume(coroutine.create(f)) end
local delayF = task and task.delay or function(t, f) spawnF(function() waitF(t); f() end) end

-- ===== ENV =====
local function getgenv_safe()
    local ok, g = pcall(function() return getgenv() end)
    return ok and g or _G
end
local function set_script_key(k)
    local env = getgenv_safe()
    env.script_key = k
    pcall(function() _G.script_key = k end)
end

-- ===== PERSISTENCIA DE KEY (writefile/readfile) =====
local function save_key_to_disk(key)
    local ok = false
    -- Tenta writefile (disponível na maioria dos executores)
    ok = pcall(function()
        -- Cria pasta se necessário
        local mk = makefolder and pcall(makefolder, "RedOnyxHub")
        writefile(KEY_FILE, key)
    end)
    if not ok then
        -- Fallback: tenta no diretório padrão
        ok = pcall(function() writefile(KEY_FILE, key) end)
    end
    if not ok then
        -- Fallback final: tenta syn.writefile
        ok = pcall(function()
            if syn and syn.writefile then
                syn.writefile(KEY_FILE, key)
            end
        end)
    end
    if not ok then
        warn("[Red Onyx Hub] Nao foi possivel salvar key em disco")
    end
    return ok
end

local function load_key_from_disk()
    local key = ""
    -- Tenta readfile
    local ok = pcall(function() key = readfile(KEY_FILE) end)
    if not ok then
        -- Fallback syn.readfile
        ok = pcall(function()
            if syn and syn.readfile then
                key = syn.readfile(KEY_FILE)
            end
        end)
    end
    if ok and type(key) == "string" and #key > 0 then
        print("[Red Onyx Hub] Key carregada do disco")
        return key
    end
    return ""
end

local function delete_key_from_disk()
    pcall(function() delfile(KEY_FILE) end)
    pcall(function()
        if syn and syn.delfile then syn.delfile(KEY_FILE) end
    end)
end

-- ===== CONST =====
local Players = game:GetService("Players")
local SCRIPT_ID = "0ae9fe4cf963e3a13d25eed0e2ce5940"
local MAIN_URL = "https://raw.githubusercontent.com/flazhy/QuantumOnyx/refs/heads/main/QuantumOnyx.lua"
local PREMIUM_URL = "https://api.luarmor.net/files/v4/loaders/0ae9fe4cf963e3a13d25eed0e2ce5940.lua"
local KEY_LINK = "https://ads.luarmor.net/get_key?for=Quantum_Onyx_Keysytem-NdUqNPMGBobv"

-- ===== PALETA =====
local RED   = Color3.fromRGB(255, 45, 60)
local RED_T = Color3.fromRGB(255, 110, 120)
local RED_D = Color3.fromRGB(140, 18, 32)
local DK    = Color3.fromRGB(12, 9, 11)
local DKW   = Color3.fromRGB(22, 16, 18)
local PAN   = Color3.fromRGB(30, 22, 25)
local CRD   = Color3.fromRGB(42, 30, 35)
local TX    = Color3.fromRGB(245, 240, 242)
local TD    = Color3.fromRGB(172, 150, 156)

-- ===== HTTP COM TIMEOUT REAL =====
local function http_get(url, timeout)
    timeout = timeout or 15
    local result = nil
    local done = false
    local t0 = tick()

    spawnF(function()
        local ok, data = pcall(function() return game:HttpGet(url) end)
        if ok then result = data end
        done = true
    end)

    while not done and tick() - t0 < timeout do
        waitF(0.05)
    end

    if done and type(result) == "string" and #result > 0 then
        return result
    end

    local req = type(syn) == "table" and syn.request or request or http_request
    if req then
        local ok, resp = pcall(req, { Url = url, Method = "GET" })
        if ok and resp and resp.StatusCode == 200 and type(resp.Body) == "string" and #resp.Body > 0 then
            return resp.Body
        end
    end

    return ""
end

-- ===== CODES ATUALIZADOS =====
local function get_all_codes()
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

-- ===== REDEEM ALL CODES =====
local function redeem_all_codes()
    local VirtualUser = game:GetService("VirtualUser")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local LocalPlayer = Players.LocalPlayer
    if not LocalPlayer then return 0 end

    local redeemRemote = nil
    local function search(container, names)
        if not container then return nil end
        for _, name in ipairs(names) do
            local found = container:FindFirstChild(name)
            if found then return found end
        end
        return nil
    end
    local names = {"Redeem", "RedeemCode", "CodeRedemption", "ClaimCode", "CheckCode"}
    redeemRemote = search(ReplicatedStorage, names)
    if not redeemRemote then redeemRemote = search(LocalPlayer, names) end

    if not redeemRemote then
        warn("[Red Onyx Hub] Remote de redeem nao encontrado")
        return 0
    end

    local codes = get_all_codes()
    local redeemed = 0
    for i, code in ipairs(codes) do
        local ok = pcall(function()
            if redeemRemote:IsA("RemoteFunction") then
                return redeemRemote:InvokeServer(code)
            elseif redeemRemote:IsA("RemoteEvent") then
                redeemRemote:FireServer(code)
                return true
            end
        end)
        if ok then redeemed = redeemed + 1 end
        if i % 5 == 0 then waitF(0.5) end
        waitF(0.1)
    end
    print("[Red Onyx Hub] Redeem: " .. redeemed .. "/" .. #codes)
    return redeemed
end

-- ===== CONFIG AUTO FARM =====
local function init_auto_farm_settings()
    local env = getgenv_safe()
    env.Settings = env.Settings or {}
    local S = env.Settings

    S["Auto Yama"]       = S["Auto Yama"] ~= nil and S["Auto Yama"] or true
    S["Auto TTK"]        = S["Auto TTK"] ~= nil and S["Auto TTK"] or true
    S["Auto Tushita"]    = S["Auto Tushita"] ~= nil and S["Auto Tushita"] or true
    S["Auto Ghoul"]      = S["Auto Ghoul"] ~= nil and S["Auto Ghoul"] or true
    S["Auto Get Ghoul"]  = S["Auto Get Ghoul"] ~= nil and S["Auto Get Ghoul"] or true
    S["Auto Redeem Codes"] = S["Auto Redeem Codes"] ~= nil and S["Auto Redeem Codes"] or true
    S["Auto Farm Level"] = S["Auto Farm Level"] ~= nil and S["Auto Farm Level"] or true
    S["Auto Quest"]      = S["Auto Quest"] ~= nil and S["Auto Quest"] or true
    S["Auto Bones"]      = S["Auto Bones"] ~= nil and S["Auto Bones"] or true
    S["Auto Elite Hunter"] = S["Auto Elite Hunter"] ~= nil and S["Auto Elite Hunter"] or true
    S["Auto Factory Raid"] = S["Auto Factory Raid"] ~= nil and S["Auto Factory Raid"] or true
    S["Auto Pirate Raid"]  = S["Auto Pirate Raid"] ~= nil and S["Auto Pirate Raid"] or true
    S["Auto CDK"]        = S["Auto CDK"] ~= nil and S["Auto CDK"] or true
    S["Auto Rengoku"]    = S["Auto Rengoku"] ~= nil and S["Auto Rengoku"] or true
    S["Auto Rainbow Haki"] = S["Auto Rainbow Haki"] ~= nil and S["Auto Rainbow Haki"] or true
    S["Auto Soul Guitar"]  = S["Auto Soul Guitar"] ~= nil and S["Auto Soul Guitar"] or true
    S["Auto Shark Anchor"] = S["Auto Shark Anchor"] ~= nil and S["Auto Shark Anchor"] or true
    S["Farm Distance"]   = S["Farm Distance"] or 2500
    S["Team"]            = S["Team"] or "Pirates"
    S["Auto Swords"]     = S["Auto Swords"] ~= nil and S["Auto Swords"] or true
    S["Auto Guns"]       = S["Auto Guns"] ~= nil and S["Auto Guns"] or true
    S["Auto Fighting Style"] = S["Auto Fighting Style"] ~= nil and S["Auto Fighting Style"] or true

    print("[Red Onyx Hub] Configuracoes de auto farm inicializadas")
end

-- ===== LOADER =====
local function load_hub(url, status_cb)
    if status_cb then status_cb("Baixando hub...") end
    local code = http_get(url)
    if type(code) ~= "string" or #code == 0 then
        return false, "download vazio (sem internet ou URL bloqueada)"
    end
    print("[Red Onyx Hub] Baixado " .. #code .. " bytes")
    if status_cb then status_cb("Compilando...") end

    local fn = (loadstring or load)(code)
    if type(fn) ~= "function" then
        return false, "compile falhou"
    end

    if status_cb then status_cb("Iniciando hub...") end
    local ok, err = pcall(fn)
    if not ok then
        return false, tostring(err)
    end
    return true, "ok"
end

-- ===== FUNÇÃO PRINCIPAL DE CARREGAMENTO =====
-- Usada tanto pelo free quanto pelo premium
local function execute_hub(is_premium, key, ui_to_destroy)
    -- Inicializa configuracoes primeiro
    init_auto_farm_settings()

    -- Se for premium, salva a key e tenta carregar premium
    if is_premium and key and key ~= "" then
        set_script_key(key)
        save_key_to_disk(key)

        local ok, err = load_hub(PREMIUM_URL)
        if ok then
            print("[Red Onyx Hub] Premium carregado com sucesso!")
            if ui_to_destroy then
                waitF(0.5)
                pcall(function() ui_to_destroy:Destroy() end)
            end
            return true
        else
            warn("[Red Onyx Hub] Erro no premium: " .. err .. " - tentando free...")
            -- Fallback: tenta free
        end
    end

    -- Carrega free (QuantumOnyx.lua original com as settings ativas)
    local ok, err = load_hub(MAIN_URL)
    if ok then
        print("[Red Onyx Hub] Hub carregado com sucesso!")
        if ui_to_destroy then
            waitF(0.5)
            pcall(function() ui_to_destroy:Destroy() end)
        end
        return true
    else
        warn("[Red Onyx Hub] Erro no hub: " .. err)
        return false, err
    end
end

-- ===== UI DE KEY =====
local function create_key_ui()
    local sg = Instance.new("ScreenGui")
    sg.Name = "KL_RedOnyxHub"
    sg.IgnoreGuiInset = true
    sg.ResetOnSpawn = false

    local ok_parent = pcall(function() sg.Parent = game:GetService("CoreGui") end)
    if not ok_parent then
        pcall(function() sg.Parent = gethui() end)
    end

    -- Tenta carregar key salva ANTES de mostrar UI
    local saved_key = load_key_from_disk()
    if saved_key ~= "" then
        print("[Red Onyx Hub] Key salva encontrada! Validando...")
        spawnF(function()
            local ok_load = execute_hub(true, saved_key, sg)
            if not ok_load then
                -- Key salva nao funcionou, mostra UI
                warn("[Red Onyx Hub] Key salva invalida, mostrando UI")
                delete_key_from_disk()
                create_key_ui_real(sg, saved_key)
            end
        end)
        return
    end

    -- Mostra UI normal
    create_key_ui_real(sg, "")
end

local function create_key_ui_real(sg, initial_key)
    -- Card principal
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 420, 0, 250)
    card.Position = UDim2.new(0.5, 0, 0.5, 0)
    card.AnchorPoint = Vector2.new(0.5, 0.5)
    card.BackgroundColor3 = DKW
    card.BorderSizePixel = 0
    card.ZIndex = 2
    card.Parent = sg

    local uc = Instance.new("UICorner")
    uc.CornerRadius = UDim.new(0, 10)
    uc.Parent = card

    local us = Instance.new("UIStroke")
    us.Color = RED
    us.Thickness = 2
    us.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    us.Parent = card

    -- Titulo
    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 38)
    titulo.Position = UDim2.new(0, 0, 0, 16)
    titulo.BackgroundTransparency = 1
    titulo.Font = Enum.Font.GothamBlack
    titulo.Text = "RED ONYX HUB"
    titulo.TextColor3 = RED_T
    titulo.TextSize = 24
    titulo.TextXAlignment = Enum.TextXAlignment.Center
    titulo.ZIndex = 3
    titulo.Parent = card

    local subtitulo = Instance.new("TextLabel")
    subtitulo.Size = UDim2.new(1, 0, 0, 14)
    subtitulo.Position = UDim2.new(0, 0, 0, 56)
    subtitulo.BackgroundTransparency = 1
    subtitulo.Font = Enum.Font.Gotham
    subtitulo.Text = "Free Version  +  Premium"
    subtitulo.TextColor3 = TD
    subtitulo.TextSize = 11
    subtitulo.TextXAlignment = Enum.TextXAlignment.Center
    subtitulo.ZIndex = 3
    subtitulo.Parent = card

    -- Input
    local input = Instance.new("TextBox")
    input.Size = UDim2.new(0, 360, 0, 34)
    input.Position = UDim2.new(0.5, 0, 0, 82)
    input.AnchorPoint = Vector2.new(0.5, 0)
    input.BackgroundColor3 = DK
    input.BorderSizePixel = 0
    input.Font = Enum.Font.Gotham
    input.PlaceholderText = "Insert your key here"
    input.PlaceholderColor3 = TD
    input.TextColor3 = TX
    input.TextSize = 13
    input.ClearTextOnFocus = false
    input.ZIndex = 3
    input.Parent = card
    if initial_key and initial_key ~= "" then
        input.Text = initial_key
    end

    local iuc = Instance.new("UICorner")
    iuc.CornerRadius = UDim.new(0, 6)
    iuc.Parent = input

    local ius = Instance.new("UIStroke")
    ius.Color = RED
    ius.Transparency = 0.5
    ius.Parent = input

    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 380, 0, 16)
    status.Position = UDim2.new(0.5, 0, 0, 120)
    status.AnchorPoint = Vector2.new(0.5, 0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.Text = ""
    status.TextColor3 = RED_T
    status.TextSize = 11
    status.TextWrapped = true
    status.TextXAlignment = Enum.TextXAlignment.Center
    status.ZIndex = 3
    status.Parent = card
    local function set_status(txt, cor)
        status.Text = txt
        if cor then status.TextColor3 = cor end
    end

    local busy = false

    -- Validar key e carregar
    local function validate_and_load(k, is_free)
        if busy then return end
        busy = true

        if is_free then
            set_status("Carregando versao free...", RED_T)
            spawnF(function()
                local ok, err = execute_hub(false, nil, sg)
                if ok then
                    -- sg foi destruido em execute_hub
                else
                    busy = false
                    set_status("Erro: " .. tostring(err), RED_T)
                end
            end)
            return
        end

        -- Premium
        if not k or k == "" then
            busy = false
            set_status("Digite uma key primeiro.", RED_T)
            return
        end

        set_status("Validando key...", RED_T)
        spawnF(function()
            -- Baixa SDK
            local code = http_get("https://sdkapi-public.luarmor.net/library.lua")
            if #code == 0 then
                busy = false
                set_status("Erro no SDK - sem internet?", RED_T)
                return
            end

            local api_fn = (loadstring or load)(code)
            if type(api_fn) ~= "function" then
                busy = false
                set_status("Erro no SDK", RED_T)
                return
            end

            local ok_api, api = pcall(api_fn)
            if not ok_api or type(api) ~= "table" then
                busy = false
                set_status("Erro no SDK - tente de novo.", RED_T)
                return
            end

            api.script_id = SCRIPT_ID
            local ok_check, result = pcall(function() return api.check_key(k) end)

            if ok_check and type(result) == "table" and result.code == "KEY_VALID" then
                set_status("Key valida! Carregando...", RED_T)
                waitF(0.3)

                -- SALVA a key em disco AGORA
                save_key_to_disk(k)
                set_script_key(k)

                -- Carrega o hub (tenta premium, fallback free)
                local ok_load = execute_hub(true, k, sg)
                if ok_load then
                    -- sg destruido em execute_hub
                else
                    busy = false
                    set_status("Erro ao carregar hub", RED_T)
                    -- Nao deleta a key salva - ela e valida, o problema e o download
                end
            else
                busy = false
                local msg = "Key invalida."
                if type(result) == "table" then
                    msg = tostring(result.message or result.code)
                end
                set_status(msg, RED_T)
                -- Key invalida: deleta do disco se estava la
                delete_key_from_disk()
            end
        end)
    end

    -- Criar botao
    local function make_button(pos_x, color, label, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 112, 0, 32)
        btn.Position = UDim2.new(0, pos_x, 0, 150)
        btn.BackgroundColor3 = color
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Text = label
        btn.TextColor3 = TX
        btn.TextSize = 11
        btn.AutoButtonColor = false
        btn.ZIndex = 3
        btn.Parent = card

        local bc = Instance.new("UICorner")
        bc.CornerRadius = UDim.new(0, 6)
        bc.Parent = btn

        local bs = Instance.new("UIStroke")
        bs.Color = RED
        bs.Transparency = 0.55
        bs.Parent = btn

        btn.MouseButton1Click:Connect(function()
            local ok, err = pcall(callback)
            if not ok then warn("[Red Onyx Hub] Erro no botao: " .. tostring(err)) end
        end)
        pcall(function()
            btn.Activated:Connect(function()
                local ok, err = pcall(callback)
                if not ok then warn("[Red Onyx Hub] Erro no botao: " .. tostring(err)) end
            end)
        end)
        return btn
    end

    make_button(38,  RED_D, "Free Version", function() validate_and_load(nil, true) end)
    make_button(154, RED_D, "Get Key", function()
        pcall(function() (setclipboard or toclipboard)(KEY_LINK) end)
        set_status("Link copiado!", RED_T)
    end)
    make_button(270, RED,   "Enter Key", function() validate_and_load(input.Text, false) end)
end

-- ===== PATCHER DE TEMA =====
spawnF(function()
    local containers = { game:GetService("CoreGui") }
    local plr = Players.LocalPlayer
    if plr then table.insert(containers, plr.PlayerGui) end

    local env = getgenv_safe()
    if typeof(env.HIDEUI) == "Instance" then table.insert(containers, env.HIDEUI) end
    pcall(function()
        if gethui then
            local hui = gethui()
            if typeof(hui) == "Instance" then table.insert(containers, hui) end
        end
    end)

    local base_guis = {}
    pcall(function()
        for _, container in ipairs(containers) do
            if typeof(container) == "Instance" then
                for _, gui in ipairs(container:GetChildren()) do
                    if gui:IsA("ScreenGui") then base_guis[gui] = true end
                end
            end
        end
    end)

    local tracked_guis = {}
    local watched_objects = {}

    local function luminance(c) return c.R + c.G + c.B end
    local function saturation(c) return math.max(c.R, c.G, c.B) - math.min(c.R, c.G, c.B) end
    local function is_colored(c) return saturation(c) > 0.1 and luminance(c) > 0.3 end
    local function map_background(c)
        if is_colored(c) then return luminance(c) > 1.3 and RED or RED_D end
        local l = luminance(c)
        if l < 0.1725 then return DK elseif l < 0.2608 then return DKW elseif l < 0.3608 then return PAN else return CRD end
    end
    local function map_text(c)
        if is_colored(c) then return RED_T end
        local l = luminance(c); if l < 0.5 then return TX elseif l < 1.9 then return TD else return TX end
    end
    local function map_image(c)
        if is_colored(c) then return RED_T end; return luminance(c) < 1.9 and TD or TX
    end
    local function map_gradient(c)
        if is_colored(c) then return RED end
        local l = luminance(c); if l < 0.1725 then return DK elseif l < 0.2608 then return DKW elseif l < 0.3608 then return PAN else return RED end
    end
    local function is_link_text(t)
        if type(t) ~= "string" or #t == 0 then return false end
        local tl = t:lower(); return tl:find("^https?://") or tl:find("^rbxassetid://") or tl:find("^rbxasset://")
    end
    local function rename_text(obj)
        local t = obj.Text; if type(t) ~= "string" or #t == 0 or is_link_text(t) then return end
        local tl = t:lower()
        if tl:find("quantum") or tl:find("onyx") or tl:find("kaitun") then
            obj.Text = t:gsub("[Qq]uantum%s*[Oo]nyx%s*[Hh]ub", "Red Onyx Hub"):gsub("[Qq]uantum%s*[Oo]nyx", "Red Onyx Hub"):gsub("[Kk]aitun", "Red Onyx Hub"):gsub("[Qq]uantum", "Red Onyx")
        end
    end
    local function try_replace_logo(obj)
        if LOGO_RBX == "" then return false end
        if not (obj:IsA("ImageLabel") or obj:IsA("ImageButton")) then return false end
        if obj.Image == LOGO_RBX then return true end
        local img = tostring(obj.Image); if not img:find("rbxassetid://") then return false end
        local ok, size = pcall(function() return obj.AbsoluteSize end)
        if not ok or not size or size.X < 40 or size.Y < 30 then return false end
        pcall(function() obj.Image = LOGO_RBX end); print("[Red Onyx Hub] Logo trocado em " .. tostring(obj.Name)); return true
    end
    local function apply_theme(obj)
        if not obj or not obj:IsA("Instance") then return end
        if obj:IsA("GuiObject") then
            pcall(function() obj.BackgroundColor3 = map_background(obj.BackgroundColor3) end)
            pcall(function() obj.BorderColor3 = RED_D end)
        end
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
            pcall(rename_text, obj); pcall(function() obj.TextColor3 = map_text(obj.TextColor3) end); pcall(function() obj.PlaceholderColor3 = TD end)
        end
        if obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
            local has_logo = try_replace_logo(obj); if not has_logo and obj.Image ~= "" then pcall(function() obj.ImageColor3 = map_image(obj.ImageColor3) end) end
        end
        if obj:IsA("ScrollingFrame") then pcall(function() obj.ScrollBarImageColor3 = RED_T end) end
        if obj:IsA("UIStroke") then pcall(function() obj.Color = RED end) end
        if obj:IsA("UIGradient") then
            pcall(function()
                local ks = obj.Color.Keypoints; local nks = {}
                for i = 1, #ks do nks[i] = ColorSequenceKeypoint.new(ks[i].Time, map_gradient(ks[i].Value)) end
                obj.Color = ColorSequence.new(nks)
            end)
        end
    end
    local function is_ours(gui)
        local n = tostring(gui.Name); return n:find("^KL_") or n:find("^ROX_") or n:find("RedOnix", 1, true) or n:find("RedOnyx", 1, true)
    end
    local function is_native(gui)
        local n = tostring(gui.Name):lower(); return n:find("robloxgui", 1, true) or n:find("playerlist", 1, true) or n:find("players", 1, true) or n:find("notifications", 1, true) or n:find("backpack", 1, true) or n:find("hotbar", 1, true) or n:find("health", 1, true) or n:find("chat", 1, true) or n:find("leaderboard", 1, true) or n:find("topbar", 1, true) or n:find("emotes", 1, true)
    end
    local function is_under_tracked(inst)
        local ok, root = pcall(function() local r = inst; while r and r.Parent and not r:IsA("ScreenGui") do r = r.Parent end; return r end)
        if not ok or not root or not root:IsA("ScreenGui") then return false end; return tracked_guis[root] == true
    end
    local function watch_object(obj)
        if watched_objects[obj] then return end; watched_objects[obj] = true
        pcall(function()
            if obj:IsA("GuiObject") then
                obj.BackgroundColor3Changed:Connect(function() if is_under_tracked(obj) then pcall(function() obj.BackgroundColor3 = map_background(obj.BackgroundColor3) end) end end)
                obj:GetPropertyChangedSignal("BorderColor3"):Connect(function() if is_under_tracked(obj) then pcall(function() obj.BorderColor3 = RED_D end) end end)
            end
            if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
                obj.TextColor3Changed:Connect(function() if is_under_tracked(obj) then pcall(function() obj.TextColor3 = map_text(obj.TextColor3) end) end end)
                obj:GetPropertyChangedSignal("Text"):Connect(function() if is_under_tracked(obj) then pcall(rename_text, obj) end end)
            end
            if obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                obj.ImageColor3Changed:Connect(function() if is_under_tracked(obj) then pcall(function() obj.ImageColor3 = map_image(obj.ImageColor3) end) end end)
                obj.ImageChanged:Connect(function() if is_under_tracked(obj) then pcall(function() if not try_replace_logo(obj) and obj.Image ~= "" then pcall(function() obj.ImageColor3 = map_image(obj.ImageColor3) end) end end) end end)
            end
            if obj:IsA("UIStroke") then obj.ColorChanged:Connect(function() if is_under_tracked(obj) then pcall(function() obj.Color = RED end) end end) end
            if obj:IsA("UIGradient") then
                obj:GetPropertyChangedSignal("Color"):Connect(function() if is_under_tracked(obj) then pcall(function() local ks = obj.Color.Keypoints; local nks = {}; for i = 1, #ks do nks[i] = ColorSequenceKeypoint.new(ks[i].Time, map_gradient(ks[i].Value)) end; obj.Color = ColorSequence.new(nks) end) end end)
            end
        end)
    end
    local function track_gui(gui)
        if not gui or not gui:IsA("ScreenGui") or is_ours(gui) or is_native(gui) or tracked_guis[gui] then return end
        tracked_guis[gui] = true
        pcall(function()
            gui.DescendantAdded:Connect(function(desc) delayF(0.03, function() pcall(function() local function walk(x) pcall(apply_theme, x); pcall(watch_object, x); for _, child in ipairs(x:GetChildren()) do walk(child) end end; walk(desc) end) end) end)
            for _, obj in ipairs(gui:GetDescendants()) do pcall(apply_theme, obj); pcall(watch_object, obj) end
        end)
    end
    local function scan_containers()
        pcall(function()
            for _, container in ipairs(containers) do
                if typeof(container) == "Instance" then
                    for _, gui in ipairs(container:GetChildren()) do
                        if gui:IsA("ScreenGui") and not is_ours(gui) and not is_native(gui) then
                            if not base_guis[gui] then track_gui(gui)
                            else
                                local n = tostring(gui.Name):lower()
                                if n:find("hub", 1, true) or n:find("kaitun", 1, true) or n:find("quantum", 1, true) or n:find("onyx", 1, true) or n:find("^ui") or n:find("lib", 1, true) then track_gui(gui) end
                            end
                        end
                    end
                end
            end
            for gui in pairs(tracked_guis) do if not gui.Parent then tracked_guis[gui] = nil end end
        end)
    end
    scan_containers()
    local start_time = tick()
    spawnF(function()
        while tick() - start_time < 60 do waitF(0.5); scan_containers() end
        local logo_tick = 0
        while true do waitF(2); scan_containers(); logo_tick = logo_tick + 1; if logo_tick % 5 == 0 then pcall(function() for gui in pairs(tracked_guis) do for _, obj in ipairs(gui:GetDescendants()) do try_replace_logo(obj) end end end) end end
    end)
    print("[Red Onyx Hub] Patcher ativo.")
end)

-- ===== INICIALIZACAO =====
local ok_ui, err_ui = pcall(create_key_ui)
if not ok_ui then
    warn("[Red Onyx Hub] Erro na UI: " .. tostring(err_ui))
    pcall(function()
        local e = Instance.new("ScreenGui"); e.Name = "ROX_Error"; e.IgnoreGuiInset = true
        local l = Instance.new("TextLabel"); l.Size = UDim2.new(1, -60, 0, 80); l.Position = UDim2.new(0.5, 0, 0.9, 0); l.AnchorPoint = Vector2.new(0.5, 1); l.BackgroundColor3 = DK; l.BackgroundTransparency = 0.2; l.BorderSizePixel = 0; l.Text = "RED ONYX HUB ERROR:\n" .. tostring(err_ui); l.TextColor3 = RED_T; l.TextSize = 13; l.TextWrapped = true; l.TextXAlignment = Enum.TextXAlignment.Center; l.Parent = e; e.Parent = game:GetService("CoreGui")
    end)
end
print("[Red Onyx Hub] v11 OK")
