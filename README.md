-- ══════════════════════════════════════════════════════════════════════
-- STEAL EGG FLUENT — Dùng Fluent Library (dawid-scripts)
-- AlignPosition + RunningNoPhysics | Tween >800 không giật lùi
-- Dropdown lọc Rarity: Common → Divine
-- ══════════════════════════════════════════════════════════════════════

-- ===== LOAD FLUENT LIBRARY =====
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- ═════════════ ANTI-CHEAT BYPASS ═══════════════

-- Anti-Kick
local OldNC = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
    if getnamecallmethod() == "Kick" then return nil end
    return OldNC(self, ...)
end))

-- Anti-AFK
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- Block punish remotes
pcall(function()
    for _, r in pairs(ReplicatedStorage:GetDescendants()) do
        if r:IsA("RemoteEvent") then
            local n = string.lower(r.Name)
            if n:find("kick") or n:find("ban") or n:find("punish") or n:find("detect") then
                r.FireServer = function() end
            end
        end
    end
end)

-- Anti-Void
spawn(function()
    while task.wait(0.5) do
        pcall(function()
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if hrp and h and h.Health > 0 and hrp.Position.Y < -500 then
                hrp.CFrame = CFrame.new(515.2, 70.6, -364.3)
            end
        end)
    end
end)

