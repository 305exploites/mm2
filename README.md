-- ============================================================
-- Silent Aim — clean minimal panel
-- Dark red theme. Mobile-optimized.
-- Drag ONLY through the top strip.
-- Square toggle button, loadstring-safe.
-- ============================================================

local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")

local player    = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui", 10)
if not playerGui then
    playerGui = player:FindFirstChildOfClass("PlayerGui")
end
local camera    = workspace.CurrentCamera

-- ============================================================
-- PERSISTENCE CHECK
-- ============================================================
do
    local existing = playerGui:FindFirstChild("N3X_SilentAim")
    if existing then
        existing.Enabled = true
        existing.DisplayOrder = 2147483647
        local mainFrame = existing:FindFirstChild("MainFrame")
        if mainFrame then mainFrame.Visible = true end
        local topBtn = existing:FindFirstChild("OpenClose")
        if topBtn then
            topBtn.Visible = true
            topBtn.Parent = existing
        end
        warn("[Silent Aim] GUI already loaded — keeping existing instance.")
        return
    end
end

local connections = {}
local function keep(c) table.insert(connections, c); return c end

-- ============================================================
-- DEVICE DETECTION
-- ============================================================
local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled

-- ============================================================
-- THEME
-- ============================================================
local BG        = Color3.fromRGB(22, 8, 12)
local BG_PANEL  = Color3.fromRGB(30, 12, 18)
local BG_ROW    = Color3.fromRGB(38, 14, 22)
local BG_ROW_HI = Color3.fromRGB(52, 18, 28)
local STROKE    = Color3.fromRGB(90, 34, 48)
local STROKE_HI = Color3.fromRGB(140, 58, 76)
local ACCENT    = Color3.fromRGB(196, 118, 138)
local ACCENT_D  = Color3.fromRGB(148, 78, 96)
local TEXT      = Color3.fromRGB(232, 214, 220)
local TEXT_MUT  = Color3.fromRGB(168, 130, 142)
local ON_C      = Color3.fromRGB(140, 44, 66)
local OFF_C     = Color3.fromRGB(58, 22, 32)
local KNOB      = Color3.fromRGB(226, 200, 208)
local FONT      = Enum.Font.Garamond
local FONT_B    = Enum.Font.Garamond

local BG_T      = 0.25

local function corner(o, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 6)
    c.Parent = o
    return c
end

local function stroke(o, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color or STROKE
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.Parent = o
    return s
end

-- ============================================================
-- SCREEN GUI — top-most, on PlayerGui
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "N3X_SilentAim"
ScreenGui.Parent = playerGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = false          -- keep button below the top bar
ScreenGui.DisplayOrder = 2147483647       -- absolute top

-- ============================================================
-- FOV CIRCLE
-- ============================================================
local FovCircle = Instance.new("Frame")
FovCircle.Name = "FovCircle"
FovCircle.Parent = ScreenGui
FovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FovCircle.BackgroundTransparency = 1
FovCircle.BorderSizePixel = 0
FovCircle.Visible = false
FovCircle.ZIndex = 2

local fovCorner = Instance.new("UICorner")
fovCorner.CornerRadius = UDim.new(1, 0)
fovCorner.Parent = FovCircle

local fovStroke = Instance.new("UIStroke")
fovStroke.Color = ACCENT
fovStroke.Thickness = 1
fovStroke.Transparency = 0.4
fovStroke.Parent = FovCircle

_G.N3X_FovCircle = {
    frame = FovCircle,
    show = function(radius, visible)
        local r = radius * 2
        FovCircle.Size = UDim2.new(0, r, 0, r)
        FovCircle.Visible = visible == true
    end
}

keep(RunService.RenderStepped:Connect(function()
    if not FovCircle.Visible then return end
    local m = UserInputService:GetMouseLocation()
    FovCircle.Position = UDim2.new(0, m.X, 0, m.Y)
end))

-- ============================================================
-- RESPONSIVE PANEL SIZE
-- ============================================================
local PANEL_W, PANEL_H
if isMobile then
    local vp = camera and camera.ViewportSize or Vector2.new(800, 600)
    PANEL_W = math.clamp(vp.X * 0.78, 260, 340)
    PANEL_H = math.clamp(vp.Y * 0.55, 260, 380)
else
    PANEL_W = 310
    PANEL_H = 350
end

local ROW_H       = isMobile and 30 or 24
local SWITCH_W    = isMobile and 44 or 38
local SWITCH_H    = isMobile and 24 or 20
local LABEL_W     = isMobile and 108 or 118
local SLIDER_W    = isMobile and 76  or 90
local GAP         = 8

local DRAG_HANDLE_H = isMobile and 52 or 44

-- ============================================================
-- MAIN PANEL
-- ============================================================
local Frame = Instance.new("Frame")
Frame.Name = "MainFrame"
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = BG
Frame.BackgroundTransparency = BG_T
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0.5, 0.5)
Frame.Position = isMobile and UDim2.new(0.5, 0, 0.55, 0) or UDim2.new(0.5, 0, 0.5, 0)
Frame.Size = UDim2.new(0, PANEL_W, 0, PANEL_H)
Frame.Active = true
Frame.Visible = true
Frame.ZIndex = 1
Frame.ClipsDescendants = false
corner(Frame, 10)
stroke(Frame, STROKE_HI, 1, 0.35)

