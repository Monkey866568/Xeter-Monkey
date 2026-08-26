-- ══════════════════════════════════════════════════════════════
-- STEAL EGG — FLUENT UI + RARITY FILTER
-- ══════════════════════════════════════════════════════════════

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- ===== ANTI-CHEAT =====
local OldNC = hookmetamethod(game, "_namecall", newcclosure(function(self, ...)
    if getnamecallmethod() == "Kick" then return nil end
    return OldNC(self, ...)
end))
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController() VirtualUser:ClickButton2(Vector2.new())
end)
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

-- ===== MAP DATA =====
local EggSlots = {
    ["Forest"] = {
CFrame.new(605.0,67.6,-326.5), CFrame.new(600.2,67.7,-332.7),
        CFrame.new(593.0,67.7,-332.4), CFrame.new(597.2,67.6,-324.4), CFrame.new(590.6,67.7,-325.6)},
    ["Lake"] = {
        CFrame.new(750.3,67.7,-411.6), CFrame.new(743.7,67.8,-412.8),
        CFrame.new(735.9,67.6,-410.7), CFrame.new(747.9,67.7,-404.8), CFrame.new(740.7,67.6,-404.5)},
    ["Desert"] = {
        CFrame.new(952.8,67.6,-328.5), CFrame.new(957.6,67.6,-322.4),
        CFrame.new(943.2,67.6,-321.4), CFrame.new(945.6,67.8,-328.3), CFrame.new(949.8,67.8,-320.2)},
    ["Jungle"] = {
        CFrame.new(1189.0,67.7,-413.4), CFrame.new(1186.0,67.6,-405.1),
        CFrame.new(1195.6,67.7,-412.2), CFrame.new(1181.2,67.7,-411.2), CFrame.new(1193.2,67.7,-405.3)},
    ["Snow"] = {
        CFrame.new(1488.2,68.1,-318.8), CFrame.new(1492.4,68.1,-310.7),
        CFrame.new(1485.8,68.0,-311.9), CFrame.new(1495.4,68.0,-319.1), CFrame.new(1500.2,68.2,-312.9)},
    ["Volcano"] = {
        CFrame.new(1879.2,67.5,-401.8), CFrame.new(1885.8,67.5,-400.6),
        CFrame.new(1883.4,67.5,-393.8), CFrame.new(1876.2,67.5,-393.5), CFrame.new(1871.4,67.5,-399.7)},
    ["Abyss Ocean"] = {
        CFrame.new(2284.6,67.6,-331.4), CFrame.new(2277.3,67.7,-331.1),
        CFrame.new(2275.0,67.6,-324.3), CFrame.new(2281.6,67.8,-323.1), CFrame.new(2289.4,67.5,-325.2)},
    ["Prehistoric"] = {
        CFrame.new(2805.8,67.8,-400.1), CFrame.new(2810.6,67.8,-393.9),
        CFrame.new(2817.8,67.8,-394.2), CFrame.new(2820.2,67.6,-401.0), CFrame.new(2813.6,67.5,-402.2)},
    ["Cosmic"] = {
        CFrame.new(3387.7,67.6,-329.0), CFrame.new(3395.0,67.6,-329.2),
        CFrame.new(3385.4,67.6,-322.1), CFrame.new(3399.8,67.6,-323.1), CFrame.new(3397.9,72.5,-311.2)},
    ["Cherry Blossom"] = {
        CFrame.new(4034.7,67.7,-398.9), CFrame.new(4024.3,67.7,-393.0),
        CFrame.new(4020.3,67.7,-399.7), CFrame.new(4028.4,67.8,-400.9), CFrame.new(4031.5,68.0,-392.4)},
}
local SafeZone = CFrame.new(515.2, 70.6, -364.3)
local mapOrder = {"Forest","Lake","Desert","Jungle","Snow","Volcano","Abyss Ocean","Prehistoric","Cosmic","Cherry Blossom"}

-- ===== RARITY =====
local RarityList = {"Common","Uncommon","Rare","Epic","Legendary","Divine"}
local RarityColors = {
    Common=Color3.fromRGB(180,180,180), Uncommon=Color3.fromRGB(80,200,80),
    Rare=Color3.fromRGB(60,140,255), Epic=Color3.fromRGB(180,80,255),
    Legendary=Color3.fromRGB(255,180,30), Divine=Color3.fromRGB(255,60,80),
}
local RarityFilter = {Common=true, Uncommon=true, Rare=true, Epic=true, Legendary=true, Divine=true}

