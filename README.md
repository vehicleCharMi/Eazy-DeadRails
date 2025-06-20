local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Eazy | DeadRails",
   Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
   LoadingTitle = "Eazy | Cheat",
   LoadingSubtitle = "by MRkot",
   Theme = "Ocean", -- Check https://docs.sirius.menu/rayfield/configuration/themes

   ToggleUIKeybind = "Q", -- The keybind to toggle the UI visibility (string like "K" or Enum.KeyCode)

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil, -- Create a custom folder for your hub/game
      FileName = "Big Hub"
   },

   Discord = {
      Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
      Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ ABCD would be ABCD
      RememberJoins = true -- Set this to false to make them join the discord every time they load it up
   },

   KeySystem = false, -- Set this to true to use our key system
   KeySettings = {
      Title = "Untitled",
      Subtitle = "Key System",
      Note = "No method of obtaining the key is provided", -- Use this to tell the user how to get a key
      FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
      SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
      GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
      Key = {"Hello"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
   }
})

local Tab = Window:CreateTab("Tab Player", 4483362458) -- Title, Image

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local noclipConnection
local noclipEnabled = false

local Toggle = Tab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "Toggle1",
    Callback = function(Value)
        noclipEnabled = Value
        if Value then
            -- Включить noclip
            noclipConnection = RunService.Stepped:Connect(function()
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end)
        else
            -- Выключить noclip
            if noclipConnection then
                noclipConnection:Disconnect()
                noclipConnection = nil
            end
            -- Вернуть столкновение
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end,
})


local Tab = Window:CreateTab("Tab ESP", 4483362458) -- Title, Image

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP_Enabled = false
local ESP_Objects = {}

-- Фильтр: Игрок или NPC с Humanoid
local function IsValidCharacter(model)
    return model:FindFirstChild("Humanoid") and model:FindFirstChild("HumanoidRootPart")
end

local function IsPlayer(model)
    return Players:GetPlayerFromCharacter(model) ~= nil
end

local function CreateESP(model)
    if ESP_Objects[model] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false

    local name = Drawing.new("Text")
    name.Size = 13
    name.Outline = true
    name.Center = true

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Transparency = 1

    -- Установим цвет
    if IsPlayer(model) then
        box.Color = Color3.fromRGB(0, 255, 0)
        name.Color = Color3.fromRGB(255, 255, 255)
        tracer.Color = Color3.fromRGB(0, 255, 0)
    else
        box.Color = Color3.fromRGB(255, 0, 0)
        name.Color = Color3.fromRGB(255, 0, 0)
        tracer.Color = Color3.fromRGB(255, 0, 0)
    end

    ESP_Objects[model] = {
        Box = box,
        Name = name,
        Tracer = tracer
    }
end

local function RemoveESP(model)
    if ESP_Objects[model] then
        for _, v in pairs(ESP_Objects[model]) do
            v:Remove()
        end
        ESP_Objects[model] = nil
    end
end

local function UpdateESP()
    for model, drawings in pairs(ESP_Objects) do
        if model and IsValidCharacter(model) and model:FindFirstChild("Humanoid").Health > 0 then
            local hrp = model:FindFirstChild("HumanoidRootPart")
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            if onScreen then
                local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
                local scale = distance / 25
                local height = 3 * scale
                local width = 1.5 * scale

                drawings.Box.Position = Vector2.new(pos.X - width / 2, pos.Y - height / 2)
                drawings.Box.Size = Vector2.new(width, height)
                drawings.Box.Visible = true

                drawings.Name.Text = IsPlayer(model) and Players:GetPlayerFromCharacter(model).Name or model.Name
                drawings.Name.Position = Vector2.new(pos.X, pos.Y - height / 2 - 14)
                drawings.Name.Visible = true

                drawings.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                drawings.Tracer.To = Vector2.new(pos.X, pos.Y)
                drawings.Tracer.Visible = true
            else
                drawings.Box.Visible = false
                drawings.Name.Visible = false
                drawings.Tracer.Visible = false
            end
        else
            RemoveESP(model)
        end
    end
end

-- Поиск всех подходящих моделей
local function ScanForCharacters()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and IsValidCharacter(obj) then
            CreateESP(obj)
        end
    end
end