local innerStroke = Instance.new("UIStroke")
innerStroke.Color = Color3.fromRGB(70, 26, 38)
innerStroke.Thickness = 1
innerStroke.Transparency = 0.6
innerStroke.Parent = Frame

-- Drag handle
local DragHandle = Instance.new("Frame")
DragHandle.Name = "DragHandle"
DragHandle.Parent = Frame
DragHandle.BackgroundTransparency = 1
DragHandle.BorderSizePixel = 0
DragHandle.Position = UDim2.new(0, 0, 0, 0)
DragHandle.Size = UDim2.new(1, 0, 0, DRAG_HANDLE_H)
DragHandle.ZIndex = 15
DragHandle.Active = true

local Content = Instance.new("Frame")
Content.Parent = Frame
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 0, 0, 0)
Content.Size = UDim2.new(1, 0, 1, 0)
Content.ZIndex = 4

-- ============================================================
-- ROW HELPERS
-- ============================================================
local function makeScroll(parent)
    local sf = Instance.new("ScrollingFrame")
    sf.Parent = parent
    sf.BackgroundTransparency = 1
    sf.BorderSizePixel = 0
    sf.Position = UDim2.new(0, 14, 0, 14)
    sf.Size = UDim2.new(1, -24, 1, -26)
    sf.CanvasSize = UDim2.new(0, 0, 0, 0)
    sf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    sf.ScrollBarThickness = 2
    sf.ScrollBarImageColor3 = ACCENT_D
    sf.ZIndex = 4
    local l = Instance.new("UIListLayout")
    l.Padding = UDim.new(0, isMobile and 10 or 8)
    l.SortOrder = Enum.SortOrder.LayoutOrder
    l.Parent = sf
    return sf
end

local function Section(parent, text)
    local l = Instance.new("TextLabel")
    l.Parent = parent
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 0, isMobile and 22 or 20)
    l.Font = FONT_B
    l.Text = string.upper(text)
    l.TextColor3 = ACCENT
    l.TextSize = isMobile and 14 or 13
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.TextYAlignment = Enum.TextYAlignment.Center
    l.ZIndex = 4
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(60, 22, 32)
    s.Thickness = 1
    s.Parent = l
end

local function Divider(parent)
    local d = Instance.new("Frame")
    d.Parent = parent
    d.BackgroundColor3 = STROKE
    d.BackgroundTransparency = 0.4
    d.BorderSizePixel = 0
    d.Size = UDim2.new(1, 0, 0, 1)
    d.ZIndex = 4
end

local function makeRow(parent, h)
    local f = Instance.new("Frame")
    f.Parent = parent
    f.BackgroundTransparency = 1
    f.Size = UDim2.new(1, 0, 0, h or ROW_H)
    f.ZIndex = 4
    return f
end

local function makeLabel(holder, text)
    local lbl = Instance.new("TextLabel")
    lbl.Parent = holder
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(0, LABEL_W, 1, 0)
    lbl.Font = FONT
    lbl.Text = text
    lbl.TextColor3 = TEXT
    lbl.TextSize = isMobile and 16 or 15
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextYAlignment = Enum.TextYAlignment.Center
    lbl.TextTruncate = Enum.TextTruncate.AtEnd
    lbl.ZIndex = 4
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(20, 6, 10)
    s.Thickness = 1
    s.Transparency = 0.3
    s.Parent = lbl
    return lbl
end

