-- ============================================================
-- N3X CrossHairx — Silent Aim (Mobile + PC)
-- Dark red theme. Auto-scales for mobile screens.
-- Silent aim uses Mouse.Hit hook (PC) + TouchTap hook (mobile).
-- Enable + FOV toggles. 100% hit. Prediction locked on.
-- C4 Jump toggle. Hide button labeled "HIDE". Max FOV 1500.
-- ============================================================

local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")

local player    = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera    = workspace.CurrentCamera

-- ============================================================
-- PERSISTENCE CHECK
-- ============================================================
do
    local existing = playerGui:FindFirstChild("N3X_CrossHairx")
    if existing then
        existing.Enabled = true
        existing.DisplayOrder = 999
        local mainFrame = existing:FindFirstChild("MainFrame")
        if mainFrame then mainFrame.Visible = true end
        warn("[N3X CrossHairx] GUI already loaded — keeping existing instance.")
        return
    end
end

local connections = {}
local function keep(c) table.insert(connections, c); return c end

-- ============================================================
-- DEVICE DETECTION
-- ============================================================
local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
local viewportAtLoad = camera.ViewportSize
local isSmallScreen = viewportAtLoad.X < 800 or viewportAtLoad.Y < 500

-- ============================================================
-- THEME
-- ============================================================
local BG        = Color3.fromRGB(22, 8, 12)
local BG_HDR    = Color3.fromRGB(30, 12, 18)
local BG_LIST   = Color3.fromRGB(30, 12, 18)
local BG_ROW    = Color3.fromRGB(38, 14, 22)
local BG_ROW_HI = Color3.fromRGB(52, 18, 28)
local STROKE    = Color3.fromRGB(90, 34, 48)
local STROKE_HI = Color3.fromRGB(140, 58, 76)
local PINK      = Color3.fromRGB(196, 118, 138)
local HOT_PINK  = Color3.fromRGB(220, 90, 120)
local ACCENT    = Color3.fromRGB(196, 118, 138)
local ACCENT_D  = Color3.fromRGB(148, 78, 96)
local TEXT      = Color3.fromRGB(232, 214, 220)
local ON_C      = Color3.fromRGB(140, 44, 66)
local OFF_C     = Color3.fromRGB(58, 22, 32)
local KNOB      = Color3.fromRGB(226, 200, 208)
local FONT      = Enum.Font.Garamond

local BG_T = 0.15

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
-- RESPONSIVE SIZE
-- ============================================================
local PANEL_W, PANEL_H
local ROW_H, SWITCH_W, SWITCH_H, LABEL_W, SLIDER_W, SLIDER_GAP
local FONT_SIZE, FONT_SIZE_SMALL, TITLE_SIZE

if isMobile or isSmallScreen then
    -- Mobile / small screens: shrink everything
    local vp = viewportAtLoad
    PANEL_W = math.clamp(vp.X * 0.85, 260, 340)
    PANEL_H = math.clamp(vp.Y * 0.55, 240, 320)
    ROW_H       = 32
    SWITCH_W    = 44
    SWITCH_H    = 24
    LABEL_W     = 112
    SLIDER_W    = math.max(80, PANEL_W - 40 - LABEL_W - 24)
    SLIDER_GAP  = 12
    FONT_SIZE        = 15
    FONT_SIZE_SMALL  = 13
    TITLE_SIZE       = 15
else
    -- Desktop
    PANEL_W = 360
    PANEL_H = 260
    ROW_H       = 24
    SWITCH_W    = 38
    SWITCH_H    = 20
    LABEL_W     = 118
    SLIDER_W    = 110
    SLIDER_GAP  = 12
    FONT_SIZE        = 15
    FONT_SIZE_SMALL  = 13
    TITLE_SIZE       = 15
end

-- SCREEN GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "N3X_CrossHairx"
ScreenGui.Parent = playerGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999

-- FOV CIRCLE
local FovCircle = Instance.new("Frame")
FovCircle.Name = "N3X_FovCircle"
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
fovStroke.Color = PINK
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

-- FOV circle follows mouse on PC, sits at screen center on mobile
keep(RunService.RenderStepped:Connect(function()
    if not FovCircle.Visible then return end
    if isMobile then
        FovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
    else
        local m = UserInputService:GetMouseLocation()
        FovCircle.Position = UDim2.new(0, m.X, 0, m.Y)
    end
end))

-- MAIN FRAME
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

-- Header
local Header = Instance.new("Frame")
Header.Parent = Frame
Header.BackgroundColor3 = BG_HDR
Header.BackgroundTransparency = 0
Header.BorderSizePixel = 0
Header.Size = UDim2.new(1, 0, 0, 34)
Header.ZIndex = 5
corner(Header, 10)

