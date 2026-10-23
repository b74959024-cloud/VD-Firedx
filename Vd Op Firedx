-- =====================================================
-- 🔥 FIRED HUB by Lafarzen — Violence District
-- Rebuild dari source user (tanpa lib eksternal)
-- =====================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CollectionService = game:GetService("CollectionService")
local CoreGui = game:GetService("CoreGui")

local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ===== CONFIG GLOBAL =====
_G.Vc = _G.Vc or {}
_G.Vc.ScriptAlive = true

_G.Vc.AimConfig = _G.Vc.AimConfig or {
    SPEAR_AuraSpeed = 165,
    SPEAR_AuraGravity = 96.5,
    SPEAR_Speed = 165,
    SPEAR_Gravity = 103,
    SPEAR_MaxDist = 1000,
    Veil_LeadMultiplier = 1.4,
    Veil_FOV = 120,
    Veil_ShowFOV = true,
    Aim_SilentVeilV2 = false,
    Flash_Silent = false,
    Flash_YOffset = 0,
}
_G.Vc.Config = _G.Vc.Config or {
    SpearSmart_enable = true,
    Surv_PerfectVault = false,
    Surv_Aimbot_Enabled = false,
    Surv_Aimbot_ShowFOV = false,
    Surv_Aimbot_Radius = 120,
    Surv_Aimbot_Predict = 0.2,
    Surv_Aimbot_Smoothness = 0.25,
    Killer_Aimbot_Enabled = false,
    Killer_Aimbot_MaxDist = 500,
    Killer_Aimbot_Smoothness = 0.25,
}
_G.Vc.CounterParry = _G.Vc.CounterParry or {
    Enabled = false, Thread = nil, Track = nil, Interval = 0.1,
    Distance = 12, AnimId = "rbxassetid://136962284480779",
}
_G.Vc.AutoStalk = _G.Vc.AutoStalk or { Enabled = false, Conn = nil, LastFire = 0, Range = 50 }
_G.Vc.Moonwalk = _G.Vc.Moonwalk or { Enabled = false, Gui = nil, GuiVisible = false, IsActive = false }
_G.Vc.Speed = _G.Vc.Speed or { Enabled = false, Value = 40 }
_G.Vc.StretchRes = _G.Vc.StretchRes or { Enabled = false, Amount = 1.15 }
_G.Vc.CameraFOV = _G.Vc.CameraFOV or { Enabled = false, Value = 70 }
_G.Vc.InfinityZoom = _G.Vc.InfinityZoom or { Enabled = false }
_G.Vc.InstantTPGate = _G.Vc.InstantTPGate or { Enabled = false }
_G.Vc.ThirdPerson = _G.Vc.ThirdPerson or { Enabled = false }
_G.Vc.UnlimitedVaultEnabled = false
_G.Vc.FlowstateNoCD = _G.Vc.FlowstateNoCD or { Enabled = false, Connection = nil, CharAddedConn = nil }
_G.Vc.NoFall = _G.Vc.NoFall or { Enabled = false }
_G.Vc.BypassGate = _G.Vc.BypassGate or { Enabled = false }
_G.Vc.SelfHeal = _G.Vc.SelfHeal or { Enabled = false }
_G.Vc.NextKiller = _G.Vc.NextKiller or { Enabled = false, IndicatorGui = nil }
_G.Vc.MapPredict = _G.Vc.MapPredict or { Enabled = false }
_G.Vc.FakePerks = _G.Vc.FakePerks or {
    QuickRecoveryEnabled = false, PerfectLandingEnabled = false,
    FlowstateEnabled = false, FakeAdrenalineRush = false,
    boostActive = false, perfectLandingBoostActive = false,
    PerkCooldown = 10, lastQRTime = 0, lastPLTime = 0, lastFSTime = 0,
    lastARTime = 0, fsOnCooldown = false,
    adrenalineConns = {}, qrConnection = nil, plConnection = nil,
    fsConnection = nil, fsAnimConnection = nil,
}
_G.Vc.StreamerMode = _G.Vc.StreamerMode or { Enabled = false, SpooferConns = {} }
_G.Vc.GodMode = false
_G.Vc.InfiniteLungeEnabled = false

_G.Vc.ESPConfig = _G.Vc.ESPConfig or {
    Master = false, Player = true, Killer = true, Status = true,
    HookCount = true, ItemIcon = true, Name = true, Distance = true,
    Outline = false, Generator = true, GeneratorName = true,
    Pallet = true, Hook = true, Window = true, Gate = true,
    SCP = true, KillerWarn = true, MaxDistance = 500,
}
_G.Vc.ESPColors = _G.Vc.ESPColors or {
    Player = Color3.fromRGB(0, 170, 255),
    Killer = Color3.fromRGB(255, 0, 0),
    SCP = Color3.fromRGB(255, 0, 255),
    Generator = Color3.fromRGB(255, 128, 0),
    GeneratorDone = Color3.fromRGB(0, 255, 0),
    Pallet = Color3.fromRGB(255, 255, 0),
    Hook = Color3.fromRGB(0, 255, 255),
    Window = Color3.fromRGB(255, 255, 255),
    Gate = Color3.fromRGB(128, 255, 128),
    Injured = Color3.fromRGB(255, 140, 0),
    Downed = Color3.fromRGB(255, 40, 40),
    Hooked = Color3.fromRGB(240, 240, 255),
}
_G.Vc.ParryConfig = _G.Vc.ParryConfig or { ParryCircle = false, AutoParry = false, ParryRadius = 10, ParryAggressive = false }
_G.Vc.State = _G.Vc.State or { AutoParryAdornment = nil, ParryCooldown = false, ThirdPersonConn = nil }

-- ===== HELPER =====
local function GetRole()
    local t = LP.Team
    return t and t.Name or "Survivor"