local function makeSwitch(holder, x, default, callback)
    local trackF = Instance.new("Frame")
    trackF.Parent = holder
    trackF.AnchorPoint = Vector2.new(0, 0.5)
    trackF.Position = UDim2.new(0, x, 0.5, 0)
    trackF.Size = UDim2.new(0, SWITCH_W, 0, SWITCH_H)
    trackF.BackgroundColor3 = default and ON_C or OFF_C
    trackF.BorderSizePixel = 0
    trackF.ZIndex = 4
    corner(trackF, 1000)

    local ts = Instance.new("UIStroke")
    ts.Color = default and STROKE_HI or STROKE
    ts.Thickness = 1
    ts.Transparency = 0.2
    ts.Parent = trackF

    local knob = Instance.new("Frame")
    knob.Parent = trackF
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = KNOB
    knob.BorderSizePixel = 0
    knob.Size = UDim2.new(0, SWITCH_H - 4, 0, SWITCH_H - 4)
    knob.Position = default and UDim2.new(1, -SWITCH_H/2, 0.5, 0) or UDim2.new(0, SWITCH_H/2, 0.5, 0)
    knob.ZIndex = 5
    corner(knob, 1000)
    local ks = Instance.new("UIStroke")
    ks.Color = Color3.fromRGB(60, 20, 30)
    ks.Thickness = 1
    ks.Transparency = 0.3
    ks.Parent = knob

    local click = Instance.new("TextButton")
    click.Parent = trackF
    click.BackgroundTransparency = 1
    click.Text = ""
    click.Size = UDim2.new(1, 0, 1, 0)
    click.AutoButtonColor = false
    click.ZIndex = 6

    local state = default and true or false
    local function setState(v, fire)
        state = v == true
        TweenService:Create(knob, TweenInfo.new(0.18, Enum.EasingStyle.Quad), {
            Position = state and UDim2.new(1, -SWITCH_H/2, 0.5, 0) or UDim2.new(0, SWITCH_H/2, 0.5, 0)
        }):Play()
        TweenService:Create(trackF, TweenInfo.new(0.18), {
            BackgroundColor3 = state and ON_C or OFF_C
        }):Play()
        TweenService:Create(ts, TweenInfo.new(0.18), {
            Color = state and STROKE_HI or STROKE
        }):Play()
        if fire and callback then task.spawn(callback, state) end
    end
    click.MouseButton1Click:Connect(function() setState(not state, true) end)
    return setState
end

local function Toggle(parent, label, default, callback)
    local h = makeRow(parent, ROW_H)
    makeLabel(h, label)
    return h, makeSwitch(h, LABEL_W + GAP, default, callback)
end

local SLIDER_X = LABEL_W + GAP + SWITCH_W + 10

local function Slider(parent, label, default, min, max, onChange, onToggle)
    min = min or 0; max = max or 100
    local pct = math.clamp((default - min) / (max - min), 0, 1)
    local h = makeRow(parent, ROW_H)
    local lbl = makeLabel(h, label .. ": " .. default)
    makeSwitch(h, LABEL_W + GAP, false, onToggle)

    local trackF = Instance.new("Frame")
    trackF.Parent = h
    trackF.AnchorPoint = Vector2.new(0, 0.5)
    trackF.BackgroundColor3 = Color3.fromRGB(40, 16, 24)
    trackF.BorderSizePixel = 0
    trackF.Position = UDim2.new(0, SLIDER_X, 0.5, 0)
    trackF.Size = UDim2.new(0, SLIDER_W, 0, isMobile and 6 or 5)
    trackF.ZIndex = 4
    corner(trackF, 1000)
    local ts = Instance.new("UIStroke")
    ts.Color = STROKE
    ts.Thickness = 1
    ts.Transparency = 0.3
    ts.Parent = trackF

    local fill = Instance.new("Frame")
    fill.Parent = trackF
    fill.BackgroundColor3 = ACCENT_D
    fill.BorderSizePixel = 0
    fill.Size = UDim2.new(pct, 0, 1, 0)
    fill.ZIndex = 5
    corner(fill, 1000)

    local knob = Instance.new("Frame")
    knob.Parent = trackF
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = KNOB
    knob.BorderSizePixel = 0
    knob.Position = UDim2.new(pct, 0, 0.5, 0)
    knob.Size = UDim2.new(0, isMobile and 16 or 12, 0, isMobile and 16 or 12)
    knob.ZIndex = 6
    corner(knob, 1000)
    local ks = Instance.new("UIStroke")
    ks.Color = Color3.fromRGB(60, 20, 30)
    ks.Thickness = 1
    ks.Transparency = 0.3
    ks.Parent = knob

    local hit = Instance.new("TextButton")
    hit.Parent = h
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.AutoButtonColor = false
    hit.Position = UDim2.new(0, SLIDER_X - 8, 0, 0)
    hit.Size = UDim2.new(0, SLIDER_W + 16, 1, 0)
    hit.ZIndex = 7

    local dragging = false
    local last = default
    local function update(x)
        local rel = math.clamp((x - trackF.AbsolutePosition.X) / trackF.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        knob.Position = UDim2.new(rel, 0, 0.5, 0)
        local v = math.floor(min + (max - min) * rel + 0.5)
        lbl.Text = label .. ": " .. v
        if v ~= last then
            last = v
            if onChange then task.spawn(onChange, v) end
        end
    end
    hit.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true; update(i.Position.X)
        end
    end)
    keep(UserInputService.InputChanged:Connect(function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            update(i.Position.X)
        end
    end))
    keep(UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end))