local HeaderMask = Instance.new("Frame")
HeaderMask.Parent = Header
HeaderMask.BackgroundColor3 = BG_HDR
HeaderMask.BorderSizePixel = 0
HeaderMask.Position = UDim2.new(0, 0, 1, -10)
HeaderMask.Size = UDim2.new(1, 0, 0, 10)
HeaderMask.ZIndex = 5

local Title = Instance.new("TextLabel")
Title.Parent = Header
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 14, 0, 0)
Title.Size = UDim2.new(1, -28, 1, 0)
Title.Font = Enum.Font.Garamond
Title.Text = "N3X CROSSHAIRX"
Title.TextColor3 = PINK
Title.TextSize = TITLE_SIZE
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.ZIndex = 6

local TitleStroke = Instance.new("UIStroke")
TitleStroke.Color = Color3.fromRGB(60, 22, 32)
TitleStroke.Thickness = 1
TitleStroke.Parent = Title

-- PAGE HOLDER
local PageHolder = Instance.new("Frame")
PageHolder.Parent = Frame
PageHolder.BackgroundTransparency = 1
PageHolder.Position = UDim2.new(0, 0, 0, 34)
PageHolder.Size = UDim2.new(1, 0, 1, -34)
PageHolder.ZIndex = 4

local SilentPage = Instance.new("Frame")
SilentPage.Name = "SilentPage"
SilentPage.Parent = PageHolder
SilentPage.BackgroundTransparency = 1
SilentPage.Size = UDim2.new(1, 0, 1, 0)
SilentPage.Position = UDim2.new(0, 0, 0, 0)
SilentPage.Visible = true
SilentPage.ZIndex = 4

-- ROW HELPERS
local function makeScroll(parent)
    local sf = Instance.new("ScrollingFrame")
    sf.Parent = parent
    sf.BackgroundTransparency = 1
    sf.BorderSizePixel = 0
    sf.Position = UDim2.new(0, 14, 0, 12)
    sf.Size = UDim2.new(1, -24, 1, -22)
    sf.CanvasSize = UDim2.new(0, 0, 0, 0)
    sf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    sf.ScrollBarThickness = 3
    sf.ScrollBarImageColor3 = ACCENT_D
    sf.ZIndex = 4
    local l = Instance.new("UIListLayout")
    l.Padding = UDim.new(0, isMobile and 8 or 6)
    l.SortOrder = Enum.SortOrder.LayoutOrder
    l.Parent = sf
    return sf
end

local function Section(parent, text)
    local l = Instance.new("TextLabel")
    l.Parent = parent
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 0, isMobile and 20 or 18)
    l.Font = FONT
    l.Text = string.upper(text)
    l.TextColor3 = ACCENT
    l.TextSize = FONT_SIZE_SMALL
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.TextYAlignment = Enum.TextYAlignment.Center
    l.ZIndex = 4
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(60, 22, 32)
    s.Thickness = 1
    s.Parent = l
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
    lbl.TextSize = FONT_SIZE
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
    return h, makeSwitch(h, LABEL_W + 8, default, callback)
end

local function Slider(parent, label, default, min, max, onChange)
    min = min or 0; max = max or 100
    local pct = math.clamp((default - min) / (max - min), 0, 1)
    local h = makeRow(parent, ROW_H)

    local lbl = Instance.new("TextLabel")
    lbl.Parent = h
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(0, LABEL_W, 1, 0)
    lbl.Font = FONT
    lbl.Text = label .. ": " .. default
    lbl.TextColor3 = TEXT
    lbl.TextSize = FONT_SIZE
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextYAlignment = Enum.TextYAlignment.Center
    lbl.TextTruncate = Enum.TextTruncate.AtEnd
    lbl.ZIndex = 4
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(20, 6, 10)
    s.Thickness = 1
    s.Transparency = 0.3
    s.Parent = lbl

    local trackX = LABEL_W + SLIDER_GAP

    local trackF = Instance.new("Frame")
    trackF.Parent = h
    trackF.AnchorPoint = Vector2.new(0, 0.5)
    trackF.BackgroundColor3 = Color3.fromRGB(40, 16, 24)
    trackF.BorderSizePixel = 0
    trackF.Position = UDim2.new(0, trackX, 0.5, 0)
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
    hit.Position = UDim2.new(0, trackX - 12, 0, 0)
    hit.Size = UDim2.new(0, SLIDER_W + 24, 1, 0)
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