end
local function GetDistance(a, b) return (a - b).Magnitude end
local function IsDowned(char)
    if not char then return false end
    local h = char:FindFirstChildOfClass("Humanoid")
    return (h and h.Health <= 0) or char:GetAttribute("Knocked") == true
end

function _G.getClosestSurvivor()
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    local best, bd = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local t = plr.Team and plr.Team.Name
            if t == "Survivors" then
                local p = plr.Character:FindFirstChild("HumanoidRootPart")
                if p then
                    local d = (p.Position - hrp.Position).Magnitude
                    if d < bd and d <= _G.Vc.AimConfig.SPEAR_MaxDist then
                        bd = d; best = p
                    end
                end
            end
        end
    end
    return best
end

local function solveTrajectory(speed, gravity, x, y)
    local g, v = gravity, speed
    local disc = (v^4) - g * (g * x * x + 2 * y * v * v)
    if disc < 0 then return nil, nil end
    local root = math.sqrt(disc)
    local aLow = math.atan((v * v - root) / (g * x))
    local aHigh = math.atan((v * v + root) / (g * x))
    local t1 = x / (v * math.cos(aLow))
    local t2 = x / (v * math.cos(aHigh))
    if t1 < t2 then return aLow, t1 else return aHigh, t2 end
end

-- ===== NOTIFY =====
local function notify(title, desc)
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = title, Text = desc, Duration = 2,
        })
    end)
end

-- ===== SPEAR INTERCEPTOR =====
local flag2 = false
local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    if getnamecallmethod() == "FireServer" and not flag2 then
        pcall(function()
            local char = LP.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp or not char:GetAttribute("spearmode") then return end
            if not _G.Vc.AimConfig.Aim_SilentVeilV2 then return end
            local target = _G.getClosestSurvivor()
            if not target then return end
            local pos = target.Position
            local myPos = hrp.Position
            local delta = pos - myPos
            local dxz = Vector3.new(delta.X, 0, delta.Z).Magnitude
            if dxz < 0.1 then return end
            local speed = _G.Vc.AimConfig.SPEAR_AuraSpeed
            local gravity = _G.Vc.AimConfig.SPEAR_AuraGravity
            local lead = _G.Vc.AimConfig.Veil_LeadMultiplier
            local angle, t = solveTrajectory(speed, gravity, dxz, delta.Y)
            if not angle then return end
            local hum = target.Parent and target.Parent:FindFirstChildOfClass("Humanoid")
            local vel = Vector3.zero
            if hum and hum.MoveDirection.Magnitude > 0 then
                vel = hum.MoveDirection * hum.WalkSpeed
            elseif target:IsA("BasePart") then
                vel = target.AssemblyLinearVelocity
            end
            local vxz = Vector3.new(vel.X, 0, vel.Z)
            local pred = pos + vxz * (t * lead)
            local nd = pred - myPos
            local ndxz = Vector3.new(nd.X, 0, nd.Z).Magnitude
            local na = select(1, solveTrajectory(speed, gravity, math.max(ndxz, 0.1), nd.Y))
            local dir
            if ndxz > 0.001 then
                dir = Vector3.new(nd.X, 0, nd.Z).Unit * math.cos(na) + Vector3.new(0, math.sin(na), 0)
            else
                dir = nd.Unit
            end
            flag2 = true
            self:FireServer(dir, speed, myPos)
            flag2 = false
        end)
    end
    return oldNamecall(self, ...)
end)
setreadonly(oldNamecall, true)

-- ===== FOV + HIGHLIGHT + TRACKER =====
local fovGui = Instance.new("ScreenGui")
fovGui.Name = "FiredHub_FOV"; fovGui.ResetOnSpawn = false; fovGui.IgnoreGuiInset = true; fovGui.Parent = CoreGui
local fovFrame = Instance.new("Frame")
fovFrame.BackgroundTransparency = 1; fovFrame.AnchorPoint = Vector2.new(0.5, 0.5)
fovFrame.Position = UDim2.new(0.5, 0, 0.5, 0); fovFrame.Visible = false; fovFrame.Parent = fovGui
Instance.new("UICorner", fovFrame).CornerRadius = UDim.new(1, 0)
local fovStroke = Instance.new("UIStroke", fovFrame)
fovStroke.Color = Color3.fromRGB(255, 80, 80); fovStroke.Thickness = 1.5

local veilHighlight = Instance.new("Highlight")
veilHighlight.FillColor = Color3.fromRGB(255, 0, 0)
veilHighlight.OutlineColor = Color3.fromRGB(255, 255, 255)
veilHighlight.FillTransparency = 0.5

local trackerPart = nil
local function updateTracker(from, to)
    if from and to and (to - from).Magnitude > 0.2 then
        if not trackerPart then
            trackerPart = Instance.new("Part")
            trackerPart.Name = "FiredHub_Tracker"; trackerPart.Anchored = true
            trackerPart.CanCollide = false; trackerPart.CanTouch = false
            trackerPart.CanQuery = false; trackerPart.CastShadow = false
            trackerPart.Material = Enum.Material.Neon
            trackerPart.Color = Color3.fromRGB(255, 80, 80); trackerPart.Parent = workspace
        end
        local mag = (to - from).Magnitude
        trackerPart.Size = Vector3.new(0.08, 0.08, mag)
        trackerPart.CFrame = CFrame.new((from + to) / 2, to)
        trackerPart.Transparency = 0.2
    elseif trackerPart then
        trackerPart.Transparency = 1
    end
end

-- ===== MOONWALK =====
function _G.StartMoonwalk()
    local c = LP.Character
    local hrp = c and c:FindFirstChild("HumanoidRootPart")
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end
    hum.AutoRotate = false
    _G.Vc.Moonwalk.Enabled = true