end

-- ============================================================
-- KEYBIND PICKER
-- ============================================================
local function KeybindRow(parent, label, getKey, setKey)
    local h = makeRow(parent, ROW_H)
    makeLabel(h, label)

    local btn = Instance.new("TextButton")
    btn.Parent = h
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, -2, 0.5, 0)
    btn.Size = UDim2.new(0, isMobile and 96 or 100, 1, 0)
    btn.BackgroundColor3 = BG_ROW
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = FONT
    btn.Text = tostring(getKey()):gsub("Enum.KeyCode.", ""):gsub("Enum.UserInputType.", "")
    btn.TextColor3 = TEXT
    btn.TextSize = isMobile and 15 or 14
    btn.ZIndex = 4
    corner(btn, 6)
    stroke(btn, STROKE, 1, 0.3)

    local listening = false
    local conn

    local function stopListening()
        listening = false
        btn.Text = tostring(getKey()):gsub("Enum.KeyCode.", ""):gsub("Enum.UserInputType.", "")
        btn.TextColor3 = TEXT
        if conn then conn:Disconnect(); conn = nil end
    end

    btn.MouseButton1Click:Connect(function()
        if listening then stopListening(); return end
        listening = true
        btn.Text = "Press..."
        btn.TextColor3 = ACCENT

        conn = UserInputService.InputBegan:Connect(function(input, gp)
            if gp then return end
            if input.UserInputType == Enum.UserInputType.Keyboard then
                setKey(input.KeyCode)
                stopListening()
            elseif input.UserInputType == Enum.UserInputType.MouseButton1
                or input.UserInputType == Enum.UserInputType.MouseButton2
                or input.UserInputType == Enum.UserInputType.MouseButton3 then
                setKey(input.UserInputType)
                stopListening()
            end
        end)

        keep(UserInputService.InputEnded:Connect(function(input)
            if not listening then return end
            if input.KeyCode == Enum.KeyCode.Escape then stopListening() end
        end))
    end)

    return btn
end

local function inputMatches(inputType, keyCode, bind)
    if not bind then return false end
    if typeof(bind) == "EnumItem" then
        if bind.EnumType == Enum.KeyCode and inputType == Enum.UserInputType.Keyboard and keyCode == bind then
            return true
        end
        if bind.EnumType == Enum.UserInputType and bind == inputType then
            return true
        end
    end
    return false
end