local function Dropdown(parent, label, options, default, onChange)
    local h = makeRow(parent, ROW_H)
    makeLabel(h, label)

    local btn = Instance.new("TextButton")
    btn.Parent = h
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, -2, 0.5, 0)
    btn.Size = UDim2.new(0, 100, 1, 0)
    btn.BackgroundColor3 = BG_ROW
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = FONT
    btn.Text = default .. "  v"
    btn.TextColor3 = TEXT
    btn.TextSize = FONT_SIZE_SMALL
    btn.ZIndex = 4
    corner(btn, 6)
    stroke(btn, STROKE, 1, 0.3)

    local list = Instance.new("Frame")
    list.Parent = parent
    list.BackgroundColor3 = BG_LIST
    list.BorderSizePixel = 0
    list.Size = UDim2.new(1, -4, 0, #options * (isMobile and 28 or 22) + 8)
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
        optBtn.Size = UDim2.new(1, 0, 0, isMobile and 26 or 20)
        optBtn.Font = FONT
        optBtn.Text = opt
        optBtn.TextColor3 = TEXT
        optBtn.TextSize = FONT_SIZE_SMALL
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
    SilentEnabled      = true,
    SilentShowFOV      = true,
    SilentTargetPart   = "HumanoidRootPart",
    SilentFOV          = 120,
    SilentVisibleCheck = false,
    SilentPredictionX  = 1.60,
    SilentPredictionY  = 1.50,
    SilentHitChance    = 100,
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

local function getPing()
    local ok, ping = pcall(function() return player:GetNetworkPing() * 1000 end)
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
    return part.Position
        + (velocity * travelTime * predictionX)
        + Vector3.new(0, velocity.Y * travelTime * predictionY, 0)
end

local function getAimPosition(part)
    if not part then return nil end
    return predictPosition(
        part,
        settings.SilentPredictionX,
        settings.SilentPredictionY,
        true
    ) or part.Position
end

-- ============================================================
-- SILENT AIM — PC (Mouse.Hit hook) + Mobile (TouchTap hook)
-- ============================================================
do
    local mouse = player:GetMouse()

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

    -- reference point for FOV: mouse on PC, screen center on mobile
    local function referencePoint()
        if isMobile then
            local vp = camera.ViewportSize
            return Vector2.new(vp.X / 2, vp.Y / 2)
        else
            return UserInputService:GetMouseLocation()
        end
    end

    local function closest()
        local ref = referencePoint()
        local best, bestD = nil, math.huge
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and alive(p) then
                local pt = resolveTargetPartSilent(p.Character)
                if pt then
                    local sp, on = camera:WorldToViewportPoint(pt.Position)
                    if on then
                        local d = (Vector2.new(sp.X, sp.Y) - ref).Magnitude
                        if d <= settings.SilentFOV and d < bestD and visible(p, pt) then
                            best, bestD = pt, d
                        end
                    end
                end
            end
        end
        return best
    end

    -- ========================================================
    -- PC: Mouse.Hit metatable hook
    -- ========================================================
    pcall(function()
        local mt = getrawmetatable and getrawmetatable(game)
        if mt and setreadonly and hookfunction and checkcaller and newcclosure then
            local oldIndex = mt.__index
            setreadonly(mt, false)
            mt.__index = newcclosure(function(self, key)
                if settings.SilentEnabled
                    and self == mouse
                    and (key == "Hit" or key == "Target" or key == "UnitRay")
                    and not checkcaller() then

                    local t = closest()
                    if t then
                        local aimPos = getAimPosition(t) or t.Position
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
                return oldIndex(self, key)
            end)
            setreadonly(mt, true)
        else
            warn("[N3X CrossHairx] Executor doesn't support metatable hooks (PC silent aim may not work).")
        end
    end)

    -- ========================================================
    -- MOBILE: UserInputService.TouchTap + TapInWorld hook
    -- Redirects the touch fire target to the closest enemy.
    -- Also hooks TouchStarted for games that read touch position.
    -- ========================================================
    if isMobile then
        -- 1) Direct tap in world — this is what most mobile games read
        --    for "where did you shoot?" — but the game usually uses
        --    UserInputService:GetMouseLocation() which on mobile returns
        --    last tap position. We can spoof via Mouse hook as well.
        -- 2) Mouse.X / Mouse.Y / Mouse.Hit on mobile return last touch,
        --    so the metatable hook above already covers it.

        -- Below: intercept TouchTap to redirect fire input before the
        -- game processes it. TapTap gives us the tap position, we don't
        -- consume it (input is preserved), we just recompute Mouse.Hit
        -- via the metatable hook above — which mobile executors support
        -- if they allow getrawmetatable.

        -- Additional guard: some mobile games read from UserInputService:GetMouseLocation()
        -- at fire time. This value is updated by TouchTap's position, so we can
        -- nudge it by hooking GetMouseLocation to return target screen position.
        if hookfunction and newcclosure and checkcaller then
            pcall(function()
                local originalGetMouseLocation = UserInputService.GetMouseLocation
                UserInputService.GetMouseLocation = newcclosure(function(self, ...)
                    if settings.SilentEnabled and not checkcaller() then
                        local t = closest()
                        if t then
                            local sp = camera:WorldToViewportPoint(t.Position)
                            if sp then
                                return Vector2.new(sp.X, sp.Y)
                            end
                        end
                    end
                    return originalGetMouseLocation(self, ...)
                end)
                print("[N3X CrossHairx] Mobile GetMouseLocation hook installed.")
            end)
        end
    end
