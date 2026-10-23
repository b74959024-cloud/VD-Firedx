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