-- ============================================================
-- DROPDOWN
-- ============================================================
local function Dropdown(parent, label, options, default, onChange)
    local h = makeRow(parent, ROW_H)
    makeLabel(h, label)

    local btn = Instance.new("TextButton")
    btn.Parent = h
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, -2, 0.5, 0)
    btn.Size = UDim2.new(0, isMobile and 96 or 100, 1, 0)
    btn.BackgroundColor3 = BG_ROW
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = FONT
    btn.Text = default .. "  v"
    btn.TextColor3 = TEXT
    btn.TextSize = isMobile and 15 or 14
    btn.ZIndex = 4
    corner(btn, 6)
    stroke(btn, STROKE, 1, 0.3)

    local list = Instance.new("Frame")
    list.Parent = parent
    list.BackgroundColor3 = BG_PANEL
    list.BorderSizePixel = 0
    list.Size = UDim2.new(1, -4, 0, #options * (isMobile and 26 or 22) + 8)
    list.Visible = false
    list.ZIndex = 20
    corner(list, 6)
    stroke(list, STROKE_HI, 1, 0.3)

    local lLayout = Instance.new("UIListLayout")
    lLayout.Padding = UDim.new(0, 2)
    lLayout.SortOrder = Enum.SortOrder.LayoutOrder
    lLayout.Parent = list

    local lPad = Instance.new("UIPadding")
    lPad.PaddingTop = UDim.new(0, 4)
    lPad.PaddingBottom = UDim.new(0, 4)
    lPad.PaddingLeft = UDim.new(0, 4)
    lPad.PaddingRight = UDim.new(0, 4)
    lPad.Parent = list

    local open = false
    local function setOpen(v)
        open = v == true
        list.Visible = open
        if open then
            local absPos = btn.AbsolutePosition
            local absSize = btn.AbsoluteSize
            local parentAbs = list.Parent.AbsolutePosition
            list.Position = UDim2.new(0, absPos.X - parentAbs.X, 0, absPos.Y - parentAbs.Y + absSize.Y + 2)
        end
    end

    btn.MouseButton1Click:Connect(function() setOpen(not open) end)

    for _, opt in ipairs(options) do
        local optBtn = Instance.new("TextButton")
        optBtn.Parent = list
        optBtn.BackgroundColor3 = BG_ROW
        optBtn.BorderSizePixel = 0
        optBtn.AutoButtonColor = false
        optBtn.Size = UDim2.new(1, 0, 0, isMobile and 24 or 20)
        optBtn.Font = FONT
        optBtn.Text = opt
        optBtn.TextColor3 = TEXT
        optBtn.TextSize = isMobile and 15 or 14
        optBtn.ZIndex = 21
        corner(optBtn, 4)

        optBtn.MouseEnter:Connect(function() optBtn.BackgroundColor3 = BG_ROW_HI end)
        optBtn.MouseLeave:Connect(function() optBtn.BackgroundColor3 = BG_ROW end)
        optBtn.MouseButton1Click:Connect(function()
            btn.Text = opt .. "  v"
            setOpen(false)
            if onChange then task.spawn(onChange, opt) end
        end)
    end

    keep(UserInputService.InputBegan:Connect(function(i, gp)
        if gp then return end
        if not open then return end
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            local m = UserInputService:GetMouseLocation()
            local ap = btn.AbsolutePosition
            local as = btn.AbsoluteSize
            local lp = list.AbsolutePosition
            local ls_ = list.AbsoluteSize
            local inBtn = m.X >= ap.X and m.X <= ap.X + as.X and m.Y >= ap.Y and m.Y <= ap.Y + as.Y
            local inList = m.X >= lp.X and m.X <= lp.X + ls_.X and m.Y >= lp.Y and m.Y <= lp.Y + ls_.Y
            if not inBtn and not inList then setOpen(false) end
        end
    end))

    return btn, list
end

-- ============================================================
-- STATE
-- ============================================================
local settings = {
    SilentEnabled        = false,
    SilentAlwaysOn       = false,
    SilentTargetPart     = "Head",
    SilentFOV            = 120,
    SilentHitChance      = 100,
    SilentVisibleCheck   = false,
    SilentUseKey         = false,
    SilentKey            = Enum.KeyCode.Q,
    SilentShowFOV        = true,
    SilentPrediction     = true,
    SilentPredictionX    = 1.60,
    SilentPredictionY    = 1.50,
    SilentPingCompensate = true,
}
_G.N3X_Settings = settings

local function resolveTargetPartSilent(char)
    if not char then return nil end
    local w = settings.SilentTargetPart
    if w == "Random" then
        local picks = { "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso" }
        w = picks[math.random(1, #picks)]
    elseif w == "Nearest" then
        local mouse = UserInputService:GetMouseLocation()
        local best, bestD = nil, math.huge
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("BasePart") and d.Name ~= "HumanoidRootPart" then
                local sp, on = camera:WorldToViewportPoint(d.Position)
                if on then
                    local dd = (Vector2.new(sp.X, sp.Y) - mouse).Magnitude
                    if dd < bestD then best, bestD = d, dd end
                end
            end
        end
        return best
    end
    return char:FindFirstChild(w) or char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
end

-- ============================================================
-- MM2 PREDICTION HELPERS
-- ============================================================
local function getPing()
    local ok, ping = pcall(function()
        return player:GetNetworkPing() * 1000
    end)
    if ok and ping then return ping end
    local stats = game:GetService("Stats")
    local ok2, p = pcall(function()
        return stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    if ok2 and p then return p end
    return 50
end

local function predictPosition(part, predictionX, predictionY, pingCompensate)
    if not part then return nil end
    local velocity = part.AssemblyLinearVelocity or part.Velocity

    local pingSeconds = 0
    if pingCompensate then
        pingSeconds = getPing() / 1000 / 2
    end

    local travelTime = 0.05 + pingSeconds

    local predicted = part.Position
        + (velocity * travelTime * predictionX)
        + Vector3.new(0, velocity.Y * travelTime * predictionY, 0)

    return predicted
end

local function getAimPosition(part, aimSettings)
    if not part then return nil end
    if not aimSettings.PredictionEnabled then
        return part.Position
    end
    return predictPosition(
        part,
        aimSettings.PredictionX,
        aimSettings.PredictionY,
        aimSettings.PingCompensate
    ) or part.Position
end

-- ============================================================
-- SILENT AIM — MM2 Mouse.Hit Hook
-- ============================================================
do
    local held = false
    local mouse = player:GetMouse()

    local function isKeyHeld()
        return held or settings.SilentAlwaysOn
    end

    local function alive(plr)
        local c = plr.Character
        if not c then return false end
        local h = c:FindFirstChildOfClass("Humanoid")
        return h and h.Health > 0
    end

    local ray = RaycastParams.new()
    ray.FilterType = Enum.RaycastFilterType.Exclude
    ray.IgnoreWater = true

    local function visible(plr, part)
        if not settings.SilentVisibleCheck then return true end
        if not part then return false end
        local f = {}
        if player.Character then table.insert(f, player.Character) end
        ray.FilterDescendantsInstances = f
        local o = camera.CFrame.Position
        local hit = workspace:Raycast(o, part.Position - o, ray)
        if not hit then return true end
        return hit.Instance and hit.Instance:IsDescendantOf(plr.Character)
    end

    local function closest()
        local mouseLoc = UserInputService:GetMouseLocation()
        local best, bestD = nil, math.huge
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and alive(p) then
                local pt = resolveTargetPartSilent(p.Character)
                if pt then
                    local sp, on = camera:WorldToViewportPoint(pt.Position)
                    if on then
                        local d = (Vector2.new(sp.X, sp.Y) - mouseLoc).Magnitude
                        if d <= settings.SilentFOV and d < bestD and visible(p, pt) then
                            best, bestD = pt, d
                        end
                    end
                end
            end
        end
        return best
    end

    pcall(function()
        local mt = getrawmetatable and getrawmetatable(game)
        if mt and setreadonly and hookfunction and checkcaller and newcclosure then
            local oldIndex = mt.__index
            setreadonly(mt, false)
            mt.__index = newcclosure(function(self, key)
                local isActive = settings.SilentEnabled
                    and (isKeyHeld() or (not settings.SilentUseKey))

                if isActive
                    and self == mouse
                    and (key == "Hit" or key == "Target" or key == "UnitRay")
                    and not checkcaller() then

                    if math.random(1, 100) <= settings.SilentHitChance then
                        local t = closest()
                        if t then
                            local aimPos = getAimPosition(t, {
                                PredictionEnabled = settings.SilentPrediction,
                                PredictionX       = settings.SilentPredictionX,
                                PredictionY       = settings.SilentPredictionY,
                                PingCompensate    = settings.SilentPingCompensate,
                            }) or t.Position

                            if key == "Hit" then
                                return CFrame.new(aimPos)
                            elseif key == "Target" then
                                return t
                            elseif key == "UnitRay" then
                                local origin = camera.CFrame.Position
                                local dir = (aimPos - origin).Unit
                                return Ray.new(origin, dir)
                            end
                        end
                    end
                end
                return oldIndex(self, key)
            end)
            setreadonly(mt, true)
        else
            warn("[Silent Aim] Executor doesn't support metatable hooks.")
        end
    end)

    keep(UserInputService.InputBegan:Connect(function(i, gp)
        if gp then return end
        if inputMatches(i.UserInputType, i.KeyCode, settings.SilentKey) then
            held = true
        end
    end))
    keep(UserInputService.InputEnded:Connect(function(i)
        if inputMatches(i.UserInputType, i.KeyCode, settings.SilentKey) then
            held = false
        end
    end))

    _G.N3X_SilentAim = { setEnabled = function(on) settings.SilentEnabled = on == true end }
end

-- ============================================================
-- FOV CIRCLE UPDATER
-- ============================================================
keep(RunService.RenderStepped:Connect(function()
    if settings.SilentShowFOV and settings.SilentEnabled then
        _G.N3X_FovCircle.show(settings.SilentFOV, true)
    else
        _G.N3X_FovCircle.show(0, false)
    end
end))

-- ============================================================
-- CONTENT — SILENT AIM ONLY
-- ============================================================
do
    local sc = makeScroll(Content)

    Section(sc, "Silent Aim")
    Divider(sc)

    Toggle(sc, "Enable", false, function(on)
        settings.SilentEnabled = on
        if _G.N3X_SilentAim then _G.N3X_SilentAim.setEnabled(on) end
    end)
    Toggle(sc, "Always On", false, function(on)
        settings.SilentAlwaysOn = on
    end)
    Toggle(sc, "Use Key", false, function(on)
        settings.SilentUseKey = on
    end)
    KeybindRow(sc, "Bind Key", function() return settings.SilentKey end, function(k) settings.SilentKey = k end)

    Divider(sc)
    Slider(sc, "FOV", 120, 5, 600, function(v) settings.SilentFOV = v end)
    Slider(sc, "Hit %", 100, 1, 100, function(v) settings.SilentHitChance = v end)

    Divider(sc)
    Section(sc, "Prediction")
    Divider(sc)

    Toggle(sc, "Enable Prediction", true, function(on) settings.SilentPrediction = on end)
    Toggle(sc, "Ping Compensate", true, function(on) settings.SilentPingCompensate = on end)
    Slider(sc, "Predict X %", 160, 0, 300, function(v) settings.SilentPredictionX = v / 100 end)
    Slider(sc, "Predict Y %", 150, 0, 300, function(v) settings.SilentPredictionY = v / 100 end)

    Divider(sc)
    Section(sc, "Target")
    Divider(sc)

    Dropdown(sc, "Part", { "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso", "Random", "Nearest" }, "Head", function(opt)
        settings.SilentTargetPart = opt
    end)

    Divider(sc)
    Section(sc, "Checks")
    Divider(sc)

    Toggle(sc, "Visible Check", false, function(on) settings.SilentVisibleCheck = on end)
end

-- ============================================================
-- CUTE SCREEN EFFECTS
-- ============================================================
local function playBloom()
    local bloom = Instance.new("Frame")
    bloom.Name = "Bloom"
    bloom.Parent = ScreenGui
    bloom.AnchorPoint = Vector2.new(0.5, 0.5)
    bloom.Position = Frame.Position
    bloom.Size = UDim2.new(0, 40, 0, 40)
    bloom.BackgroundColor3 = ACCENT
    bloom.BackgroundTransparency = 0.55
    bloom.BorderSizePixel = 0
    bloom.ZIndex = 40
    corner(bloom, 1000)

    local bStroke = Instance.new("UIStroke")
    bStroke.Color = STROKE_HI
    bStroke.Thickness = 2
    bStroke.Transparency = 0.4
    bStroke.Parent = bloom

    TweenService:Create(bloom, TweenInfo.new(0.55, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 260, 0, 260),
        BackgroundTransparency = 1,
        Rotation = 90,
    }):Play()
    TweenService:Create(bStroke, TweenInfo.new(0.55, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Transparency = 1,
        Thickness = 0,
    }):Play()

    task.delay(0.6, function()
        if bloom then bloom:Destroy() end
    end)
end

local function playShimmer()
    local shimmer = Instance.new("Frame")
    shimmer.Name = "Shimmer"
    shimmer.Parent = ScreenGui
    shimmer.AnchorPoint = Vector2.new(0.5, 0.5)
    shimmer.Position = Frame.Position
    shimmer.Size = UDim2.new(0, 220, 0, 220)
    shimmer.BackgroundColor3 = ACCENT_D
    shimmer.BackgroundTransparency = 1
    shimmer.BorderSizePixel = 0
    shimmer.ZIndex = 40
    corner(shimmer, 1000)

    local sStroke = Instance.new("UIStroke")
    sStroke.Color = STROKE_HI
    sStroke.Thickness = 2
    sStroke.Transparency = 0.3
    sStroke.Parent = shimmer

    TweenService:Create(shimmer, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 40, 0, 40),
        BackgroundTransparency = 0.7,
    }):Play()
    TweenService:Create(sStroke, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Transparency = 1,
    }):Play()

    task.delay(0.45, function()
        if shimmer then shimmer:Destroy() end
    end)