-- Главный Toggle
local Toggle = Tab:CreateToggle({
    Name = "ESP | Humanoid",
    CurrentValue = false,
    Flag = "Toggle1",
    Callback = function(Value)
        ESP_Enabled = Value
        if Value then
            ScanForCharacters()
            ESPLoop = RunService.RenderStepped:Connect(UpdateESP)
        else
            if ESPLoop then ESPLoop:Disconnect() end
            for model, _ in pairs(ESP_Objects) do
                RemoveESP(model)
            end
        end
    end
})

local ESP_GoldBar_Enabled = false
local GoldBar_ESP_Objects = {}
local GoldBar_Loop

-- Создание ESP-объекта для GoldBar
local function CreateGoldBarESP(part)
    if GoldBar_ESP_Objects[part] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false
    box.Color = Color3.fromRGB(255, 215, 0) -- Золотой

    local name = Drawing.new("Text")
    name.Size = 13
    name.Outline = true
    name.Center = true
    name.Color = Color3.fromRGB(255, 215, 0)
    name.Text = "GoldBar"

    GoldBar_ESP_Objects[part] = {
        Box = box,
        Name = name
    }
end

local function RemoveGoldBarESP(part)
    if GoldBar_ESP_Objects[part] then
        for _, v in pairs(GoldBar_ESP_Objects[part]) do
            v:Remove()
        end
        GoldBar_ESP_Objects[part] = nil
    end
end

local function UpdateGoldBarESP()
    for part, drawings in pairs(GoldBar_ESP_Objects) do
        if part and part:IsDescendantOf(workspace) then
            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                local size = 1.5
                drawings.Box.Position = Vector2.new(pos.X - size * 10, pos.Y - size * 10)
                drawings.Box.Size = Vector2.new(size * 20, size * 20)
                drawings.Box.Visible = true

                drawings.Name.Position = Vector2.new(pos.X, pos.Y - size * 12)
                drawings.Name.Visible = true
            else
                drawings.Box.Visible = false
                drawings.Name.Visible = false
            end
        else
            RemoveGoldBarESP(part)
        end
    end
end

-- Скан объектов в мире
local function ScanForGoldBars()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find("goldbar") then
            CreateGoldBarESP(obj)
        end
    end
end

-- Добавь этот Toggle
Tab:CreateToggle({
   Name = "ESP | GoldBar",
   CurrentValue = false,
   Flag = "GoldESP",
   Callback = function(Value)
      ESP_GoldBar_Enabled = Value
      if Value then
          ScanForGoldBars()
          GoldBar_Loop = RunService.RenderStepped:Connect(UpdateGoldBarESP)
      else
          if GoldBar_Loop then GoldBar_Loop:Disconnect() end
          for part, _ in pairs(GoldBar_ESP_Objects) do
              RemoveGoldBarESP(part)
          end
      end
   end,
})

local ESP_Bond_Enabled = false
local Bond_ESP_Objects = {}
local Bond_Loop

-- Создание ESP для модели Bond
local function CreateBondESP(model)
    if Bond_ESP_Objects[model] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false
    box.Color = Color3.fromRGB(100, 255, 255)

    local name = Drawing.new("Text")
    name.Size = 13
    name.Outline = true
    name.Center = true
    name.Color = Color3.fromRGB(100, 255, 255)
    name.Text = "Bond"

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Transparency = 1
    tracer.Color = Color3.fromRGB(100, 255, 255)

    Bond_ESP_Objects[model] = {
        Box = box,
        Name = name,
        Tracer = tracer
    }
end

local function RemoveBondESP(model)
    if Bond_ESP_Objects[model] then
        for _, v in pairs(Bond_ESP_Objects[model]) do
            v:Remove()
        end
        Bond_ESP_Objects[model] = nil
    end
end

-- Обновление позиции и отрисовка ESP
local function UpdateBondESP()
    for model, drawings in pairs(Bond_ESP_Objects) do
        if model and model:IsDescendantOf(workspace) then
            local part = model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
            if part then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local size = 1.5
                    drawings.Box.Position = Vector2.new(pos.X - size * 10, pos.Y - size * 10)
                    drawings.Box.Size = Vector2.new(size * 20, size * 20)
                    drawings.Box.Visible = true

                    drawings.Name.Position = Vector2.new(pos.X, pos.Y - size * 12)
                    drawings.Name.Visible = true

                    drawings.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    drawings.Tracer.To = Vector2.new(pos.X, pos.Y)
                    drawings.Tracer.Visible = true
                else
                    drawings.Box.Visible = false
                    drawings.Name.Visible = false
                    drawings.Tracer.Visible = false
                end
            else
                RemoveBondESP(model)
            end
        else
            RemoveBondESP(model)
        end
    end