end
function _G.StopMoonwalk()
    _G.Vc.Moonwalk.Enabled = false
    local c = LP.Character
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then hum.AutoRotate = true end
end
function _G.ToggleMoonwalkStatus()
    if _G.Vc.Moonwalk.Enabled then _G.StopMoonwalk() else _G.StartMoonwalk() end
    notify("Moonwalk", _G.Vc.Moonwalk.Enabled and "AKTIF" or "NONAKTIF")
end

-- ===== INFINITE LUNGE =====
function _G.EnableInfiniteLunge()
    _G.Vc.InfiniteLungeEnabled = true
    local c = LP.Character
    if c then
        c:SetAttribute("lungeboost", 999)
        if _G.Vc.InfLungeAttrConn then _G.Vc.InfLungeAttrConn:Disconnect() end
        _G.Vc.InfLungeAttrConn = c:GetAttributeChangedSignal("lungeboost"):Connect(function()
            if _G.Vc.InfiniteLungeEnabled and c:GetAttribute("lungeboost") ~= 999 then
                c:SetAttribute("lungeboost", 999)
            end
        end)
    end
    notify("Infinite Lunge", "AKTIF (999x)")
end
function _G.DisableInfiniteLunge()
    _G.Vc.InfiniteLungeEnabled = false
    if _G.Vc.InfLungeAttrConn then _G.Vc.InfLungeAttrConn:Disconnect(); _G.Vc.InfLungeAttrConn = nil end
    local c = LP.Character
    if c then c:SetAttribute("lungeboost", 1) end
    notify("Infinite Lunge", "NONAKTIF")
end

-- ===== COUNTER PARRY =====
function _G.StopCounterParry()
    _G.Vc.CounterParry.Enabled = false
    if _G.Vc.CounterParry.Thread then task.cancel(_G.Vc.CounterParry.Thread); _G.Vc.CounterParry.Thread = nil end
    if _G.Vc.CounterParry.Track then pcall(function() _G.Vc.CounterParry.Track:Stop() end); _G.Vc.CounterParry.Track = nil end
end
function _G.StartCounterParry()
    if _G.Vc.CounterParry.Thread then return end
    _G.Vc.CounterParry.Thread = task.spawn(function()
        while _G.Vc.CounterParry.Enabled and _G.Vc.ScriptAlive do
            local c = LP.Character
            if c and c.Parent and GetRole() == "Killer" then
                local hum = c:FindFirstChildOfClass("Humanoid")
                local animator = hum and hum:FindFirstChildOfClass("Animator")
                local hrp = c:FindFirstChild("HumanoidRootPart")
                if animator and hrp then
                    if not _G.Vc.CounterParry.Track or not _G.Vc.CounterParry.Track.Parent then
                        local anim = Instance.new("Animation")
                        anim.AnimationId = _G.Vc.CounterParry.AnimId
                        local ok, track = pcall(function() return animator:LoadAnimation(anim) end)
                        if ok and track then _G.Vc.CounterParry.Track = track end
                    end
                    local near = false
                    for _, p in ipairs(Players:GetPlayers()) do
                        if p ~= LP and p.Team and p.Team.Name == "Survivors" and p.Character then
                            local phrp = p.Character:FindFirstChild("HumanoidRootPart")
                            if phrp and (hrp.Position - phrp.Position).Magnitude <= _G.Vc.CounterParry.Distance then
                                near = true; break
                            end
                        end
                    end
                    if near and _G.Vc.CounterParry.Track then
                        pcall(function()
                            _G.Vc.CounterParry.Track:Play()
                            _G.Vc.CounterParry.Track:AdjustWeight(0)
                        end)
                        task.wait(0.05)
                        pcall(function() _G.Vc.CounterParry.Track:Stop() end)
                    end
                end
            end
            task.wait(_G.Vc.CounterParry.Interval)
        end
        _G.Vc.CounterParry.Thread = nil
    end)
end
function _G.ToggleCounterParry(on)
    if on then
        if GetRole() ~= "Killer" then notify("Counter Auto Parry", "Khusus Killer!"); return false end
        _G.Vc.CounterParry.Enabled = true
        _G.StartCounterParry()
        notify("Counter Auto Parry", "AKTIF")
    else
        _G.StopCounterParry()
        notify("Counter Auto Parry", "NONAKTIF")
    end
end

-- ===== AUTO STALK =====
function _G.StopAutoStalk()
    _G.Vc.AutoStalk.Enabled = false
    if _G.Vc.AutoStalk.Conn then _G.Vc.AutoStalk.Conn:Disconnect(); _G.Vc.AutoStalk.Conn = nil end
end
function _G.StartAutoStalk()
    if _G.Vc.AutoStalk.Conn then return end
    _G.Vc.AutoStalk.Conn = RunService.Heartbeat:Connect(function()
        if not _G.Vc.AutoStalk.Enabled or not _G.Vc.ScriptAlive then return end
        if GetRole() ~= "Killer" then return end
        if tick() - _G.Vc.AutoStalk.LastFire < 0.35 then return end
        local c = LP.Character
        local hrp = c and c:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local best, bd = nil, math.huge
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP and p.Character then
                local h = p.Character:FindFirstChildOfClass("Humanoid")
                local phrp = p.Character:FindFirstChild("HumanoidRootPart")
                if h and phrp and h.Health > 30 then
                    local d = (phrp.Position - hrp.Position).Magnitude
                    if d <= _G.Vc.AutoStalk.Range and d < bd then bd = d; best = p end
                end
            end
        end
        if not best then return end
        local remotes = ReplicatedStorage:FindFirstChild("Remotes")
        local killers = remotes and remotes:FindFirstChild("Killers")
        local stalker = killers and killers:FindFirstChild("Stalker")
        local start = stalker and stalker:FindFirstChild("StartStalking")
        if start then
            _G.Vc.AutoStalk.LastFire = tick()
            pcall(function() start:FireServer(best) end)
        end
    end)