end

-- ============================================================
-- OPEN / CLOSE BUTTON — square, top center
-- ============================================================
local BTN_SIZE = isMobile and 46 or 40

local OpenCloseBtn = Instance.new("TextButton")
OpenCloseBtn.Name = "OpenClose"
OpenCloseBtn.Parent = ScreenGui
OpenCloseBtn.AnchorPoint = Vector2.new(0.5, 0)
OpenCloseBtn.Position = UDim2.new(0.5, 0, 0, isMobile and 22 or 14)
OpenCloseBtn.Size = UDim2.new(0, BTN_SIZE, 0, BTN_SIZE)
OpenCloseBtn.BackgroundColor3 = BG_PANEL
OpenCloseBtn.BackgroundTransparency = 0
OpenCloseBtn.BorderSizePixel = 0
OpenCloseBtn.AutoButtonColor = false
OpenCloseBtn.Font = Enum.Font.GothamBold
OpenCloseBtn.Text = "X"
OpenCloseBtn.TextColor3 = ACCENT
OpenCloseBtn.TextSize = isMobile and 20 or 16
OpenCloseBtn.ZIndex = 100
OpenCloseBtn.Active = true
OpenCloseBtn.Visible = true
corner(OpenCloseBtn, 6)          -- rounded square, not a circle
stroke(OpenCloseBtn, STROKE_HI, 1.5, 0.1)