end

-- Поиск моделей Bond
local function ScanForBonds()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name:lower():find("bond") then
            local mainPart = obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")
            if mainPart then
                CreateBondESP(obj)
            end
        end
    end
end

-- Тоггл интерфейса
Tab:CreateToggle({
   Name = "ESP | Bond",
   CurrentValue = false,
   Flag = "BondESP",
   Callback = function(Value)
      ESP_Bond_Enabled = Value
      if Value then
          ScanForBonds()
          Bond_Loop = RunService.RenderStepped:Connect(UpdateBondESP)
      else
          if Bond_Loop then Bond_Loop:Disconnect() end
          for model, _ in pairs(Bond_ESP_Objects) do
              RemoveBondESP(model)
          end
      end
   end,
})

local Tab = Window:CreateTab("Tab | Game", 4483362458) -- Title, Image

local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")

local Brightnight_Connection

Tab:CreateToggle({
   Name = "Full | Brightnight",
   CurrentValue = false,
   Flag = "Brightnight",
   Callback = function(Value)
      if Value then
         -- Включаем постоянную яркость без изменения времени
         Brightnight_Connection = RunService.RenderStepped:Connect(function()
            Lighting.Brightness = 5
            Lighting.FogEnd = 999999
            Lighting.FogStart = 0
            Lighting.GlobalShadows = false
            Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
            Lighting.Ambient = Color3.new(1, 1, 1)
            Lighting.EnvironmentDiffuseScale = 1
            Lighting.EnvironmentSpecularScale = 1
         end)
      else
         if Brightnight_Connection then Brightnight_Connection:Disconnect() end
         -- Возврат к стандартным значениям
         Lighting.Brightness = 2
         Lighting.FogEnd = 1000
         Lighting.FogStart = 0
         Lighting.GlobalShadows = true
         Lighting.OutdoorAmbient = Color3.new(0.5, 0.5, 0.5)
         Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
         Lighting.EnvironmentDiffuseScale = 0.5
         Lighting.EnvironmentSpecularScale = 0.5
      end
   end,
})

local Tab = Window:CreateTab("ESP | Guns", 4483362458) -- Title, Image

local ESP_Rifle_Enabled = false
local Rifle_ESP_Objects = {}
local Rifle_Loop

-- Создание ESP для Rifle
local function CreateRifleESP(model)
    if Rifle_ESP_Objects[model] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false
    box.Color = Color3.fromRGB(255, 165, 0) -- Оранжевый

    local name = Drawing.new("Text")
    name.Size = 13
    name.Outline = true
    name.Center = true
    name.Color = Color3.fromRGB(255, 165, 0)
    name.Text = "Rifle"

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Transparency = 1
    tracer.Color = Color3.fromRGB(255, 165, 0)

    Rifle_ESP_Objects[model] = {
        Box = box,
        Name = name,
        Tracer = tracer
    }
end

-- Удаление ESP
local function RemoveRifleESP(model)
    if Rifle_ESP_Objects[model] then
        for _, v in pairs(Rifle_ESP_Objects[model]) do
            v:Remove()
        end
        Rifle_ESP_Objects[model] = nil
    end
end