end
function _G.ToggleAutoStalk(on)
    if on then
        if GetRole() ~= "Killer" then notify("Auto Stalk", "Khusus Killer!"); return false end
        _G.Vc.AutoStalk.Enabled = true; _G.StartAutoStalk(); notify("Auto Stalk", "AKTIF")
    else
        _G.StopAutoStalk(); notify("Auto Stalk", "NONAKTIF")
    end
end

-- ===== FAKE PERKS =====
local function isSurvivor()
    local t = LP.Team
    return t and t.Name == "Survivors"
end
local function hasQuickRecoveryAnim(char)
    local h = char:FindFirstChildOfClass("Humanoid")
    local a = h and h:FindFirstChildOfClass("Animator")
    if not a then return false end
    for _, t in ipairs(a:GetPlayingAnimationTracks()) do
        if t.Animation and string.find(tostring(t.Animation.AnimationId), "126081405469607") then return true end
    end
    return false
end
local function qrBoost(char)
    local F = _G.Vc.FakePerks
    if not F.QuickRecoveryEnabled or not isSurvivor() or F.boostActive then return end
    if tick() - F.lastQRTime < F.PerkCooldown then return end
    F.boostActive = true; F.lastQRTime = tick()
    local now = tick()
    task.spawn(function()
        while F.QuickRecoveryEnabled and tick() - now < 3 do
            if char and char.Parent then char:SetAttribute("speedboost", 1.4) end
            task.wait()
        end
        if char and char.Parent then char:SetAttribute("speedboost", 1) end
        F.boostActive = false
    end)
end
function _G.ToggleQuickRecovery(on)
    _G.Vc.FakePerks.QuickRecoveryEnabled = on
    local F = _G.Vc.FakePerks
    if F.qrConnection then F.qrConnection:Disconnect(); F.qrConnection = nil end
    if on and LP.Character then
        F.qrConnection = LP.Character:GetAttributeChangedSignal("isvaulting"):Connect(function()
            local c = LP.Character
            if c and c:GetAttribute("isvaulting") == true then
                task.spawn(function()
                    task.wait(0.05)
                    local q = hasQuickRecoveryAnim(c)
                    if not q then task.wait(0.05); q = hasQuickRecoveryAnim(c) end
                    if not q and c:GetAttribute("isvaulting") == true then qrBoost(c) end
                end)
            end
        end)
    end
    notify("Fake Quick Recovery", on and "AKTIF" or "NONAKTIF")
end
local function plBoost(char)
    local F = _G.Vc.FakePerks
    if not F.PerfectLandingEnabled or not isSurvivor() or F.perfectLandingBoostActive then return end
    if tick() - F.lastPLTime < F.PerkCooldown then return end
    F.perfectLandingBoostActive = true; F.lastPLTime = tick()
    local now = tick()
    task.spawn(function()
        while F.perfectLandingBoostActive and tick() - now < 3 do
            if char and char.Parent then char:SetAttribute("speedboost", 1.4) end
            task.wait()
        end
        if char and char.Parent then char:SetAttribute("speedboost", 1) end
        F.perfectLandingBoostActive = false
    end)
end
function _G.TogglePerfectLanding(on)
    _G.Vc.FakePerks.PerfectLandingEnabled = on
    local F = _G.Vc.FakePerks
    if F.plConnection then F.plConnection:Disconnect(); F.plConnection = nil end
    if on and LP.Character then
        F.plConnection = LP.Character:GetAttributeChangedSignal("speedboost"):Connect(function()
            local c = LP.Character
            if c and isSurvivor() and c:GetAttribute("speedboost") == 0.625 then plBoost(c) end
        end)
    end
    notify("Fake Perfect Landing", on and "AKTIF" or "NONAKTIF")
end
function _G.ToggleFlowstate(on)
    _G.Vc.FakePerks.FlowstateEnabled = on
    local F = _G.Vc.FakePerks
    if F.fsConnection then F.fsConnection:Disconnect(); F.fsConnection = nil end
    if F.fsAnimConnection then F.fsAnimConnection:Disconnect(); F.fsAnimConnection = nil end
    if on and LP.Character then
        local c = LP.Character
        local h = c:WaitForChild("Humanoid", 3)
        local a = h and h:WaitForChild("Animator", 3)
        if a then
            F.fsAnimConnection = a.AnimationPlayed:Connect(function(tr)
                if tr.Animation and tr.Animation.AnimationId == "rbxassetid://136962284480779" then
                    local conn
                    conn = tr.Stopped:Connect(function()
                        if conn then conn:Disconnect() end
                        F.lastFSTime = tick(); F.fsOnCooldown = true
                        if c and c.Parent then c:SetAttribute("Flowstate", false) end
                        task.delay(F.PerkCooldown, function()
                            F.fsOnCooldown = false
                            if F.FlowstateEnabled and c and c.Parent then c:SetAttribute("Flowstate", true) end
                        end)
                    end)
                end
            end)
        end
        F.fsConnection = c:GetAttributeChangedSignal("Flowstate"):Connect(function()
            if not F.FlowstateEnabled or not isSurvivor() then return end
            local st = c:GetAttribute("Flowstate")
            if st == true then
                if F.fsOnCooldown or tick() - F.lastFSTime < F.PerkCooldown then c:SetAttribute("Flowstate", false) end
            elseif st == false and not F.fsOnCooldown and tick() - F.lastFSTime >= F.PerkCooldown then
                c:SetAttribute("Flowstate", true)
            end
        end)
        task.wait(0.5)
        if F.FlowstateEnabled and c and c.Parent then c:SetAttribute("Flowstate", true) end
    end
    notify("Fake Flowstate", on and "AKTIF" or "NONAKTIF")
end