-- ===== CONFIG =====
local Config = {Speed = 800, HopWhenDone = true, AutoStart = true, SelectedMap = "All"}
local isRunning = false
local totalEggs = 0
local totalHops = 0
local totalSkipped = 0
local lastRarity = "-"

-- ===== PRO TWEEN (AlignPosition + RunningNoPhysics) =====
local alignPos, alignOri, att0, att1
local function InitAlign()
    if att0 then return end
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    att0 = Instance.new("Attachment") att0.Name = "AP0" att0.Parent = hrp
    att1 = Instance.new("Attachment") att1.Name = "AP1" att1.Parent = hrp
    alignPos = Instance.new("AlignPosition")
    alignPos.Mode = Enum.PositionAlignmentMode.OneAttachment alignPos.Attachment0 = att0
    alignPos.MaxForce = math.huge alignPos.Responsiveness = 200
    alignPos.ReactionForceEnabled = false alignPos.Enabled = false alignPos.Parent = hrp
    alignOri = Instance.new("AlignOrientation")
    alignOri.Mode = Enum.OrientationAlignmentMode.OneAttachment alignOri.Attachment0 = att0
    alignOri.MaxTorque = math.huge alignOri.Responsiveness = 200
    alignOri.ReactionTorqueEnabled = false alignOri.Enabled = false alignOri.Parent = hrp
end

local isTweening = false
local function ProTween(targetCFrame, speed)
    local char = LocalPlayer.Character
    if not char then return false end
    local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
    if not hrp or not hum then return false end
    speed = speed or 800
    InitAlign() if not att0 then return false end
    if not att0.Parent or not att0.Parent:IsDescendantOf(char) then
        att0 = Instance.new("Attachment") att0.Name="AP0" att0.Parent = hrp
        att1 = Instance.new("Attachment") att1.Name="AP1" att1.Parent = hrp
        alignPos.Attachment0 = att0 alignOri.Attachment0 = att0
    end
    isTweening = true
    hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
    for _, p in pairs(char:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end
    att1.CFrame = targetCFrame att1.Parent = workspace
    alignPos.Position = targetCFrame.Position alignPos.Enabled = true
    alignOri.CFrame = targetCFrame alignOri.Enabled = true
    local t0 = tick()
    while isTweening and task.wait(0.05) do
        if (hrp.Position - targetCFrame.Position).Magnitude <= 10 then break end
        alignPos.Position = targetCFrame.Position alignOri.CFrame = targetCFrame
        if tick() - t0 > 30 then hrp.CFrame = targetCFrame break end
        pcall(function()
            if hum:GetState() ~= Enum.HumanoidStateType.RunningNoPhysics then
                hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics) end
            for _, p in pairs(char:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end
        end)
    end
    alignPos.Enabled = false alignOri.Enabled = false
    hrp.CFrame = targetCFrame
    pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    isTweening = false
    return true
end

-- ===== EGG COLLECT WITH RARITY =====
local function GetEggRarity(obj)
    local name = obj.Name:lower()
    for _, rarity in ipairs(RarityList) do
        if name:find(rarity:lower()) then return rarity end
    end
    local part = obj:IsA("BasePart") and obj or obj:FindFirstChildWhichIsA("BasePart")
    if part then
        local c = part.Color
local r, g, b = math.floor(c.R * 255), math.floor(c.G * 255), math.floor(c.B*255)
        if r > 200 and g < 80 and b < 80 then return "Divine" end
        if r > 200 and g > 140 and b < 60 then return "Legendary" end
        if r > 140 and g < 100 and b > 200 then return "Epic" end
        if r < 100 and g > 100 and b > 200 then return "Rare" end
        if r < 100 and g > 180 and b < 100 then return "Uncommon" end
        return "Common"
    end
    return nil
end

local function CollectEggAt(position)
    local char = LocalPlayer.Character
    if not char then return false, nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return false, nil end
    for _, obj in pairs(workspace:GetDescendants()) do
        local name = string.lower(obj.Name)
        if name:find("egg") or name:find("trung") then
            local part = obj:IsA("BasePart") and obj or obj:FindFirstChild("Handle") or obj:FindFirstChildWhichIsA("BasePart")
            if part and (part.Position - position.Position).Magnitude <= 30 then
                local rarity = GetEggRarity(obj)
                if rarity and not RarityFilter[rarity] then return false, rarity end
                pcall(function() firetouchinterest(hrp, part, 0) firetouchinterest(hrp, part, 1) end)
                local handle = obj:FindFirstChild("Handle")
                if handle then pcall(function() handle.CFrame = hrp.CFrame firetouchinterest(hrp, handle, 0) firetouchinterest(hrp, handle, 1) end) end
                local prompt = obj:FindFirstChildWhichIsA("ProximityPrompt")
                if prompt then pcall(fireproximityprompt, prompt) end
                task.wait(0.2)
                return true, rarity or "Common"
            end
        end
    end
    return false, nil
end

local function HopServer()
    pcall(function()
        local sb = ReplicatedStorage:FindFirstChild("__ServerBrowser")
        if not sb then return false end
        for i = math.random(1, 50), 100 do
local servers = sb:InvokeServer(i)
            for _, v in next, servers do
                if tonumber(v["Count"]) < 15 then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, v["Id"])
                    return true
                end
            end
        end
    end)
    return false