local btnGlow = Instance.new("UIStroke")
btnGlow.Color = ACCENT_D
btnGlow.Thickness = 2
btnGlow.Transparency = 0.7
btnGlow.Parent = OpenCloseBtn

OpenCloseBtn.MouseEnter:Connect(function()
    TweenService:Create(OpenCloseBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = BG_ROW_HI
    }):Play()
end)
OpenCloseBtn.MouseLeave:Connect(function()
    TweenService:Create(OpenCloseBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = BG_PANEL
    }):Play()
end)

local isOpen = true
local function setOpen(v)
    if v == isOpen then return end
    isOpen = v

    if isOpen then
        playShimmer()
        Frame.Visible = true
        Frame.Size = UDim2.new(0, PANEL_W * 0.9, 0, PANEL_H * 0.9)
        TweenService:Create(Frame, TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, PANEL_W, 0, PANEL_H),
        }):Play()
        OpenCloseBtn.Text = "X"
    else
        playBloom()
        local t = TweenService:Create(Frame, TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Size = UDim2.new(0, PANEL_W * 0.85, 0, PANEL_H * 0.85),
        })
        t:Play()
        t.Completed:Connect(function()
            Frame.Visible = false
            Frame.Size = UDim2.new(0, PANEL_W, 0, PANEL_H)
        end)
        OpenCloseBtn.Text = "O"
    end
