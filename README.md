-- ══════════════════════════════════════════════════════════════
-- ROBLOX ANTI-CHEAT BYPASS v3
-- Chống kick, chống detect teleport, speed, fly, noclip
-- ══════════════════════════════════════════════════════════════

-- ===== 1. ANTI-KICK =====
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Hook __namecall để chặn Kick
local OldNameCall
OldNameCall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "Kick" then
        print("[BYPASS] Blocked Kick:", ...)
        return nil
    end
    return OldNameCall(self, ...)
end))

-- ===== 2. ANTI-SPEED DETECT =====
-- Giữ WalkSpeed/JumpPower mỗi frame (chống game force reset)
local RunService = game:GetService("RunService")
local originalWalkSpeed = 16
local originalJumpPower = 50

RunService.Heartbeat:Connect(function()
    pcall(function()
        local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
        if h then
            if h.WalkSpeed ~= originalWalkSpeed then
                h.WalkSpeed = originalWalkSpeed
            end
            if h.JumpPower ~= originalJumpPower then
                h.JumpPower = originalJumpPower
            end
        end
    end)
end)

-- ===== 3. ANTI-NOCLIP DETECT =====
local noclipEnabled = false

RunService.Stepped:Connect(function()
    if noclipEnabled then
        pcall(function()
            local c = LocalPlayer.Character
            if c then
                for _, part in pairs(c:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end
end)

-- ===== 4. ANTI-TELEPORT DETECT =====
-- Dùng CFrame lerp thay vì set thẳng để server không thấy nhảy vị trí

function SafeTeleport(targetCFrame, duration)
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local dist = (targetCFrame.Position - hrp.Position).Magnitude
    local tween = game:GetService("TweenService"):Create(hrp, TweenInfo.new(
        duration or (dist / 1000),
        Enum.EasingStyle.Linear
    ), {CFrame = targetCFrame})

    tween:Play()
    tween.Completed:Wait()
end

-- ===== 5. ANTI-VOID KILL =====
spawn(function()
    while task.wait(0.5) do
        pcall(function()
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if hrp and h and h.Health > 0 then
                if hrp.Position.Y < -500 then
                    hrp.CFrame = CFrame.new(0, 100, 0)
                end
            end
        end)
    end
end)

-- ===== 6. ANTI-AFK =====
local VirtualUser = game:GetService("VirtualUser")
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- ===== 7. ANTI-PUNISHMENT REMOTE =====
-- Chặn các remote có tên kick/ban/punish/detect/security
pcall(function()
    for _, remote in pairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
        if remote:IsA("RemoteEvent") then
            local name = string.lower(remote.Name)
            if name:find("kick") or name:find("ban") or name:find("punish") 
               or name:find("detect") or name:find("security") or name:find("anti") then
                remote.FireServer = function(...)
                    print("[BYPASS] Blocked remote:", remote.Name)
                end
            end
        end
    end
end)

-- ===== 8. ANTI-HUMANOID STATE DETECT =====
spawn(function()
    while task.wait(1) do
        pcall(function()
            local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if h and h.PlatformStand then
                h.PlatformStand = false
            end
        end)
    end
end)

-- ===== PUBLIC API =====
getgenv().Bypass = {
    Noclip = function(v)
        noclipEnabled = v
        if not v then
            pcall(function()
                for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then part.CanCollide = true end
                end
            end)
        end
    end,
    SafeTP = SafeTeleport,
    SetSpeed = function(speed)
        originalWalkSpeed = speed
        pcall(function()
            LocalPlayer.Character.Humanoid.WalkSpeed = speed
        end)
    end,
    SetJump = function(power)
        originalJumpPower = power
        pcall(function()
            LocalPlayer.Character.Humanoid.JumpPower = power
        end)
    end
}

print("[BYPASS] Anti-Cheat Bypass v3 — Loaded!")
print("  Anti-Kick, Anti-Speed Detect, Anti-TP Detect")
print("  Anti-Void, Anti-AFK, Anti-Punish Remote")
print("  Bypass.Noclip(true) | Bypass.SafeTP(CFrame)")
print("  Bypass.SetSpeed(50)  | Bypass.SetJump(100)")
local speed = 300

local eggPosition = Vector3.new(3387.7, 67.6, -329.0)

local safeZonePosition = Vector3.new(515.2, 70.6, -364.3) -- ⚠️ THAY TỌA ĐỘ SAFE ZONE CỦA BẠN VÀO ĐÂY



local player = game.Players.LocalPlayer



-- Hàm kích hoạt Prompt khẩn cấp

local function forceInteractPrompt()

local char = player.Character

if not char or not char:FindFirstChild("HumanoidRootPart") then return end



local hrp = char.HumanoidRootPart

local closestPrompt = nil

local shortestDist = 30



for _, prompt in pairs(workspace:GetDescendants()) do

if prompt:IsA("ProximityPrompt") then

local parentPart = prompt:FindFirstAncestorWhichIsA("BasePart") or prompt.Parent

if parentPart and parentPart:IsA("BasePart") then

local dist = (hrp.Position - parentPart.Position).Magnitude

if dist < shortestDist then

shortestDist = dist

closestPrompt = prompt

end

end

end

end



if closestPrompt then

closestPrompt.Enabled = true

if fireproximityprompt then

fireproximityprompt(closestPrompt)

else

closestPrompt:InputHoldBegin()

task.wait(closestPrompt.HoldDuration)

closestPrompt:InputHoldEnd()

end

end

end



-- Hàm di chuyển tối ưu (không bị khựng)

local function walkTo(humanoid, hrp, destination)

local jumpConn = humanoid.MoveToFinished:Connect(function(reached)

if not reached and (hrp.Position - destination).Magnitude > 4 then

humanoid.Jump = true

end

end)



-- Giảm khoảng cách chấp nhận dừng xuống 4 studs để tránh kẹt

while (hrp.Position - destination).Magnitude > 4 do

if humanoid.Health <= 0 then break end

humanoid:MoveTo(destination)

task.wait(0.15)

end



if jumpConn then jumpConn:Disconnect() end

end



-- Quy trình chính

local function startBotProcess(char)

local humanoid = char:WaitForChild("Humanoid")

local hrp = char:WaitForChild("HumanoidRootPart")



humanoid.WalkSpeed = speed



local speedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()

if humanoid.WalkSpeed ~= speed then humanoid.WalkSpeed = speed end

end)



task.spawn(function()

-- BƯỚC 1: Chạy tới Trứng

print("Đang chạy tới vị trí trứng...")

walkTo(humanoid, hrp, eggPosition)



-- BƯỚC 2: Nhặt trứng phản hồi nhanh (Spam kích hoạt trong 0.3s)

print("Đã tới nơi! Đang nhặt trứng...")

for _ = 1, 3 do

forceInteractPrompt()

task.wait(0.05)

end



-- BƯỚC 3: Rút lui ngay lập tức không chờ đợi

if humanoid.Health > 0 then

print("Rút lui về Safe Zone ngay lập tức!")

walkTo(humanoid, hrp, safeZonePosition)

print("Đã về đến Safe Zone!")

end



if speedConnection then speedConnection:Disconnect() end

end)

end



-- Khởi chạy

if player.Character then startBotProcess(player.Character) end

player.CharacterAdded:Connect(startBotProcess) 