end

-- ══════════════════════════════════════════════════════════════
-- FLUENT GUI
-- ══════════════════════════════════════════════════════════════
local gui = LocalPlayer:WaitForChild("PlayerGui")
if gui:FindFirstChild("EggFluent") then gui.EggFluent:Destroy() end
local SG = Instance.new("ScreenGui") SG.Name = "EggFluent" SG.ResetOnSpawn = false SG.Parent = gui

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 320, 0, 440)
Main.Position = UDim2.new(0, 15, 0.5, -220)
Main.BackgroundColor3 = Color3.fromRGB(32, 32, 38)
Main.BorderSizePixel = 0 Main.ClipsDescendants = true Main.Parent = SG
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)

local sh = Instance.new("ImageLabel") sh.Size = UDim2.new(1,18,1,18) sh.Position = UDim2.new(0,-9,0,-9)
sh.BackgroundTransparency = 1 sh.Image = "rbxassetid://6015897843"
sh.ImageColor3 = Color3.fromRGB(0,0,0) sh.ImageTransparency = 0.5
sh.ScaleType = Enum.ScaleType.Slice sh.SliceCenter = Rect.new(49,49,450,450)
sh.ZIndex = -1 sh.Parent = Main

-- Header
local Header = Instance.new("Frame") Header.Size = UDim2.new(1,0,0,44)
Header.BackgroundColor3 = Color3.fromRGB(40,40,50) Header.BorderSizePixel = 0
Header.ZIndex = 5 Header.Parent = Main
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)
local HF = Instance.new("Frame") HF.Size = UDim2.new(1,0,0,12) HF.Position = UDim2.new(0,0,1,-12)
HF.BackgroundColor3 = Color3.fromRGB(40,40,50) HF.BorderSizePixel = 0 HF.ZIndex = 5 HF.Parent = Header

local Ic = Instance.new("TextLabel") Ic.Text = "🥚" Ic.Size = UDim2.new(0,30,1,0)
Ic.Position = UDim2.new(0,10,0,0) Ic.BackgroundTransparency = 1 Ic.TextSize = 18 Ic.ZIndex = 6 Ic.Parent = Header

local Ttl = Instance.new("TextLabel") Ttl.Text = "Steal Egg" Ttl.Size = UDim2.new(0,120,1,0)
Ttl.Position = UDim2.new(0,38,0,0) Ttl.BackgroundTransparency = 1
Ttl.TextColor3 = Color3.fromRGB(240,240,245) Ttl.TextSize = 16
Ttl.Font = Enum.Font.GothamBold Ttl.TextXAlignment = Enum.TextXAlignment.Left Ttl.ZIndex = 6 Ttl.Parent = Header

local St = Instance.new("TextLabel") St.Text = "FLUENT" St.Size = UDim2.new(0,42,0,14)
St.Position = UDim2.new(0,128,0,15) St.BackgroundTransparency = 1
St.TextColor3 = Color3.fromRGB(100,160,255) St.TextSize = 9 St.Font = Enum.Font.GothamBold St.ZIndex = 6 St.Parent = Header