-- ===== ADRENALINE =====
local function adrenalineBoost(char)
    local F = _G.Vc.FakePerks
    if not F.FakeAdrenalineRush or not isSurvivor() or F.boostActive then return end
    if tick() - F.lastARTime < F.PerkCooldown then return end
    F.boostActive = true; F.lastARTime = tick()
    local now = tick()
    task.spawn(function()
        while F.FakeAdrenalineRush and tick() - now < 5 do
            if char and char.Parent then char:SetAttribute("speedboost", 1.4) end
            task.wait()
        end
        if char and char.Parent then char:SetAttribute("speedboost", 1) end
        F.boostActive = false
    end)
end
function _G.SetupFakeAdrenalineRush(on)
    local F = _G.Vc.FakePerks
    F.FakeAdrenalineRush = on
    for _, c in ipairs(F.adrenalineConns) do pcall(function() c:Disconnect() end) end
    F.adrenalineConns = {}
    if not on then notify("Fake Adrenaline Rush", "NONAKTIF"); return end
    if LP.Character then
        local h = LP.Character:FindFirstChildOfClass("Humanoid")
        if h then
            local last = h.Health
            table.insert(F.adrenalineConns, h.HealthChanged:Connect(function(hp)
                if not F.FakeAdrenalineRush then return end
                if hp < last and hp <= 50 and hp > 0 then adrenalineBoost(LP.Character) end
                last = hp
            end))
        end
    end
    notify("Fake Adrenaline Rush", "AKTIF")
end

-- ===== STREAMER MODE =====
function _G.ToggleStreamerMode(on)
    _G.Vc.StreamerMode.Enabled = on
    local SM = _G.Vc.StreamerMode
    for _, c in ipairs(SM.SpooferConns) do if c.Connected then c:Disconnect() end end
    table.clear(SM.SpooferConns)
    local pg = LP:FindFirstChild("PlayerGui")
    if not pg then notify("Streamer Mode", on and "AKTIF" or "NONAKTIF"); return end
    if not on then
        for _, d in ipairs(pg:GetDescendants()) do
            if d:IsA("TextLabel") or d:IsA("TextButton") or d:IsA("TextBox") then
                local o = d:GetAttribute("OriginalText")
                if o then d.Text = o; d:SetAttribute("OriginalText", nil) end
            elseif d:IsA("ImageLabel") or d:IsA("ImageButton") then
                local o = d:GetAttribute("OriginalImage")
                if o then d.Image = o; d:SetAttribute("OriginalImage", nil) end
            end
        end
        notify("Streamer Mode", "NONAKTIF")
        return
    end
    local function spoof(obj)
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
            local function f()
                if not SM.Enabled then return end
                if not obj:GetAttribute("OriginalText") then obj:SetAttribute("OriginalText", obj.Text) end
                local txt = obj.Text
                for _, p in ipairs(Players:GetPlayers()) do
                    txt = string.gsub(txt, p.Name, "Player")
                    txt = string.gsub(txt, p.DisplayName, "Player")
                end
                if txt ~= obj.Text then obj.Text = txt end
            end
            f()
            table.insert(SM.SpooferConns, obj:GetPropertyChangedSignal("Text"):Connect(f))
        elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
            local function f()
                if not SM.Enabled then return end
                for _, p in ipairs(Players:GetPlayers()) do
                    if string.find(obj.Image, tostring(p.UserId)) then
                        if not obj:GetAttribute("OriginalImage") then obj:SetAttribute("OriginalImage", obj.Image) end
                        obj.Image = "rbxassetid://106073659711836"
                        break
                    end
                end
            end
            f()
            table.insert(SM.SpooferConns, obj:GetPropertyChangedSignal("Image"):Connect(f))
        end
    end
    for _, d in ipairs(pg:GetDescendants()) do spoof(d) end
    table.insert(SM.SpooferConns, pg.DescendantAdded:Connect(function(d) task.defer(function() spoof(d) end) end))
    notify("Streamer Mode", "AKTIF")
end

-- ===== ESP =====
local ESPtbl = { Generators = {}, Pallets = {}, Hooks = {}, Windows = {}, Gates = {}, SCPs = {} }