-- Обновление ESP
local function UpdateRifleESP()
    for model, drawings in pairs(Rifle_ESP_Objects) do
        if model and model:IsDescendantOf(workspace) then
            local part = model:IsA("Model") and (model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")) or model
            if part then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local size = 1.5
                    drawings.Box.Position = Vector2.new(pos.X - size * 10, pos.Y - size * 10)
                    drawings.Box.Size = Vector2.new(size * 20, size * 20)
                    drawings.Box.Visible = true

                    drawings.Name.Position = Vector2.new(pos.X, pos.Y - size * 12)
                    drawings.Name.Visible = true

                    drawings.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    drawings.Tracer.To = Vector2.new(pos.X, pos.Y)
                    drawings.Tracer.Visible = true
                else
                    drawings.Box.Visible = false
                    drawings.Name.Visible = false
                    drawings.Tracer.Visible = false
                end
            else
                RemoveRifleESP(model)
            end
        else
            RemoveRifleESP(model)
        end
    end
end

-- Поиск Rifle
local function ScanForRifles()
    for _, obj in pairs(workspace:GetDescendants()) do
        if (obj:IsA("Model") or obj:IsA("BasePart")) and obj.Name:lower():find("rifle") then
            local part = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")) or obj
            if part then
                CreateRifleESP(obj)
            end
        end
    end
end

-- Тоггл
Tab:CreateToggle({
   Name = "ESP | Rifle",
   CurrentValue = false,
   Flag = "RifleESP",
   Callback = function(Value)
      ESP_Rifle_Enabled = Value
      if Value then
          ScanForRifles()
          Rifle_Loop = RunService.RenderStepped:Connect(UpdateRifleESP)
      else
          if Rifle_Loop then Rifle_Loop:Disconnect() end
          for model, _ in pairs(Rifle_ESP_Objects) do
              RemoveRifleESP(model)
          end
      end
   end,
})

local ESP_Shotgun_Enabled = false
local Shotgun_ESP_Objects = {}
local Shotgun_Loop

-- Создание ESP для Shotgun
local function CreateShotgunESP(model)
    if Shotgun_ESP_Objects[model] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false
    box.Color = Color3.fromRGB(255, 100, 100) -- Розовато-красный

    local name = Drawing.new("Text")
    name.Size = 13
    name.Outline = true
    name.Center = true
    name.Color = Color3.fromRGB(255, 100, 100)
    name.Text = "ShotGun"

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Transparency = 1
    tracer.Color = Color3.fromRGB(255, 100, 100)

    Shotgun_ESP_Objects[model] = {
        Box = box,
        Name = name,
        Tracer = tracer
    }
end

-- Удаление ESP
local function RemoveShotgunESP(model)
    if Shotgun_ESP_Objects[model] then
        for _, v in pairs(Shotgun_ESP_Objects[model]) do
            v:Remove()
        end
        Shotgun_ESP_Objects[model] = nil
    end
end

-- Обновление ESP
local function UpdateShotgunESP()
    for model, drawings in pairs(Shotgun_ESP_Objects) do
        if model and model:IsDescendantOf(workspace) then
            local part = model:IsA("Model") and (model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")) or model
            if part then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local size = 1.5
                    drawings.Box.Position = Vector2.new(pos.X - size * 10, pos.Y - size * 10)
                    drawings.Box.Size = Vector2.new(size * 20, size * 20)
                    drawings.Box.Visible = true

                    drawings.Name.Position = Vector2.new(pos.X, pos.Y - size * 12)
                    drawings.Name.Visible = true

                    drawings.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    drawings.Tracer.To = Vector2.new(pos.X, pos.Y)
                    drawings.Tracer.Visible = true
                else
                    drawings.Box.Visible = false
                    drawings.Name.Visible = false
                    drawings.Tracer.Visible = false
                end
            else
                RemoveShotgunESP(model)
            end
        else
            RemoveShotgunESP(model)
        end
    end
end

-- Поиск ShotGun
local function ScanForShotguns()
    for _, obj in pairs(workspace:GetDescendants()) do
        if (obj:IsA("Model") or obj:IsA("BasePart")) and obj.Name:lower():find("shotgun") then
            local part = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")) or obj
            if part then
                CreateShotgunESP(obj)
            end
        end
    end
end

-- Тоггл
Tab:CreateToggle({
   Name = "ESP | ShotGun",
   CurrentValue = false,
   Flag = "ShotgunESP",
   Callback = function(Value)
      ESP_Shotgun_Enabled = Value
      if Value then
          ScanForShotguns()
          Shotgun_Loop = RunService.RenderStepped:Connect(UpdateShotgunESP)
      else
          if Shotgun_Loop then Shotgun_Loop:Disconnect() end
          for model, _ in pairs(Shotgun_ESP_Objects) do
              RemoveShotgunESP(model)
          end
      end
   end,
})
	