local Cl = Instance.new("TextButton") Cl.Text = "✕" Cl.Size = UDim2.new(0,28,0,28)
Cl.Position = UDim2.new(1,-36,0,8) Cl.BackgroundColor3 = Color3.fromRGB(255,70,70)
Cl.TextColor3 = Color3.new(1,1,1) Cl.TextSize = 13 Cl.Font = Enum.Font.GothamBold
Cl.BorderSizePixel = 0 Cl.ZIndex = 6 Cl.Parent = Header
Instance.new("UICorner", Cl).CornerRadius = UDim.new(0, 7)
Cl.MouseButton1Click:Connect(function() isRunning = false SG:Destroy() end)

local Mn = Instance.new("TextButton") Mn.Text = "—" Mn.Size = UDim2.new(0,28,0,28)
Mn.Position = UDim2.new(1,-68,0,8) Mn.BackgroundColor3 = Color3.fromRGB(60,60,75)
Mn.TextColor3 = Color3.new(1,1,1) Mn.TextSize = 14 Mn.Font = Enum.Font.GothamBold
Mn.BorderSizePixel = 0 Mn.ZIndex = 6 Mn.Parent = Header
Instance.new("UICorner", Mn).CornerRadius = UDim.new(0, 7)
local minimized = false
Mn.MouseButton1Click:Connect(function()
    minimized = not minimized
    TweenService:Create(Main, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {
        Size = minimized and UDim2.new(0,320,0,44) or UDim2.new(0,320,0,440)
    }):Play()
end)

-- Drag
local drag, dragS, startP, dragI
Header.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 and not minimized then
        drag = true dragS = i.Position startP = Main.Position
        i.Changed:Connect(function() if i.UserInputState ==
Enum.UserInputState.End then drag = false end end)
    end
end)
Header.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement then dragI = i end end)
UserInputService.InputChanged:Connect(function(i)
    if i == dragI and drag then
        Main.Position = UDim2.new(startP.X.Scale, startP.X.Offset+(i.Position-dragS).X, startP.Y.Scale, startP.Y.Offset+(i.Position-dragS).Y)
    end
end)

-- ===== TAB SYSTEM =====
local TabBar = Instance.new("Frame") TabBar.Size = UDim2.new(1,0,0,34)
TabBar.Position = UDim2.new(0,0,0,44) TabBar.BackgroundColor3 = Color3.fromRGB(36,36,44)
TabBar.BorderSizePixel = 0 TabBar.ZIndex = 4 TabBar.Parent = Main
local TLL = Instance.new("UIListLayout") TLL.FillDirection = Enum.FillDirection.Horizontal
TLL.SortOrder = Enum.SortOrder.LayoutOrder TLL.Parent = TabBar
local TPd = Instance.new("UIPadding") TPd.PaddingLeft = UDim.new(0,4) TPd.PaddingTop = UDim.new(0,3) TPd.Parent = TabBar

local TabInd = Instance.new("Frame") TabInd.Size = UDim2.new(0,60,0,2)
TabInd.Position = UDim2.new(0,4,1,-2) TabInd.BackgroundColor3 = Color3.fromRGB(100,160,255)
TabInd.BorderSizePixel = 0 TabInd.ZIndex = 5 TabInd.Parent = TabBar
Instance.new("UICorner", TabInd).CornerRadius = UDim.new(0, 1)

local ContentFrame = Instance.new("Frame") ContentFrame.Size = UDim2.new(1,0,1,-82)
ContentFrame.Position = UDim2.new(0,0,0,82) ContentFrame.BackgroundTransparency = 1
ContentFrame.ClipsDescendants = true ContentFrame.ZIndex = 3 ContentFrame.Parent = Main

local tabs = {} local currentTab = nil