local function createESP(parent, name, opts)
    if not parent then return end
    local label = opts.name or ""
    local hasName = label ~= ""
    local single = not hasName
    local info = opts.infoRich or ""
    local existing = parent:FindFirstChild(name)
    if existing and existing:GetAttribute("Layout") ~= (single and "single" or "multi") then
        existing:Destroy(); existing = nil
    end
    if not existing then
        local bb = Instance.new("BillboardGui")
        bb.Name = name; bb.AlwaysOnTop = true
        bb:SetAttribute("Layout", single and "single" or "multi")
        bb.Size = single and UDim2.new(0, 220, 0, 22) or UDim2.new(0, 220, 0, 34)
        bb.StudsOffset = Vector3.new(0, opts.offsetY or 4, 0)
        bb.Parent = parent
        local box = Instance.new("Frame", bb)
        box.Name = "Box"; box.BackgroundTransparency = 1
        box.Size = UDim2.new(1, 0, 1, 0); box.AnchorPoint = Vector2.new(0.5, 0.5)
        box.Position = UDim2.new(0.5, 0, 0.5, 0)
        if single then
            local list = Instance.new("UIListLayout", box)
            list.FillDirection = Enum.FillDirection.Horizontal
            list.HorizontalAlignment = Enum.HorizontalAlignment.Center
            list.VerticalAlignment = Enum.VerticalAlignment.Center
            list.Padding = UDim.new(0, 4)
            local ico = Instance.new("ImageLabel", box)
            ico.Name = "Icon"; ico.BackgroundTransparency = 1
            ico.Size = UDim2.new(0, 14, 0, 14); ico.Visible = false
            local txt = Instance.new("TextLabel", box)
            txt.Name = "Text"; txt.BackgroundTransparency = 1
            txt.AutomaticSize = Enum.AutomaticSize.XY
            txt.Font = Enum.Font.GothamBold; txt.TextSize = 12
            txt.RichText = true; txt.TextStrokeTransparency = 0
        else
            local list = Instance.new("UIListLayout", box)
            list.FillDirection = Enum.FillDirection.Vertical
            list.HorizontalAlignment = Enum.HorizontalAlignment.Center
            list.VerticalAlignment = Enum.VerticalAlignment.Center
            local row = Instance.new("Frame", box)
            row.Name = "Row1"; row.BackgroundTransparency = 1
            row.AutomaticSize = Enum.AutomaticSize.XY; row.Size = UDim2.new(0, 0, 0, 16)
            local l2 = Instance.new("UIListLayout", row)
            l2.FillDirection = Enum.FillDirection.Horizontal
            l2.HorizontalAlignment = Enum.HorizontalAlignment.Center
            l2.Padding = UDim.new(0, 4)
            local ico = Instance.new("ImageLabel", row)
            ico.Name = "Icon"; ico.BackgroundTransparency = 1
            ico.Size = UDim2.new(0, 14, 0, 14); ico.Visible = false
            local nm = Instance.new("TextLabel", row)
            nm.Name = "Name"; nm.BackgroundTransparency = 1
            nm.AutomaticSize = Enum.AutomaticSize.XY
            nm.Font = Enum.Font.GothamBold; nm.TextSize = 12
            nm.TextStrokeTransparency = 0
            local inf = Instance.new("TextLabel", box)
            inf.Name = "Info"; inf.BackgroundTransparency = 1
            inf.AutomaticSize = Enum.AutomaticSize.XY
            inf.Font = Enum.Font.GothamBold; inf.TextSize = 11
            inf.RichText = true; inf.TextStrokeTransparency = 0
        end
        existing = bb
    end
    local box = existing:FindFirstChild("Box")
    if single then
        local t = box and box:FindFirstChild("Text")
        local i = box and box:FindFirstChild("Icon")
        if t then
            local c = opts.color and string.format("#%02X%02X%02X",
                math.floor(opts.color.R*255), math.floor(opts.color.G*255), math.floor(opts.color.B*255)) or "#FFFFFF"
            local nm = hasName and string.format("<font color=\"%s\">%s</font>", c, label) or ""
            local combined = nm
            if info ~= "" then combined = (combined == "" and info) or (combined .. " <font color=\"#888\">·</font> " .. info) end
            t.Text = combined
        end
        if i then i.Image = opts.icon or ""; i.Visible = opts.icon ~= nil and opts.icon ~= "" end
    else
        local row = box and box:FindFirstChild("Row1")
        local nm = row and row:FindFirstChild("Name")
        local i = row and row:FindFirstChild("Icon")
        local inf = box and box:FindFirstChild("Info")
        if nm then nm.Text = label; nm.TextColor3 = opts.color or Color3.new(1,1,1) end
        if i then i.Image = opts.icon or ""; i.Visible = opts.icon ~= nil and opts.icon ~= "" end
        if inf then if info == "" then inf.Visible = false else inf.Visible = true; inf.Text = info end end
    end
end

local function getItemIcon(itemName)
    local items = ReplicatedStorage:FindFirstChild("Items")
    if not items then return nil end
    local it = items:FindFirstChild(itemName)
    if not it then return nil end
    local img
    pcall(function() img = it.Texture or it.Image end)
    if img and img ~= "" then
        local id = tostring(img):match("%d+")
        if id then return string.format("rbxthumb://type=Asset&id=%s&w=420&h=420", id) end
    end
    return nil
end

local function survivorStatus(char)
    if not char then return nil, nil end
    if char:GetAttribute("IsHooked") then return "HOOKED", _G.Vc.ESPColors.Hooked end
    if char:GetAttribute("Knocked") then return "DOWN", _G.Vc.ESPColors.Downed end
    local h = char:FindFirstChildOfClass("Humanoid")
    if h and h.Health > 0 and h.Health < h.MaxHealth then return "INJURED", _G.Vc.ESPColors.Injured end
    return nil, nil
end

local function buildInfo(status, stColor, dist, hooks, isKiller)
    local parts = {}
    if status then
        local col = "#FFFFFF"
        if status == "INJURED" then col = "#FF8C00"
        elseif status == "DOWN" then col = "#FF2828"
        elseif status == "HOOKED" then col = "#F0F0FF" end
        table.insert(parts, string.format("<font color=\"%s\">%s</font>", col, status))
    end
    if dist then table.insert(parts, string.format("<font color=\"#FFFFFF\">%dm</font>", dist)) end
    if not isKiller and hooks and status ~= "HOOKED" then
        local col = "#C8C8C8"
        if hooks >= 3 then col = "#FF3232" elseif hooks >= 2 then col = "#FFC832" end
        table.insert(parts, string.format("<font color=\"%s\">Hooked %d</font>", col, hooks))
    end
    return table.concat(parts, " <font color=\"#888\">·</font> ")
end