end

-- FOV CIRCLE UPDATER
keep(RunService.RenderStepped:Connect(function()
    if settings.SilentShowFOV and settings.SilentEnabled then
        _G.N3X_FovCircle.show(settings.SilentFOV, true)
    else
        _G.N3X_FovCircle.show(0, false)
    end
end))

-- ============================================================
-- SILENT AIM PAGE
-- ============================================================
do
    local sc = makeScroll(SilentPage)

    Section(sc, "Silent Aim")
    Toggle(sc, "Enable", true, function(on) settings.SilentEnabled = on end)
    Toggle(sc, "Show FOV Circle", true, function(on) settings.SilentShowFOV = on end)
    Slider(sc, "FOV", 120, 5, 1500, function(v) settings.SilentFOV = v end)

    Section(sc, "Prediction")
    Slider(sc, "Predict X %", 160, 0, 300, function(v) settings.SilentPredictionX = v / 100 end)
    Slider(sc, "Predict Y %", 150, 0, 300, function(v) settings.SilentPredictionY = v / 100 end)

    Section(sc, "Target")
    Dropdown(sc, "Part",
        { "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso", "Random", "Nearest" },
        "HumanoidRootPart",
        function(opt) settings.SilentTargetPart = opt end)

    Section(sc, "Checks")
    Toggle(sc, "Visible Check", false, function(on) settings.SilentVisibleCheck = on end)

    Section(sc, "Extras")
    do
        local c4Enabled  = false
        local c4Conns    = {}
        local c4CooldownUntil = 0
        local C4_COOLDOWN = 24
        local c4Placing  = false
        local c4LastPlace = 0
        local c4HookedTools = {}

        local function c4DisconnectAll()
            for _, c in ipairs(c4Conns) do pcall(function() c:Disconnect() end) end
            c4Conns = {}
        end

        local function c4FindBomb()
            local char = player.Character
            if not char then return nil end
            local t = char:FindFirstChild("FakeBomb")
            if t and t:IsA("Tool") then return t end
            return nil
        end

        local function c4CenterCFrame()
            local char = player.Character
            if not char then return nil end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp then return nil end
            local cam = workspace.CurrentCamera
            if not cam then return nil end
            local camDir = cam.CFrame.LookVector
            local pos = hrp.Position + (camDir * 5)
            local look = camDir
            local right = Vector3.new(0, 1, 0):Cross(look)
            local up = look:Cross(right)
            return CFrame.new(
                pos.X, pos.Y, pos.Z,
                right.X, up.X, look.X,
                right.Y, up.Y, look.Y,
                right.Z, up.Z, look.Z
            )
        end

        local function c4MakeJump()
            local char = player.Character
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.Jumping) end)
            end
        end

        local function c4SuppressDrop(tool)
            if not tool or c4HookedTools[tool] then return end
            c4HookedTools[tool] = true
            pcall(function() tool.CanBeDropped = false end)
            tool.Activated:Connect(function() end)
        end

        local function c4PlaceBomb()
            if c4Placing then return end
            if os.clock() < c4CooldownUntil then return end
            local now = os.clock()
            if now - c4LastPlace < 0.3 then return end

            local bomb = c4FindBomb()
            if not bomb then return end
            local remote = bomb:FindFirstChild("Remote")
            if not remote then return end
            local cf = c4CenterCFrame()
            if not cf then return end

            c4Placing = true
            c4LastPlace = now
            local ok = pcall(function() remote:FireServer(cf, 50) end)
            if ok then
                c4CooldownUntil = os.clock() + C4_COOLDOWN
                c4MakeJump()
            end
            task.delay(0.2, function() c4Placing = false end)
        end

        local function c4IsBlockingGui(pos)
            local hits = playerGui:GetGuiObjectsAtPosition(pos.X, pos.Y)
            for _, h in ipairs(hits) do
                if h.Visible and (h:IsA("GuiButton") or h:IsA("TextBox")) then
                    return true
                end
            end
            return false
        end

        local function c4Start()
            c4DisconnectAll()
            table.insert(c4Conns, RunService.Heartbeat:Connect(function()
                if not c4Enabled then return end
                local bomb = c4FindBomb()
                if bomb then c4SuppressDrop(bomb) end
            end))
            table.insert(c4Conns, UserInputService.InputBegan:Connect(function(input, gp)
                if gp then return end
                if not c4Enabled then return end
                if UserInputService:GetFocusedTextBox() then return end
                if input.UserInputType ~= Enum.UserInputType.MouseButton1
                    and input.UserInputType ~= Enum.UserInputType.Touch then return end
                if not c4FindBomb() then return end
                if c4IsBlockingGui(input.Position) then return end
                c4PlaceBomb()
            end))
        end

        local function c4Stop()
            c4DisconnectAll()
            c4CooldownUntil = 0
            c4HookedTools = {}
        end

        Toggle(sc, "C4 Jump", false, function(on)
            c4Enabled = on
            if on then c4Start() else c4Stop() end
        end)
    end