local function createTab(name, icon)
    local btn = Instance.new("TextButton")
    btn.Text = (icon or "") .. " " .. name btn.Size = UDim2.new(0, 100, 0, 28)
    btn.BackgroundColor3 = Color3.fromRGB(36,36,44) btn.TextColor3 = Color3.fromRGB(140,140,160)
    btn.TextSize = 11 btn.Font = Enum.Font.GothamMedium btn.BorderSizePixel = 0
    btn.ZIndex = 5 btn.AutoButtonColor = false btn.Parent = TabBar
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    local page = Instance.new("Frame") page.Size = UDim2.new(1,0,0,0)
    page.BackgroundTransparency = 1 page.AutomaticSize = Enum.AutomaticSize.Y
    page.Visible = false page.ZIndex = 3 page.Parent = ContentFrame

    local scroll = Instance.new("ScrollingFrame") scroll.Size = UDim2.new(1,0,1,0)
    scroll.BackgroundTransparency = 1 scroll.ScrollBarThickness = 3
    scroll.ScrollBarImageColor3 = Color3.fromRGB(80,80,100)
    scroll.CanvasSize = UDim2.new(0,0,0,0) scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scroll.BorderSizePixel = 0 scroll.Parent = page

    local pL = Instance.new("UIListLayout") pL.SortOrder = Enum.SortOrder.LayoutOrder
    pL.Padding = UDim.new(0, 4) pL.Parent = scroll
    local pP = Instance.new("UIPadding") pP.PaddingTop = UDim.new(0, 8)
    pP.PaddingLeft = UDim.new(0, 12) pP.PaddingRight = UDim.new(0, 12) pP.Parent = scroll

    tabs[name] = {button = btn, page = scroll}

    btn.MouseButton1Click:Connect(function()
        if currentTab and tabs[currentTab] then
            tabs[currentTab].page.Parent.Visible = false
            tabs[currentTab].button.TextColor3 = Color3.fromRGB(140,140,160)
            tabs[currentTab].button.BackgroundColor3 = Color3.fromRGB(36,36,44)
        end
        page.Visible = true
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.BackgroundColor3 = Color3.fromRGB(45,45,58)
        TweenService:Create(TabInd, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            Position = UDim2.new(0, btn.AbsolutePosition.X - TabBar.AbsolutePosition.X, 1, -2),
            Size = UDim2.new(0, btn.AbsoluteSize.X, 0, 2)
        }):Play()
        currentTab = name
    end)
    return scroll
end

-- ===== WIDGETS =====
local function FLabel(parent, text, color, sz)
    local l = Instance.new("TextLabel") l.Text = text l.Size = UDim2.new(1,0,0,sz or 18)
    l.BackgroundTransparency = 1 l.TextColor3 = color or Color3.fromRGB(200,200,210)
    l.TextSize = sz and (sz-4) or 12 l.Font = Enum.Font.Gotham
    l.TextXAlignment = Enum.TextXAlignment.Left l.Parent = parent return l
end

local function FSection(parent, text)
    local l = Instance.new("TextLabel") l.Text = text:upper() l.Size = UDim2.new(1,0,0,20)
    l.BackgroundTransparency = 1 l.TextColor3 = Color3.fromRGB(90,90,120)
    l.TextSize = 10 l.Font = Enum.Font.GothamBold l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = parent
    local ln = Instance.new("Frame") ln.Size = UDim2.new(1,0,0,1) ln.Position = UDim2.new(0,0,1,-4)
    ln.BackgroundColor3 = Color3.fromRGB(50,50,65) ln.BorderSizePixel = 0 ln.Parent = l
    return l
end

local function FToggle(parent, label, default, accent, cb)
    accent = accent or Color3.fromRGB(100,160,255)
    local card = Instance.new("Frame") card.Size = UDim2.new(1,0,0,36)
    card.BackgroundColor3 = Color3.fromRGB(42,42,52) card.BorderSizePixel = 0 card.Parent = parent
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)
    local lbl = Instance.new("TextLabel") lbl.Text = label
    lbl.Size = UDim2.new(1,-56,1,0) lbl.Position = UDim2.new(0,14,0,0)
    lbl.BackgroundTransparency = 1 lbl.TextColor3 = Color3.fromRGB(220,220,225)
    lbl.TextSize = 12 lbl.Font = Enum.Font.GothamMedium lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = card
    local track = Instance.new("Frame") track.Size = UDim2.new(0,40,0,20)
    track.Position = UDim2.new(1,-50,0.5,-10)
    track.BackgroundColor3 = default and accent or Color3.fromRGB(60,60,75)
    track.BorderSizePixel = 0 track.Parent = track.Parent -- fix: Parent = card first
    track.Parent = card
    Instance.new("UICorner", track).CornerRadius = UDim.new(0, 10)
    local thumb = Instance.new("Frame") thumb.Size = UDim2.new(0,14,0,14)
    thumb.Position = default and UDim2.new(1,-16,0.5,-7) or UDim2.new(0,3,0.5,-7)
    thumb.BackgroundColor3 = Color3.fromRGB(255,255,255) thumb.BorderSizeParent = 0
    thumb.BorderSizePixel = 0 thumb.Parent = track
    Instance.new("UICorner", thumb).CornerRadius = UDim.new(0, 7)
    local on = default
    local function upd()
        TweenService:Create(track, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            BackgroundColor3 = on and accent or Color3.fromRGB(60,60,75)
        }):Play()
        TweenService:Create(thumb, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
            Position = on and UDim2.new(1,-16,0.5,-7) or UDim2.new(0,3,0.5,-7)
        }):Play()
    end
    track.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then on = not on upd() if cb then cb(on) end end
    end)
    return {set = function(v) on = v upd() if cb then cb(v) end end, get = function() return on end}