function _G.UpdatePlayerESP()
    local C = _G.Vc.ESPConfig
    if not C.Master then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            if hrp and (Camera.CFrame.Position - hrp.Position).Magnitude <= C.MaxDistance then
                local isKiller = plr.Team and plr.Team.Name == "Killer"
                local show = isKiller and C.Killer or (not isKiller and C.Player)
                if show then
                    local hn = isKiller and "KEH" or "PEH"
                    local tn = isKiller and "KE_Text" or "PE_Text"
                    local col = isKiller and _G.Vc.ESPColors.Killer or _G.Vc.ESPColors.Player
                    local hl = plr.Character:FindFirstChild(hn)
                    if not hl then
                        hl = Instance.new("Highlight"); hl.Name = hn; hl.Parent = plr.Character
                    end
                    hl.FillColor = col; hl.OutlineColor = col
                    hl.FillTransparency = C.Outline and 1 or 0.5
                    local status, stCol
                    if not isKiller and C.Status then status, stCol = survivorStatus(plr.Character) end
                    local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
                    local hooks
                    if not isKiller and C.HookCount then
                        hooks = plr:GetAttribute("HookCount") or 0
                    end
                    local name = C.Name and plr.Name or nil
                    if isKiller then
                        local sel = plr:GetAttribute("SelectedKiller")
                        if sel then name = tostring(sel) end
                    end
                    local info = buildInfo(status, stCol, C.Distance and dist or nil, hooks, isKiller)
                    createESP(plr.Character, tn, {
                        name = name or "",
                        infoRich = info,
                        color = col,
                        icon = C.ItemIcon and not isKiller and getItemIcon(plr:GetAttribute("EquippedItem") or "") or nil,
                        offsetY = 4,
                    })
                end
            end
        end
    end
end

local function updateSCPESP()
    local C = _G.Vc.ESPConfig
    if not C.Master or not C.SCP then return end
    for _, scp in ipairs(ESPtbl.SCPs) do
        if scp and scp.Parent then
            local h = scp:FindFirstChildOfClass("Humanoid")
            local hrp = scp:FindFirstChild("HumanoidRootPart") or scp.PrimaryPart
            if h and hrp and h.Health > 0 and math.floor(h.WalkSpeed) == 9 and (Camera.CFrame.Position - hrp.Position).Magnitude <= C.MaxDistance then
                local hl = scp:FindFirstChild("SCPEH")
                if not hl then hl = Instance.new("Highlight"); hl.Name = "SCPEH"; hl.Parent = scp end
                hl.FillColor = _G.Vc.ESPColors.SCP; hl.OutlineColor = _G.Vc.ESPColors.SCP
                hl.FillTransparency = C.Outline and 1 or 0.5
            end
        end
    end
end

-- ===== SPEED / MOONWALK / FOV LOOP =====
RunService.RenderStepped:Connect(function(dt)
    if not _G.Vc.ScriptAlive then return end

    if _G.Vc.Speed.Enabled then
        local c = LP.Character
        local h = c and c:FindFirstChildOfClass("Humanoid")
        if h and h.MoveDirection.Magnitude > 0 then
            local hrp = c:FindFirstChild("HumanoidRootPart")
            if hrp then
                local n = (_G.Vc.Speed.Value - 0.1) * 10
                if n > 0 then hrp.CFrame = hrp.CFrame + h.MoveDirection * n * dt end
            end
        end
    end

    if _G.Vc.Moonwalk.Enabled then
        local c = LP.Character
        local hrp = c and c:FindFirstChild("HumanoidRootPart")
        local h = c and c:FindFirstChildOfClass("Humanoid")
        if hrp and h then
            h.AutoRotate = false
            local look = Vector3.new(Camera.CFrame.LookVector.X, 0, Camera.CFrame.LookVector.Z)
            local right = Vector3.new(Camera.CFrame.RightVector.X, 0, Camera.CFrame.RightVector.Z)
            if look.Magnitude > 0.001 then look = look.Unit end
            if right.Magnitude > 0.001 then right = right.Unit end
            local sway = math.sin(tick() * 35)
            local dir = CFrame.Angles(0, math.rad(30 * sway), 0) * look
            hrp.CFrame = CFrame.lookAt(hrp.Position, hrp.Position + dir)
            h:Move(-look * 1.2 + right * (sway * 1.05), false)
        end
    end

    if _G.Vc.StretchRes.Enabled then
        Camera.CFrame = Camera.CFrame * CFrame.new(0, 0, 0, 1, 0, 0, 0, _G.Vc.StretchRes.Amount, 0, 0, 0, 1)
    end

    local char = LP.Character
    local spearMode = char and char:GetAttribute("spearmode") == true
    local active = _G.Vc.AimConfig.Aim_SilentVeilV2 and spearMode
    if active and _G.Vc.AimConfig.Veil_ShowFOV then
        fovFrame.Visible = true
        fovFrame.Size = UDim2.new(0, _G.Vc.AimConfig.Veil_FOV * 2, 0, _G.Vc.AimConfig.Veil_FOV * 2)
        local tgt = _G.getClosestSurvivor()
        if tgt and tgt.Parent then
            veilHighlight.Parent = tgt.Parent
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local head = char:FindFirstChild("Head") or hrp
            updateTracker(head and head.Position, tgt.Position)
        else
            veilHighlight.Parent = nil
            updateTracker(nil, nil)
        end
    else
        fovFrame.Visible = false
        veilHighlight.Parent = nil
        updateTracker(nil, nil)
    end
end)

-- ESP loop
task.spawn(function()
    while _G.Vc.ScriptAlive do
        task.wait(2)
        if _G.Vc.ESPConfig.Master then
            pcall(_G.UpdatePlayerESP)
            pcall(updateSCPESP)
        end
    end
end)

-- GodMode + FOV
RunService.Heartbeat:Connect(function()
    if _G.Vc.GodMode then
        local c = LP.Character
        local h = c and c:FindFirstChildOfClass("Humanoid")
        if h and h.Health < h.MaxHealth then h.Health = h.MaxHealth end
    end
    if _G.Vc.CameraFOV.Enabled then Camera.FieldOfView = _G.Vc.CameraFOV.Value end
end)