end

OpenCloseBtn.MouseButton1Click:Connect(function()
    setOpen(not isOpen)
end)

keep(UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        setOpen(not isOpen)
    end
end))

-- Safety: ensure the button is on screen after everything initializes
task.defer(function()
    task.wait(0.1)
    if OpenCloseBtn and OpenCloseBtn.Parent then
        OpenCloseBtn.Visible = true
        OpenCloseBtn.ZIndex = 100
    end
    if ScreenGui and ScreenGui.Parent ~= playerGui then
        ScreenGui.Parent = playerGui
    end
end)

-- ============================================================
-- DRAG — ONLY via the top strip
-- ============================================================
local function pointInsideDragHandle(pos)
    if not Frame.Visible then return false end
    local ap = DragHandle.AbsolutePosition
    local as = DragHandle.AbsoluteSize
    return pos.X >= ap.X and pos.X <= ap.X + as.X
       and pos.Y >= ap.Y and pos.Y <= ap.Y + as.Y
end

local dragState = {
    active = false,
    pending = false,
    startInput = nil,
    startPos = nil,
    moveConn = nil,
    endConn = nil,
}
local DRAG_DEADZONE = 4

local function stopDrag()
    dragState.active = false
    dragState.pending = false
    dragState.startInput = nil
    if dragState.moveConn then dragState.moveConn:Disconnect(); dragState.moveConn = nil end
    if dragState.endConn then dragState.endConn:Disconnect(); dragState.endConn = nil end
end

keep(UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.UserInputType ~= Enum.UserInputType.MouseButton1
        and input.UserInputType ~= Enum.UserInputType.Touch then return end
    if not Frame.Visible then return end

    local pos = input.Position
    if not pointInsideDragHandle(pos) then return end

    dragState.pending = true
    dragState.active = false
    dragState.startInput = pos
    dragState.startPos = Frame.Position

    dragState.moveConn = UserInputService.InputChanged:Connect(function(m)
        if not dragState.pending and not dragState.active then return end
        if m.UserInputType ~= Enum.UserInputType.MouseMovement
            and m.UserInputType ~= Enum.UserInputType.Touch then return end
        local d = m.Position - dragState.startInput
        if not dragState.active and d.Magnitude > DRAG_DEADZONE then
            dragState.active = true
            dragState.pending = false
        end
        if dragState.active then
            Frame.Position = UDim2.new(
                dragState.startPos.X.Scale, dragState.startPos.X.Offset + d.X,
                dragState.startPos.Y.Scale, dragState.startPos.Y.Offset + d.Y)
        end
    end)
    keep(dragState.moveConn)

    dragState.endConn = UserInputService.InputEnded:Connect(function(m)
        if m.UserInputType == Enum.UserInputType.MouseButton1
            or m.UserInputType == Enum.UserInputType.Touch then
            stopDrag()
        end
    end)
    keep(dragState.endConn)
end))

print("[Silent Aim] Clean panel loaded.")
