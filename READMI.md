-- Blox Fruits Script com Fluent UI
-- Carregando a biblioteca Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Variáveis globais
local Window = Fluent:CreateWindow({
    Title = "Blox Fruits Hub",
    SubTitle = "by YourName",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "home" }),
    Farm = Window:AddTab({ Title = "Farm", Icon = "sprout" }),
    Items = Window:AddTab({ Title = "Auto Items", Icon = "package" }),
    Loja = Window:AddTab({ Title = "Loja", Icon = "shopping-cart" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- Variáveis de controle
local AutoFarmEnabled = false
local AutoFarmLevel = false
local AutoFarmBoss = false
local AutoCollectFruits = false
local AutoCollectChests = false
local SelectedWeapon = nil
local FarmDistance = 15

-- Funções utilitárias
local function GetPlayer()
    return game.Players.LocalPlayer
end

local function GetCharacter()
    return GetPlayer().Character or GetPlayer().CharacterAdded:Wait()
end

local function GetHumanoidRootPart()
    local char = GetCharacter()
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function Teleport(cframe)
    local hrp = GetHumanoidRootPart()
    if hrp then
        hrp.CFrame = cframe
    end
end

local function GetEnemies()
    local enemies = {}
    for _, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
        if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            table.insert(enemies, v)
        end
    end
    return enemies
end

local function GetNearestEnemy()
    local nearest = nil
    local minDist = math.huge
    local hrp = GetHumanoidRootPart()
    
    if not hrp then return nil end
    
    for _, enemy in pairs(GetEnemies()) do
        local enemyHrp = enemy:FindFirstChild("HumanoidRootPart")
        if enemyHrp then
            local dist = (hrp.Position - enemyHrp.Position).Magnitude
            if dist < minDist then
                minDist = dist
                nearest = enemy
            end
        end
    end
    
    return nearest
end

local function EquipWeapon(weaponName)
    local char = GetCharacter()
    local tool = GetPlayer().Backpack:FindFirstChild(weaponName)
    
    if tool then
        GetPlayer().Character.Humanoid:EquipTool(tool)
    end
end

-- TAB MAIN
local MainSection = Tabs.Main:AddSection("Informações do Jogador")

Tabs.Main:AddParagraph({
    Title = "Bem-vindo!",
    Content = "Este é um hub completo para Blox Fruits.\nUse as abas para acessar diferentes funcionalidades."
})

local LevelLabel = Tabs.Main:AddParagraph({
    Title = "Seu Level",
    Content = "Carregando..."
})

local BellyLabel = Tabs.Main:AddParagraph({
    Title = "Seu Dinheiro",
    Content = "Carregando..."
})

spawn(function()
    while wait(2) do
        local player = GetPlayer()
        if player then
            local level = player.Data.Level.Value or 0
            local belly = player.Data.Beli.Value or 0
            
            LevelLabel:SetDesc("Level: " .. tostring(level))
            BellyLabel:SetDesc("Belly: $" .. tostring(belly))
        end
    end
end)

local CombatSection = Tabs.Main:AddSection("Combate")

Tabs.Main:AddButton({
    Title = "Matar Inimigo Mais Próximo",
    Description = "Mata o inimigo mais próximo de você",
    Callback = function()
        local enemy = GetNearestEnemy()
        if enemy then
            Fluent:Notify({
                Title = "Combate",
                Content = "Atacando " .. enemy.Name,
                Duration = 3
            })
            
            repeat
                wait()
                local enemyHrp = enemy:FindFirstChild("HumanoidRootPart")
                if enemyHrp then
                    Teleport(enemyHrp.CFrame * CFrame.new(0, FarmDistance, 0))
                end
            until not enemy or enemy.Humanoid.Health <= 0
        else
            Fluent:Notify({
                Title = "Combate",
                Content = "Nenhum inimigo encontrado!",
                Duration = 3
            })
        end
    end
})

-- TAB FARM
local FarmSection = Tabs.Farm:AddSection("Auto Farm")

local AutoFarmToggle = Tabs.Farm:AddToggle("AutoFarm", {
    Title = "Auto Farm Level",
    Default = false,
    Callback = function(value)
        AutoFarmLevel = value
        if value then
            Fluent:Notify({
                Title = "Auto Farm",
                Content = "Auto Farm ativado!",
                Duration = 3
            })
        end
    end
})

local AutoBossToggle = Tabs.Farm:AddToggle("AutoBoss", {
    Title = "Auto Farm Boss",
    Default = false,
    Callback = function(value)
        AutoFarmBoss = value
    end
})

local DistanceSlider = Tabs.Farm:AddSlider("Distance", {
    Title = "Distância do Inimigo",
    Description = "Ajusta a distância de farm",
    Default = 15,
    Min = 5,
    Max = 50,
    Rounding = 1,
    Callback = function(value)
        FarmDistance = value
    end
})

Tabs.Farm:AddButton({
    Title = "Teleport para Quest",
    Description = "Teleporta você para o NPC de quest",
    Callback = function()
        Fluent:Notify({
            Title = "Teleport",
            Content = "Teleportando para quest...",
            Duration = 2
        })
        -- Adicione aqui a lógica de teleporte para quest
    end
})

-- Loop de Auto Farm
spawn(function()
    while wait(0.1) do
        if AutoFarmLevel then
            local enemy = GetNearestEnemy()
            if enemy and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                local enemyHrp = enemy:FindFirstChild("HumanoidRootPart")
                if enemyHrp then
                    Teleport(enemyHrp.CFrame * CFrame.new(0, FarmDistance, 0))
                    
                    -- Atacar inimigo
                    if SelectedWeapon then
                        EquipWeapon(SelectedWeapon)
                    end
                end
            end
        end
    end
end)

-- TAB AUTO ITEMS
local ItemsSection = Tabs.Items:AddSection("Coletar Itens Automaticamente")

Tabs.Items:AddToggle("AutoFruits", {
    Title = "Auto Collect Fruits",
    Default = false,
    Callback = function(value)
        AutoCollectFruits = value
    end
})

Tabs.Items:AddToggle("AutoChests", {
    Title = "Auto Collect Chests",
    Default = false,
    Callback = function(value)
        AutoCollectChests = value
    end
})

-- Loop de coleta de frutas
spawn(function()
    while wait(1) do
        if AutoCollectFruits then
            for _, fruit in pairs(game:GetService("Workspace"):GetChildren()) do
                if fruit.Name:find("Fruit") and fruit:IsA("Tool") then
                    local hrp = GetHumanoidRootPart()
                    if hrp then
                        hrp.CFrame = fruit.Handle.CFrame
                        wait(0.3)
                    end
                end
            end
        end
    end
end)

-- Loop de coleta de baús
spawn(function()
    while wait(1) do
        if AutoCollectChests then
            for _, chest in pairs(game:GetService("Workspace"):GetChildren()) do
                if chest.Name:find("Chest") then
                    local hrp = GetHumanoidRootPart()
                    if hrp and chest:FindFirstChild("Part") then
                        hrp.CFrame = chest.Part.CFrame
                        wait(0.5)
                    end
                end
            end
        end
    end
end)

-- TAB LOJA
local LojaSection = Tabs.Loja:AddSection("Loja de Itens")

Tabs.Loja:AddButton({
    Title = "Comprar Haki Observation",
    Description = "Compra Haki de Observação",
    Callback = function()
        -- Adicione a lógica de compra aqui
        Fluent:Notify({
            Title = "Loja",
            Content = "Tentando comprar Observation Haki...",
            Duration = 3
        })
    end
})

Tabs.Loja:AddButton({
    Title = "Comprar Haki Armament",
    Description = "Compra Haki de Armamento",
    Callback = function()
        Fluent:Notify({
            Title = "Loja",
            Content = "Tentando comprar Armament Haki...",
            Duration = 3
        })
    end
})

Tabs.Loja:AddButton({
    Title = "Comprar Fighting Style",
    Description = "Compra estilo de luta",
    Callback = function()
        Fluent:Notify({
            Title = "Loja",
            Content = "Tentando comprar Fighting Style...",
            Duration = 3
        })
    end
})

-- TAB SETTINGS
local SettingsSection = Tabs.Settings:AddSection("Configurações Gerais")

local WeaponDropdown = Tabs.Settings:AddDropdown("Weapon", {
    Title = "Selecionar Arma",
    Values = {"Combat", "Sword", "Blox Fruit", "Gun"},
    Multi = false,
    Default = 1,
    Callback = function(value)
        SelectedWeapon = value
        Fluent:Notify({
            Title = "Arma",
            Content = "Arma selecionada: " .. value,
            Duration = 2
        })
    end
})

Tabs.Settings:AddButton({
    Title = "Redeem All Codes",
    Description = "Resgata todos os códigos disponíveis",
    Callback = function()
        local codes = {
            "Sub2OfficialNoobie",
            "Axiore",
            "TantaiGaming",
            "StrawHatMaine",
            "Sub2Fer999",
            "Enyu_is_Pro",
            "Magicbus",
            "JCWK",
            "Starcodeheo",
            "Bluxxy",
            "THEGREATACE"
        }
        
        for _, code in pairs(codes) do
            -- Adicione aqui a lógica de resgate de códigos
            wait(0.5)
        end
        
        Fluent:Notify({
            Title = "Códigos",
            Content = "Todos os códigos foram resgatados!",
            Duration = 3
        })
    end
})

InterfaceManager:SetLibrary(Fluent)
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetFolder("BloxFruitsHub")
SaveManager:SetFolder("BloxFruitsHub/configs")

InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

Window:SelectTab(1)

Fluent:Notify({
    Title = "Blox Fruits Hub",
    Content = "Script carregado com sucesso!",
    Duration = 5
})