-- ===== GUI FIRED HUB =====
local hub = Instance.new("ScreenGui")
hub.Name = "FiredHub"; hub.ResetOnSpawn = false; hub.Parent = CoreGui

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 320, 0, 480)
main.Position = UDim2.new(0, 20, 0, 60)
main.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
main.BorderSizePixel = 0; main.Active = true; main.Draggable = true
main.Parent = hub
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)
local st = Instance.new("UIStroke", main)
st.Color = Color3.fromRGB(255, 100, 50); st.Thickness = 1.5

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 34); title.BackgroundTransparency = 1
title.Text = "🔥 Fired Hub — Lafarzen"; title.TextColor3 = Color3.fromRGB(255, 180, 60)
title.Font = Enum.Font.GothamBold; title.TextSize = 15; title.Parent = main

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, -16, 1, -44)
scroll.Position = UDim2.new(0, 8, 0, 40)
scroll.BackgroundTransparency = 1; scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 4; scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.Parent = main
local list = Instance.new("UIListLayout", scroll)
list.Padding = UDim.new(0, 4); list.SortOrder = Enum.SortOrder.LayoutOrder
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y

local function mkToggle(text, default, cb)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(32, 32, 42)
    btn.Text = (default and "● " or "○ ") .. text
    btn.TextColor3 = default and Color3.fromRGB(120, 255, 120) or Color3.fromRGB(200, 200, 200)
    btn.Font = Enum.Font.GothamMedium; btn.TextSize = 12
    btn.BorderSizePixel = 0; btn.Parent = scroll
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    local state = default
    btn.MouseButton1Click:Connect(function()
        state = not state
                btn.Text = (state and "● " or "○ ") .. text
        btn.TextColor3 = state and Color3.fromRGB(120, 255, 120) or Color3.fromRGB(200, 200, 200)
        pcall(cb, state)
    end)
end

local function mkButton(text, cb)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(45, 35, 35)
    btn.Text = "▶ " .. text
    btn.TextColor3 = Color3.fromRGB(255, 180, 120)
    btn.Font = Enum.Font.GothamMedium; btn.TextSize = 12
    btn.BorderSizePixel = 0; btn.Parent = scroll
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    btn.MouseButton1Click:Connect(function() pcall(cb) end)
end

local function header(txt)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 22)
    lbl.BackgroundTransparency = 1
    lbl.Text = "— " .. txt .. " —"
    lbl.TextColor3 = Color3.fromRGB(255, 140, 80)
    lbl.Font = Enum.Font.GothamBold; lbl.TextSize = 11
    lbl.Parent = scroll
end

header("AIM")
mkToggle("Silent Aim Veil (Spear)", false, function(s) _G.Vc.AimConfig.Aim_SilentVeilV2 = s end)
mkToggle("Show Veil FOV", true, function(s) _G.Vc.AimConfig.Veil_ShowFOV = s end)
mkToggle("Smart Lead (Spear)", true, function(s) _G.Vc.Config.SpearSmart_enable = s end)

header("KILLER")
mkToggle("Counter Auto Parry", false, function(s) _G.ToggleCounterParry(s) end)
mkToggle("Auto Stalk", false, function(s) _G.ToggleAutoStalk(s) end)
mkButton("Infinite Lunge ON", function() _G.EnableInfiniteLunge() end)
mkButton("Infinite Lunge OFF", function() _G.DisableInfiniteLunge() end)

header("SURVIVOR / FAKE PERKS")
mkToggle("Fake Quick Recovery", false, function(s) _G.ToggleQuickRecovery(s) end)
mkToggle("Fake Perfect Landing", false, function(s) _G.TogglePerfectLanding(s) end)
mkToggle("Fake Flowstate", false, function(s) _G.ToggleFlowstate(s) end)
mkToggle("Fake Adrenaline Rush", false, function(s) _G.SetupFakeAdrenalineRush(s) end)

header("MOVEMENT")
mkToggle("Speed Hack", false, function(s) _G.Vc.Speed.Enabled = s end)
mkToggle("Moonwalk", false, function(s) _G.ToggleMoonwalkStatus() end)
mkToggle("God Mode", false, function(s) _G.Vc.GodMode = s end)
mkToggle("Stretch Res", false, function(s) _G.Vc.StretchRes.Enabled = s end)

header("ESP")
mkToggle("ESP Master", false, function(s) _G.Vc.ESPConfig.Master = s end)
mkToggle("ESP Player", true, function(s) _G.Vc.ESPConfig.Player = s end)
mkToggle("ESP Killer", true, function(s) _G.Vc.ESPConfig.Killer = s end)
mkToggle("ESP Status", true, function(s) _G.Vc.ESPConfig.Status = s end)
mkToggle("ESP Hook Count", true, function(s) _G.Vc.ESPConfig.HookCount = s end)
mkToggle("ESP Item Icon", true, function(s) _G.Vc.ESPConfig.ItemIcon = s end)
mkToggle("ESP Name", true, function(s) _G.Vc.ESPConfig.Name = s end)
mkToggle("ESP Distance", true, function(s) _G.Vc.ESPConfig.Distance = s end)
mkToggle("ESP Outline Only", false, function(s) _G.Vc.ESPConfig.Outline = s end)

header("MISC")
mkToggle("Streamer Mode", false, function(s) _G.ToggleStreamerMode(s) end)
mkToggle("Infinity Zoom", false, function(s)
    _G.Vc.InfinityZoom.Enabled = s
    if s then LP.CameraMaxZoomDistance = math.huge; LP.CameraMinZoomDistance = 0
    else LP.CameraMaxZoomDistance = 12; LP.CameraMinZoomDistance = 0.5 end
end)
mkToggle("Camera FOV", false, function(s) _G.Vc.CameraFOV.Enabled = s end)
mkToggle("No Fall Damage", false, function(s) _G.Vc.NoFall.Enabled = s end)
mkButton("Instan Escape", function()
    local c = LP.Character; local hrp = c and c:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, d in ipairs(workspace:GetDescendants()) do
        if string.lower(d.Name) == "fininshline" and d:IsA("BasePart") then
            hrp.CFrame = d.CFrame + Vector3.new(0, 5, 0); return
        end
    end
end)

print("[Fired Hub] Loaded ✅")
notify("Fired Hub", "Loaded! by Lafarzen")