-- Anti-Noclip (luôn bật khi tween)
RunService.Stepped:Connect(function()
    pcall(function()
        local char = LocalPlayer.Character
        if char and isTweening then
            for _, p in pairs(char:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end
    end)
end)

-- ═════════════ RARITY SYSTEM ═══════════════
local RarityOrder = {"Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "Divine"}

local function GetRarityIndex(rarity)
    for i, name in ipairs(RarityOrder) do
        if name:lower() == rarity:lower() then return i end
    end
    return 1
end

-- Tìm rarity trong tên object hoặc TextLabel con
local function GetEggRarity(objName)
    local lower = string.lower(objName)
    for _, rarity in ipairs(RarityOrder) do
        if lower:find(rarity:lower()) then
            return rarity
        end
    end
    -- Kiểm tra TextLabel/TextButton con
    for _, d in pairs(workspace:GetDescendants()) do
        if d.Name == objName then
            for _, c in pairs(d:GetDescendants()) do
                if c:IsA("TextLabel") or c:IsA("TextButton") then
                    for _, rarity in ipairs(RarityOrder) do
                        if string.lower(c.Text):find(rarity:lower()) then
                            return rarity
                        end
                    end
                end
            end
            break
        end
    end
    return "Common"
end

-- ═════════════ EGG SLOT COORDINATES (10 MAPS, 50 SLOTS) ═══════════════
local EggSlots = {
    ["Forest"] = {
        CFrame.new(605.0, 67.6, -326.5),
        CFrame.new(600.2, 67.7, -332.7),
        CFrame.new(593.0, 67.7, -332.4),
        CFrame.new(597.2, 67.6, -324.4),
        CFrame.new(590.6, 67.7, -325.6),
    },
    ["Lake"] = {
        CFrame.new(750.3, 67.7, -411.6),
        CFrame.new(743.7, 67.8, -412.8),
        CFrame.new(735.9, 67.6, -410.7),
        CFrame.new(747.9, 67.7, -404.8),
        CFrame.new(740.7, 67.6, -404.5),
    },
    ["Desert"] = {
        CFrame.new(952.8, 67.6, -328.5),
        CFrame.new(957.6, 67.6, -322.4),
        CFrame.new(943.2, 67.6, -321.4),
        CFrame.new(945.6, 67.8, -328.3),
        CFrame.new(949.8, 67.8, -320.2),
    },
    ["Jungle"] = {
        CFrame.new(1189.0, 67.7, -413.4),
        CFrame.new(1186.0, 67.6, -405.1),
        CFrame.new(1195.6, 67.7, -412.2),
        CFrame.new(1181.2, 67.7, -411.2),
        CFrame.new(1193.2, 67.7, -405.3),
    },
    ["Snow"] = {
        CFrame.new(1488.2, 68.1, -318.8),
        CFrame.new(1492.4, 68.1, -310.7),
        CFrame.new(1485.8, 68.0, -311.9),
        CFrame.new(1495.4, 68.0, -319.1),
        CFrame.new(1500.2, 68.2, -312.9),
    },
    ["Volcano"] = {
        CFrame.new(1879.2, 67.5, -401.8),
        CFrame.new(1885.8, 67.5, -400.6),
        CFrame.new(1883.4, 67.5, -393.8),
        CFrame.new(1876.2, 67.5, -393.5),
        CFrame.new(1871.4, 67.5, -399.7),
    },
    ["Abyss Ocean"] = {
        CFrame.new(2284.6, 67.6, -331.4),
        CFrame.new(2277.3, 67.7, -331.1),
        CFrame.new(2275.0, 67.6, -324.3),
        CFrame.new(2281.6, 67.8, -323.1),
        CFrame.new(2289.4, 67.5, -325.2),
    },
    ["Prehistoric"] = {
        CFrame.new(2805.8, 67.8, -400.1),
        CFrame.new(2810.6, 67.8, -393.9),
        CFrame.new(2817.8, 67.8, -394.2),
        CFrame.new(2820.2, 67.6, -401.0),
        CFrame.new(2813.6, 67.5, -402.2),
    },
    ["Cosmic"] = {
        CFrame.new(3387.7, 67.6, -329.0),
        CFrame.new(3395.0, 67.6, -329.2),
        CFrame.new(3385.4, 67.6, -322.1),
        CFrame.new(3399.8, 67.6, -323.1),
        CFrame.new(3397.9, 72.5, -311.2),
    },
    ["Cherry Blossom"] = {
        CFrame.new(4034.7, 67.7, -398.9),
        CFrame.new(4024.3, 67.7, -393.0),
        CFrame.new(4020.3, 67.7, -399.7),
        CFrame.new(4028.4, 67.8, -400.9),
        CFrame.new(4031.5, 68.0, -392.4),
    },
}

local SafeZone = CFrame.new(515.2, 70.6, -364.3)
local mapOrder = {"Forest", "Lake", "Desert", "Jungle", "Snow", "Volcano", "Abyss Ocean", "Prehistoric", "Cosmic", "Cherry Blossom"}

-- ═════════════ PRO TWEEN (AlignPosition + RunningNoPhysics) ═══════════════
local isTweening = false
local alignPos = nil
local alignOri = nil
local att0 = nil
local att1 = nil

local function InitAlign()
    if att0 then return end
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    att0 = Instance.new("Attachment")
    att0.Name = "AP_Att0"
    att0.Parent = hrp

    att1 = Instance.new("Attachment")
    att1.Name = "AP_Att1"
    att1.Parent = hrp

    alignPos = Instance.new("AlignPosition")
    alignPos.Name = "AP_Pos"
    alignPos.Mode = Enum.PositionAlignmentMode.OneAttachment
    alignPos.Attachment0 = att0
    alignPos.MaxForce = math.huge
    alignPos.Responsiveness = 200
    alignPos.ReactionForceEnabled = false
    alignPos.Enabled = false
    alignPos.Parent = hrp

    alignOri = Instance.new("AlignOrientation")
    alignOri.Name = "AP_Ori"
    alignOri.Mode = Enum.OrientationAlignmentMode.OneAttachment
    alignOri.Attachment0 = att0
    alignOri.MaxTorque = math.huge
    alignOri.Responsiveness = 200
    alignOri.ReactionTorqueEnabled = false
    alignOri.Enabled = false
    alignOri.Parent = hrp
end

local function ProTween(targetCFrame, speed)
    local char = LocalPlayer.Character
    if not char then return false end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    if not hrp or not hum then return false end

    speed = speed or 800
    InitAlign()
    if not att0 then return false end

    if not att0.Parent or not att0.Parent:IsDescendantOf(char) then
        att0 = Instance.new("Attachment") att0.Name = "AP_Att0" att0.Parent = hrp
        att1 = Instance.new("Attachment") att1.Name = "AP_Att1" att1.Parent = hrp
        alignPos.Attachment0 = att0
        alignOri.Attachment0 = att0
    end

    isTweening = true
    hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)

    for _, p in pairs(char:GetDescendants()) do
        if p:IsA("BasePart") then p.CanCollide = false end
    end

    att1.CFrame = targetCFrame
    att1.Parent = workspace

    alignPos.Position = targetCFrame.Position
    alignPos.Enabled = true

    alignOri.CFrame = targetCFrame
    alignOri.Enabled = true

    local startTime = tick()
    local maxWait = 30

    while isTweening and task.wait(0.05) do
        local dist = (hrp.Position - targetCFrame.Position).Magnitude
        if dist <= 10 then
            break
        end
        alignPos.Position = targetCFrame.Position
        alignOri.CFrame = targetCFrame

        if tick() - startTime > maxWait then
            hrp.CFrame = targetCFrame
            break
        end

        pcall(function()
            if hum:GetState() ~= Enum.HumanoidStateType.RunningNoPhysics then
                hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
            end
        end)
    end

    alignPos.Enabled = false
    alignOri.Enabled = false
    hrp.CFrame = targetCFrame

    pcall(function()
        hum:ChangeState(Enum.Running)
    end)

    isTweening = false
    return true
end

-- ═════════════ EGG COLLECT WITH RARITY FILTER ═══════════════
local minRarityIndex = 1

local function CollectEggAt(position)
    local char = LocalPlayer.Character
    if not char then return false, nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return false, nil end

    local bestEgg = nil
    local bestDist = 30
    local bestRarity = nil

    for _, obj in pairs(workspace:GetDescendants()) do
        local name = string.lower(obj.Name)
        if name:find("egg") or name:find("trung") then
            local part = obj:IsA("BasePart") and obj
                or obj:FindFirstChild("Handle")
                or obj:FindFirstChildWhichIsA("BasePart")
            if part then
                local dist = (part.Position - position.Position).Magnitude
                if dist <= bestDist then
                    local rarity = GetEggRarity(obj.Name)
                    local rarityIdx = GetRarityIndex(rarity)
                    if rarityIdx >= minRarityIndex then
                        bestDist = dist
                        bestEgg = obj
                        bestRarity = rarity
                    end
                end
            end
        end
    end

    if bestEgg then
        local part = bestEgg:IsA("BasePart") and bestEgg
            or bestEgg:FindFirstChild("Handle")
            or bestEgg:FindFirstChildWhichIsA("BasePart")
        if part then
            pcall(function()
                firetouchinterest(hrp, part, 0)
                firetouchinterest(hrp, part, 1)
            end)
            local handle = bestEgg:FindFirstChild("Handle")
            if handle then
                pcall(function()
                    handle.CFrame = hrp.CFrame
                    firetouchinterest(hrp, handle, 0)
                    firetouchinterest(hrp, handle, 1)
                end)
            end
            local prompt = bestEgg:FindFirstChildWhichIsA("ProximityPrompt")
            if prompt then
                pcall(fireproximityprompt, prompt)
            end
            pcall(function()
                for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                    if r:IsA("RemoteFunction") and string.lower(r.Name):find("collect") then
                        r:InvokeServer(bestEgg)
                        break
                    end
                end
            end)
            task.wait(0.2)
            return true, bestRarity
        end
    end

    return false, nil
end

-- ═════════════ SERVER HOP ═══════════════
local function HopServer()
    pcall(function()
        local sb = game:GetService("ReplicatedStorage"):FindFirstChild("__ServerBrowser")
        if sb and sb:IsA("RemoteFunction") then
            local servers = sb:InvokeServer()
            if type(servers) == "table" then
                for _, s in ipairs(servers) do
                    if type(s) == "table" and (s.Players or 0) < 15 then
                        TeleportService:TeleportToPlaceInstance(game.PlaceId, s.JobId)
                        return true
                    end
                end
            end
        end
    end)
    pcall(function()
        TeleportService:Teleport(game.PlaceId, LocalPlayer)
    end)
    return false
end

-- ═════════════ CONFIG ═══════════════
local Config = {
    Speed = 800,
    HopWhenDone = true,
    SelectedMap = "All",
}
local isRunning = false
local totalEggs = 0
local totalHops = 0

-- ═══════════════════════════════════════════════════════════════
--                  FLUENT LIBRARY GUI
-- ═══════════════════════════════════════════════════════════════

local Window = Fluent:CreateWindow({
    Title = "Steal Egg PRO",
    SubTitle = "AlignPosition + RunningNoPhysics",
    TabWidth = 150,
    Size = UDim2.fromOffset(520, 420),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "home" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
    Info = Window:AddTab({ Title = "Info", Icon = "info" }),
}

local Options = Fluent.Options

-- ════════ TAB: MAIN ════════
Tabs.Main:AddSection("TRANG THAI")
local StatusPara = Tabs.Main:AddParagraph({
    Title = "Trang thai",
    Content = "Dang cho..."
})
local CountPara = Tabs.Main:AddParagraph({
    Title = "Da nhat",
    Content = "0 trung"
})
local MapPara = Tabs.Main:AddParagraph({
    Title = "Map hien tai",
    Content = "---"
})
local SlotPara = Tabs.Main:AddParagraph({
    Title = "Slot",
    Content = "0/5"
})
local HopPara = Tabs.Main:AddParagraph({
    Title = "Da hop",
    Content = "0 lan"
})
local RarityPara = Tabs.Main:AddParagraph({
    Title = "Rarity cuoi",
    Content = "---"
})

Tabs.Main:AddSection("HANH DONG")

Tabs.Main:AddButton({
    Title = "BAT DAU",
    Description = "Bat dau nhap trung tu dong",
    Callback = function()
        isRunning = true
        StatusPara:SetDesc("Dang chay...")
        Fluent:Notify({ Title = "Steal Egg", Content = "Bat dau nhap trung!", Duration = 3 })
    end
})

Tabs.Main:AddButton({
    Title = "DUNG LAI",
    Description = "Dung nhap trung",
    Callback = function()
        isRunning = false
        StatusPara:SetDesc("Da dung!")
        Fluent:Notify({ Title = "Steal Egg", Content = "Da dung lai.", Duration = 3 })
    end
})

Tabs.Main:AddButton({
    Title = "HOP SERVER",
    Description = "Chuyen sang server khac ngay lap tuc",
    Callback = function()
        HopServer()
    end
})

Tabs.Main:AddButton({
    Title = "VE SAFE ZONE",
    Description = "Bay ve khu vuc an toan",
    Callback = function()
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then ProTween(SafeZone, 1000) end
    end
})

-- ════════ TAB: SETTINGS ════════
Tabs.Settings:AddSection("RARITY FILTER")

Tabs.Settings:AddDropdown("MinRarity", {
    Title = "Muc tieu toi thieu",
    Description = "Chi nhat trung co rarity >= gia tri nay",
    Values = RarityOrder,
    Multi = false,
    Default = 1,
    Callback = function(Value)
        minRarityIndex = GetRarityIndex(Value)
        Fluent:Notify({
            Title = "Rarity Filter",
            Content = "Chi nhat tu: " .. Value .. " (" .. minRarityIndex .. "/7)",
            Duration = 3
        })
    end
})

Tabs.Settings:AddSection("CAI DAT CHUNG")

Tabs.Settings:AddToggle("AutoHop", {
    Title = "Auto Hop khi het slot",
    Description = "Tu dong chuyen server khi quet xong tat ca",
    Default = true,
    Callback = function(Value)
        Config.HopWhenDone = Value
    end
})

Tabs.Settings:AddToggle("AllMaps", {
    Title = "Tat ca Maps",
    Description = "Quet toan bo 10 maps (50 slots)",
    Default = true,
    Callback = function(Value)
        Config.SelectedMap = Value and "All" or mapOrder[1]
    end
})

Tabs.Settings:AddSlider("TweenSpeed", {
    Title = "Toc do Tween",
    Description = "Studs/giay (khuyen nghi 500-1200)",
    Default = 800,
    Min = 200,
    Max = 2000,
    Rounding = 0,
    Callback = function(Value)
        Config.Speed = Value
    end
})

Tabs.Settings:AddSection("MAP CHON RIENG")

Tabs.Settings:AddDropdown("SelectedMap", {
    Title = "Chon Map cu the",
    Description = "Chi co hieu luc khi Tat ca Maps = OFF",
    Values = mapOrder,
    Multi = false,
    Default = 1,
    Callback = function(Value)
        if not Options.AllMaps.Value then
            Config.SelectedMap = Value
        end
    end
})

-- ════════ TAB: INFO ════════
Tabs.Info:AddSection("THONG TIN SCRIPT")

Tabs.Info:AddParagraph({
    Title = "Script",
    Content = "Steal Egg PRO - Fluent UI"
})
Tabs.Info:AddParagraph({
    Title = "Tween System",
    Content = "AlignPosition + RunningNoPhysics"
})
Tabs.Info:AddParagraph({
    Title = "Toc do mac dinh",
    Content = "800 studs/s - Khong giat lui"
})
Tabs.Info:AddParagraph({
    Title = "Maps",
    Content = "10 maps - 50 slots"
})
Tabs.Info:AddParagraph({
    Title = "Rarity",
    Content = "Common > Uncommon > Rare > Epic > Legendary > Mythic > Divine"
})
Tabs.Info:AddParagraph({
    Title = "Library",
    Content = "Fluent by dawid-scripts"
})

Tabs.Info:AddSection("HUONG DAN SU DUNG")

Tabs.Info:AddParagraph({
    Title = "Bước 1",
    Content = "Tab Settings > Chon Rarity toi thieu (Dropdown)"
})
Tabs.Info:AddParagraph({
    Title = "Bước 2",
    Content = "Tab Settings > Cai dat toc do, auto hop, map"
})
Tabs.Info:AddParagraph({
    Title = "Bước 3",
    Content = "Tab Main > BAM BAT DAU"
})
Tabs.Info:AddParagraph({
    Title = "Ghi chu",
    Content = "Chon Legendary = chi nhat Legendary, Mythic, Divine"
})
Tabs.Info:AddParagraph({
    Title = "Minimize",
    Content = "Nhan LeftControl de an/hien cua so"
})

-- ═══════════════════════════════════════════════════════════════
--                    MAIN LOOP
-- ═══════════════════════════════════════════════════════════════

spawn(function()
    task.wait(2)

    Fluent:Notify({
        Title = "Steal Egg PRO",
        Content = "Loaded! Dung Fluent Library " .. Fluent.Version,
        SubContent = "AlignPosition + RunningNoPhysics",
        Duration = 5
    })

    while task.wait(0.3) do
        if Fluent.Unloaded then break end
        if not isRunning then continue end

        pcall(function()
            local char = LocalPlayer.Character
            if not char or not char:FindFirstChild("HumanoidRootPart") then continue end

            local mapsToScan = {}
            if Config.SelectedMap == "All" then
                mapsToScan = mapOrder
            else
                mapsToScan = {Config.SelectedMap}
            end

            for _, mapName in ipairs(mapsToScan) do
                if Fluent.Unloaded or not isRunning then break end

                local slots = EggSlots[mapName]
                if not slots then continue end

                MapPara:SetDesc(mapName)

                for slotIdx, slotCFrame in ipairs(slots) do
                    if Fluent.Unloaded or not isRunning then break end

                    SlotPara:SetDesc(string.format("%d/%d", slotIdx, #slots))
                    StatusPara:SetDesc(string.format("Bay -> %s [%d/%d]", mapName, slotIdx, #slots))

                    ProTween(slotCFrame * CFrame.new(0, 3, 0), Config.Speed)
                    task.wait(0.3)

                    local collected, rarity = CollectEggAt(slotCFrame)
                    if collected then
                        totalEggs = totalEggs + 1
                        local rarityName = rarity or "?"
                        CountPara:SetDesc(totalEggs .. " trung")
                        RarityPara:SetDesc(rarityName)
                        StatusPara:SetDesc("Nhat duoc [" .. rarityName .. "]!")
                        task.wait(0.3)
                    else
                        RarityPara:SetDesc("Khong co phu hop")
                    end
                end

                if isRunning and not Fluent.Unloaded then
                    StatusPara:SetDesc(string.format("Xong %s -> Map tiep...", mapName))
                    task.wait(0.5)
                end
            end

            if isRunning and not Fluent.Unloaded then
                if Config.HopWhenDone then
                    StatusPara:SetDesc("Het tat ca -> Hop server...")
                    task.wait(1)
                    HopPara:SetDesc(totalHops + 1 .. " lan")
                    if HopServer() then
                        totalHops = totalHops + 1
                        task.wait(5)
                    else
                        StatusPara:SetDesc("Khong hop duoc, thu lai...")
                        task.wait(10)
                    end
                else
                    StatusPara:SetDesc("Xong tat ca! Lap lai...")
                    task.wait(2)
                end
            end
        end)
    end
end)

print("[STEAL EGG FLUENT] Loaded! Fluent Library v" .. Fluent.Version)
print("[STEAL EGG FLUENT] Speed: " .. Config.Speed .. " studs/s | Maps: 10 | Slots: 50")
print("[STEAL EGG FLUENT] AlignPosition + RunningNoPhysics = No rubber-band!"