end

local function FBtn(parent, text, accent, cb)
    accent = accent or Color3.fromRGB(100,160,255)
    local btn = Instance.new("TextButton") btn.Text = text btn.Size = UDim2.new(1,0,0,34)
    btn.BackgroundColor3 = accent btn.TextColor3 = Color3.new(1,1,1) btn.TextSize = 12
    btn.Font = Enum.Font.GothamBold btn.BorderSizeParent = 0
    btn.BorderSizePixel = 0 btn.AutoButtonColor = false btn.Parent = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    btn.MouseButton1Click:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(255,255,255)}):Play()
        TweenService:Create(btn, TweenInfo.new(0.1), {TextColor3 = accent}):Play()
        if cb then cb() end
        task.wait(0.15)
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = accent}):Play()
        TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = Color3.new(1,1,1)}):Play()
    end)
    return btn
end

-- ===== CREATE TABS =====
local tabMain = createTab("Main", "🏠")
local tabFilter = createTab("Filter", "🎯")
local tabSettings = createTab("Settings", "⚙")

-- TAB 1: MAIN
FSection(tabMain, "STATUS")
local LblStatus = FLabel(tabMain, "⏳ Chờ...", Color3.fromRGB(255,255,255), 22)
local LblMap = FLabel(tabMain, "🗺 Map: —")
local LblSlot = FLabel(tabMain, "📍 Slot: 0/5")
local LblCount = FLabel(tabMain, "🥚 Đã nhặt: 0")
local LblSkip = FLabel(tabMain, "⏭ Bỏ qua: 0")
local LblHop = FLabel(tabMain, "🔄 Hop: 0")
local LblLast = FLabel(tabMain, "📦 Lần cuối: —")

FSection(tabMain, "ACTIONS")
local btnStart = FBtn(tabMain, "▶  BẮT ĐẦU", Color3.fromRGB(60,180,80), function() isRunning = true end)
local btnStop = FBtn(tabMain, "⏹  DỪNG LẠI", Color3.fromRGB(220,60,60), function()
    isRunning = false LblStatus.Text = "🛑 Đã dừng!"
end)
FBtn(tabMain, "🔄 HOP SERVER", Color3.fromRGB(100,160,255), function() HopServer() end)
FBtn(tabMain, "🏠 VỀ SAFE ZONE", Color3.fromRGB(80,80,100), function()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then ProTween(SafeZone, 1000) end
end)

-- TAB 2: FILTER (RARITY)
FSection(tabFilter, "CHỌN RARITY NHẮT")
FLabel(tabFilter, "Bật/tắt rarity muốn nhặt:", Color3.fromRGB(140,140,160), 16)

local rarityToggles = {}
for _, rarity in ipairs(RarityList) do
    local accent = RarityColors[rarity]
    local t = FToggle(tabFilter, rarity, RarityFilter[rarity], accent, function(v)
        RarityFilter[rarity] = v
    end)
    rarityToggles[rarity] = t
end

FSection(tabFilter, "QUICK SELECT")
FBtn(tabFilter, "✅ Chọn tất cả", Color3.fromRGB(80,180,80), function()
    for _, r in ipairs(RarityList) do RarityFilter[r] = true rarityToggles[r].set(true) end
end)
FBtn(tabFilter, "❌ Bỏ tất cả", Color3.fromRGB(220,60,60), function()
    for _, r in ipairs(RarityList) do RarityFilter[r] = false rarityToggles[r].set(false) end
end)
FBtn(tabFilter, "⭐ Chỉ Rare+", Color3.fromRGB(60,140,255), function()
    for _, r in ipairs(RarityList) do
        local v = (r == "Rare" or r == "Epic" or r == "Legendary" or r == "Divine")
        RarityFilter[r] = v rarityToggles[r].set(v)
    end
end)
FBtn(tabFilter, "🌟 Chỉ Legendary+Divine",
Color3.fromRGB(255,180,30), function()
    for _, r in ipairs(RarityList) do
        local v = (r == "Legendary" or r == "Divine")
        RarityFilter[r] = v rarityToggles[r].set(v)
    end
end)
FBtn(tabFilter, "🔥 Chỉ Divine", Color3.fromRGB(255,60,80), function()
    for _, r in ipairs(RarityList) do
        local v = (r == "Divine")
        RarityFilter[r] = v rarityToggles[r].set(v)
    end
end)