end

-- ============================================================
-- OPEN / CLOSE BUTTON — smaller on mobile, positioned well
-- ============================================================
local BTN_W = isMobile and 60 or 60
local BTN_H = isMobile and 34 or 30
local BTN_X = isMobile and 12 or 20
local BTN_Y = isMobile and 100 or 120

local ToggleButton = Instance.new("TextButton")
ToggleButton.Parent = ScreenGui
ToggleButton.Position = UDim2.new(0, BTN_X, 0, BTN_Y)
ToggleButton.Size = UDim2.new(0, BTN_W, 0, BTN_H)
ToggleButton.BackgroundColor3 = BG_HDR
ToggleButton.BackgroundTransparency = 0
ToggleButton.BorderSizePixel = 0
ToggleButton.AutoButtonColor = false
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Text = "HIDE"
ToggleButton.TextColor3 = PINK
ToggleButton.TextSize = isMobile and 14 or 13
ToggleButton.ZIndex = 50
ToggleButton.Active = true
corner(ToggleButton, 6)
stroke(ToggleButton, STROKE_HI, 1.5, 0.1)

ToggleButton.MouseEnter:Connect(function()
    TweenService:Create(ToggleButton, TweenInfo.new(0.12), { BackgroundColor3 = BG_ROW_HI }):Play()
end)
ToggleButton.MouseLeave:Connect(function()
    TweenService:Create(ToggleButton, TweenInfo.new(0.12), { BackgroundColor3 = BG_HDR }):Play()
end)
ToggleButton.MouseButton1Click:Connect(function()
    Frame.Visible = not Frame.Visible
    ToggleButton.Text = Frame.Visible and "HIDE" or "SHOW"
end)

keep(UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        Frame.Visible = not Frame.Visible
        ToggleButton.Text = Frame.Visible and "HIDE" or "SHOW"
    end
end))

-- ============================================================
-- DRAG — tap anywhere on the header (works on mobile + PC)
-- ============================================================
local dragging, dragStart, startPos = false, nil, nil
keep(Header.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = i.Position; startPos = Frame.Position
    end
end))
keep(UserInputService.InputChanged:Connect(function(i)
    if dragging and dragStart and (i.UserInputType == Enum.UserInputType.MouseMovement
        or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - dragStart
        Frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end))
keep(UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end))

-- ============================================================
-- AUTO-REPOSITION on viewport change (rotation, resize)
-- ============================================================
keep(camera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
    if not Frame.Visible then return end
    local vp = camera.ViewportSize
    -- keep panel roughly centered but within bounds
    local absPos = Frame.AbsolutePosition
    local absSize = Frame.AbsoluteSize
    if absPos.X < 0 or absPos.X + absSize.X > vp.X
       or absPos.Y < 0 or absPos.Y + absSize.Y > vp.Y then
        Frame.Position = UDim2.new(0.5, 0, 0.55, 0)
    end
    -- clamp hide button
    if ToggleButton.AbsolutePosition.X + ToggleButton.AbsoluteSize.X > vp.X then
        ToggleButton.Position = UDim2.new(0, vp.X - BTN_W - 12, 0, BTN_Y)
    end
end))

print("[N3X CrossHairx] Silent Aim loaded — " .. (isMobile and "MOBILE mode" or "PC mode") .. ", FOV max 1500.")