-- TAB 3: SETTINGS
FSection(tabSettings, "TỐC ĐỘ")
local speedToggles = {}
local speeds = {
    {name = "Chậm (400)", val = 400},
    {name = "Trung bình (600)", val = 600},
    {name = "Nhanh (800)", val = 800},
    {name = "Rất nhanh (1000)", val = 1000},
    {name = "Xả làn (1200)", val = 1200},
}
for i, sp in ipairs(speeds) do
    FToggle(tabSettings, sp.name, sp.val == Config.Speed, nil, function(v)
        if v then Config.Speed = sp.val
            for j, s in ipairs(speeds) do
                if j ~= i and speedToggles[j] then speedToggles[j].set(false) end
            end
        end
    end)
end

FSection(tabSettings, "KHÁC")
FToggle(tabSettings, "Auto Hop khi hết", true, nil, function(v) Config.HopWhenDone = v end)
FToggle(tabSettings, "Tất cả Maps", true, nil, function(v) Config.SelectedMap = v and "All" or mapOrder[1] end)

FSection(tabSettings, "MAP CHỈ ĐỊNH")
local mapOptions = {"All"}
for _, m in ipairs(mapOrder) do table.insert(mapOptions, m) end
-- Simple map selection via buttons
for _, m in ipairs(mapOptions) do
    FBtn(tabSettings, m == "All" and "🗺 Tất cả Maps" or "🗺 " .. m, Color3.fromRGB(55,55,70), function()
        Config.SelectedMap = m == "All" and "All" or m
    end)
end

-- ===== OPEN FIRST TAB =====
tabs["Main"].button.MouseButton1Click:Fire()

-- ===== MAIN LOOP =====
spawn(function()
    task.wait(2)
    while task.wait(0.3) do
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
                if not isRunning then break end
                local slots = EggSlots[mapName]
                if not slots then continue end

                LblMap.Text = "🗺 Map: " .. mapName

                for slotIdx, slotCF in ipairs(slots) do
                    if not isRunning then break end
                    LblSlot.Text = string.format("📍 Slot: %d/%d", slotIdx, #slots)
                    LblStatus.Text = string.format("🚀 %s [%d/%d]", mapName, slotIdx, #slots)

                    ProTween(slotCF * CFrame.new(0,3,0), Config.Speed)
                    task.wait(0.3)

                    local ok, rarity = CollectEggAt(slotCF)
                    if ok then
                        totalEggs = totalEggs + 1
                        LblCount.Text = string.format("🥚 Đã nhặt: %d", totalEggs)
                        LblLast.Text = "📦 Lần cuối: " .. (rarity or "?")
                        LblStatus.Text = "✅ " .. (rarity or "") .. "!"
                        task.wait(0.3)
                    elseif rarity then
                        totalSkipped = totalSkipped + 1
                        LblSkip.Text = string.format("⏭ Bỏ qua: %d (%s)", totalSkipped, rarity)
                        lastRarity = rarity
                    end
                end

                if isRunning then
                    LblStatus.Text = string.format("✅ Xong %s...", mapName)
                    task.wait(0.5)
                end
            end

            if isRunning then
                if Config.HopWhenDone then
                    LblStatus.Text = "🔄 Hết → Hop server..."
                    task.wait(1)
                    if HopServer() then
                        totalHops = totalHops + 1
                        LblHop.Text = string.format("🔄 Hop: %d", totalHops)
                        task.wait(5)
                    else
                        LblStatus.Text = "⚠ Không hop được"
                        task.wait(10)
                    end
                else
                    LblStatus.Text = "✅ Xong! Lặp lại..."
                    task.wait(2)
                end
            end
        end)
    end
end)

print("[STEAL EGG FLUENT] Loaded! Tab Filter → chọn rarity → Bắt đầu")
