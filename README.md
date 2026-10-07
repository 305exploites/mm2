-- ============================================================
-- N3X CrossHairx — Silent Aim
-- Pink + Red theme. Sharper corners, more opaque.
-- Scope icon in header. Persistent across re-executions.
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
-- THEME — pink + red, sharper, more opaque
-- ============================================================
local BG        = Color3.fromRGB(28, 4, 10)
local BG_HDR    = Color3.fromRGB(58, 6, 16)
local BG_LIST   = Color3.fromRGB(36, 4, 12)
local BG_ROW    = Color3.fromRGB(48, 6, 14)
local STROKE    = Color3.fromRGB(140, 30, 55)
local STROKE_HI = Color3.fromRGB(255, 90, 130)
local PINK      = Color3.fromRGB(255, 114, 135)
local HOT_PINK  = Color3.fromRGB(255, 60, 110)
local ACCENT    = Color3.fromRGB(255, 130, 160)
local TEXT      = Color3.fromRGB(255, 220, 230)
local TEXT_MUT  = Color3.fromRGB(200, 140, 160)
local ON_C      = Color3.fromRGB(220, 40, 90)
local OFF_C     = Color3.fromRGB(80, 20, 35)
local FONT      = Enum.Font.Gotham

local BG_T      = 0.02
local BTN_T     = 0.05

local function corner(o, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 4)
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
-- SCREEN GUI
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "N3X_CrossHairx"
ScreenGui.Parent = playerGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999

-- ============================================================
-- FOV CIRCLE
-- ============================================================
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
fovStroke.Thickness = 1.5
fovStroke.Transparency = 0.15
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
-- MAIN FRAME
-- ============================================================
local Frame = Instance.new("Frame")
Frame.Name = "MainFrame"
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = BG
Frame.BackgroundTransparency = BG_T
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0.5, 0.5)
Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
Frame.Size = UDim2.new(0, 360, 0, 260)
Frame.Active = true
Frame.Visible = true
Frame.ZIndex = 1
corner(Frame, 6)
stroke(Frame, HOT_PINK, 1.5, 0.1)

-- Header
local Header = Instance.new("Frame")
Header.Parent = Frame
Header.BackgroundColor3 = BG_HDR
Header.BackgroundTransparency = 0
Header.BorderSizePixel = 0
Header.Size = UDim2.new(1, 0, 0, 34)
Header.ZIndex = 5
corner(Header, 6)

local HeaderMask = Instance.new("Frame")
HeaderMask.Parent = Header
HeaderMask.BackgroundColor3 = BG_HDR
HeaderMask.BorderSizePixel = 0
HeaderMask.Position = UDim2.new(0, 0, 1, -10)
HeaderMask.Size = UDim2.new(1, 0, 0, 10)
HeaderMask.ZIndex = 5

-- ============================================================
-- SCOPE ICON — primitives, no square
-- ============================================================
local ScopeIcon = Instance.new("Frame")
ScopeIcon.Name = "ScopeIcon"
ScopeIcon.Parent = Header
ScopeIcon.BackgroundTransparency = 1
ScopeIcon.Position = UDim2.new(0, 12, 0.5, 0)
ScopeIcon.AnchorPoint = Vector2.new(0, 0.5)
ScopeIcon.Size = UDim2.new(0, 26, 0, 26)
ScopeIcon.ZIndex = 6

-- Outer ring
local scopeRing = Instance.new("Frame")
scopeRing.Parent = ScopeIcon
scopeRing.AnchorPoint = Vector2.new(0.5, 0.5)
scopeRing.Position = UDim2.new(0.5, 0, 0.5, 0)
scopeRing.Size = UDim2.new(1, 0, 1, 0)
scopeRing.BackgroundTransparency = 1
scopeRing.BorderSizePixel = 0
scopeRing.ZIndex = 6

local ringCorner = Instance.new("UICorner")
ringCorner.CornerRadius = UDim.new(1, 0)
ringCorner.Parent = scopeRing

local ringStroke = Instance.new("UIStroke")
ringStroke.Color = PINK
ringStroke.Thickness = 1.5
ringStroke.Transparency = 0.1
ringStroke.Parent = scopeRing

-- Inner dot
local scopeDot = Instance.new("Frame")
scopeDot.Parent = ScopeIcon
scopeDot.AnchorPoint = Vector2.new(0.5, 0.5)
scopeDot.Position = UDim2.new(0.5, 0, 0.5, 0)
scopeDot.Size = UDim2.new(0, 4, 0, 4)
scopeDot.BackgroundColor3 = HOT_PINK
scopeDot.BorderSizePixel = 0
scopeDot.ZIndex = 8

local dotCorner = Instance.new("UICorner")
dotCorner.CornerRadius = UDim.new(1, 0)
dotCorner.Parent = scopeDot

local dotStroke = Instance.new("UIStroke")
dotStroke.Color = Color3.fromRGB(255, 255, 255)
dotStroke.Thickness = 1
dotStroke.Transparency = 0.4
dotStroke.Parent = scopeDot

-- Crosshair arms (4 ticks)
local function makeArm(pos, size)
    local arm = Instance.new("Frame")
    arm.Parent = ScopeIcon
    arm.AnchorPoint = Vector2.new(0.5, 0.5)
    arm.Position = pos
    arm.Size = size
    arm.BackgroundColor3 = PINK
    arm.BorderSizePixel = 0
    arm.ZIndex = 7
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(1, 0)
    c.Parent = arm
    return arm
end

makeArm(UDim2.new(0.5, 0, 0, 3), UDim2.new(0, 2, 0, 6))
makeArm(UDim2.new(0.5, 0, 1, -3), UDim2.new(0, 2, 0, 6))
makeArm(UDim2.new(0, 3, 0.5, 0), UDim2.new(0, 6, 0, 2))
makeArm(UDim2.new(1, -3, 0.5, 0), UDim2.new(0, 6, 0, 2))

-- Title
local Title = Instance.new("TextLabel")
Title.Parent = Header
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 46, 0, 0)
Title.Size = UDim2.new(1, -54, 1, 0)
Title.Font = Enum.Font.GothamBold
Title.Text = "N3X CrossHairx"
Title.TextColor3 = PINK
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.ZIndex = 6

local TitleStroke = Instance.new("UIStroke")
TitleStroke.Color = Color3.fromRGB(120, 20, 45)
TitleStroke.Thickness = 1
TitleStroke.Parent = Title

-- ============================================================
-- PAGE HOLDER
-- ============================================================
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

-- ============================================================
-- ROW HELPERS
-- ============================================================
local function makeScroll(parent)
    local sf = Instance.new("ScrollingFrame")
    sf.Parent = parent
    sf.BackgroundTransparency = 1
    sf.BorderSizePixel = 0
    sf.Position = UDim2.new(0, 12, 0, 8)
    sf.Size = UDim2.new(1, -20, 1, -16)
    sf.CanvasSize = UDim2.new(0, 0, 0, 0)
    sf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    sf.ScrollBarThickness = 3
    sf.ScrollBarImageColor3 = PINK
    sf.ZIndex = 4
    local l = Instance.new("UIListLayout")
    l.Padding = UDim.new(0, 6)
    l.SortOrder = Enum.SortOrder.LayoutOrder
    l.Parent = sf
    return sf
end

local function Section(parent, text)
    local l = Instance.new("TextLabel")
    l.Parent = parent
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 0, 16)
    l.Font = FONT
    l.Text = text
    l.TextColor3 = PINK
    l.TextSize = 11
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.ZIndex = 4
end

local function Divider(parent)
    local d = Instance.new("Frame")
    d.Parent = parent
    d.BackgroundColor3 = STROKE
    d.BackgroundTransparency = 0.2
    d.BorderSizePixel = 0
    d.Size = UDim2.new(1, 0, 0, 1)
    d.ZIndex = 4
end

local LABEL_W  = 108
local GAP      = 8
local SWITCH_W = 32
local SWITCH_H = 16

local function makeRow(parent, h)
    local f = Instance.new("Frame")
    f.Parent = parent
    f.BackgroundTransparency = 1
    f.Size = UDim2.new(1, 0, 0, h)
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
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextYAlignment = Enum.TextYAlignment.Center
    lbl.TextTruncate = Enum.TextTruncate.AtEnd
    lbl.ZIndex = 4
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
    corner(trackF, 4)

    local knob = Instance.new("Frame")
    knob.Parent = trackF
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = Color3.fromRGB(255, 220, 230)
    knob.BorderSizePixel = 0
    knob.Size = UDim2.new(0, 12, 0, 12)
    knob.Position = default and UDim2.new(1, -8, 0.5, 0) or UDim2.new(0, 8, 0.5, 0)
    knob.ZIndex = 5
    corner(knob, 3)

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
        TweenService:Create(knob, TweenInfo.new(0.15), {
            Position = state and UDim2.new(1, -8, 0.5, 0) or UDim2.new(0, 8, 0.5, 0)
        }):Play()
        TweenService:Create(trackF, TweenInfo.new(0.15), {
            BackgroundColor3 = state and ON_C or OFF_C
        }):Play()
        if fire and callback then task.spawn(callback, state) end
    end
    click.MouseButton1Click:Connect(function() setState(not state, true) end)
    return setState
end

local function Toggle(parent, label, default, callback)
    local h = makeRow(parent, 20)
    makeLabel(h, label)
    return h, makeSwitch(h, LABEL_W + GAP, default, callback)
end

local SLIDER_X = LABEL_W + GAP + SWITCH_W + 8
local SLIDER_W = 110

local function Slider(parent, label, default, min, max, onChange, onToggle)
    min = min or 0; max = max or 100
    local pct = math.clamp((default - min) / (max - min), 0, 1)
    local h = makeRow(parent, 20)
    local lbl = makeLabel(h, label .. ": " .. default)
    makeSwitch(h, LABEL_W + GAP, false, onToggle)

    local trackF = Instance.new("Frame")
    trackF.Parent = h
    trackF.AnchorPoint = Vector2.new(0, 0.5)
    trackF.BackgroundColor3 = Color3.fromRGB(60, 12, 24)
    trackF.BorderSizePixel = 0
    trackF.Position = UDim2.new(0, SLIDER_X, 0.5, 0)
    trackF.Size = UDim2.new(0, SLIDER_W, 0, 4)
    trackF.ZIndex = 4
    corner(trackF, 3)

    local fill = Instance.new("Frame")
    fill.Parent = trackF
    fill.BackgroundColor3 = PINK
    fill.BorderSizePixel = 0
    fill.Size = UDim2.new(pct, 0, 1, 0)
    fill.ZIndex = 5
    corner(fill, 3)

    local knob = Instance.new("Frame")
    knob.Parent = trackF
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = Color3.fromRGB(255, 220, 230)
    knob.BorderSizePixel = 0
    knob.Position = UDim2.new(pct, 0, 0.5, 0)
    knob.Size = UDim2.new(0, 10, 0, 10)
    knob.ZIndex = 6
    corner(knob, 3)
    local ks = Instance.new("UIStroke")
    ks.Color = HOT_PINK
    ks.Thickness = 1
    ks.Parent = knob

    local hit = Instance.new("TextButton")
    hit.Parent = h
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.AutoButtonColor = false
    hit.Position = UDim2.new(0, SLIDER_X - 5, 0, 0)
    hit.Size = UDim2.new(0, SLIDER_W + 10, 1, 0)
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
    local h = makeRow(parent, 22)
    makeLabel(h, label)

    local btn = Instance.new("TextButton")
    btn.Parent = h
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, -2, 0.5, 0)
    btn.Size = UDim2.new(0, 110, 1, 0)
    btn.BackgroundColor3 = BG_ROW
    btn.BackgroundTransparency = 0
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = FONT
    btn.Text = tostring(getKey()):gsub("Enum.KeyCode.", ""):gsub("Enum.UserInputType.", "")
    btn.TextColor3 = TEXT
    btn.TextSize = 12
    btn.ZIndex = 4
    corner(btn, 4)
    stroke(btn, STROKE, 1, 0.2)

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
        btn.Text = "Press any input..."
        btn.TextColor3 = PINK

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
    local h = makeRow(parent, 22)
    makeLabel(h, label)

    local btn = Instance.new("TextButton")
    btn.Parent = h
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, -2, 0.5, 0)
    btn.Size = UDim2.new(0, 110, 1, 0)
    btn.BackgroundColor3 = BG_ROW
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = FONT
    btn.Text = default .. "  v"
    btn.TextColor3 = TEXT
    btn.TextSize = 12
    btn.ZIndex = 4
    corner(btn, 4)
    stroke(btn, STROKE, 1, 0.2)

    local list = Instance.new("Frame")
    list.Parent = parent
    list.BackgroundColor3 = BG_LIST
    list.BorderSizePixel = 0
    list.Size = UDim2.new(1, -4, 0, #options * 20 + 8)
    list.Visible = false
    list.ZIndex = 20
    corner(list, 4)
    stroke(list, HOT_PINK, 1, 0.2)

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
        optBtn.BackgroundTransparency = 0
        optBtn.BorderSizePixel = 0
        optBtn.AutoButtonColor = false
        optBtn.Size = UDim2.new(1, 0, 0, 18)
        optBtn.Font = FONT
        optBtn.Text = opt
        optBtn.TextColor3 = TEXT
        optBtn.TextSize = 12
        optBtn.ZIndex = 21
        corner(optBtn, 3)

        optBtn.MouseEnter:Connect(function() optBtn.BackgroundColor3 = Color3.fromRGB(80, 12, 28) end)
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
    SilentEnabled      = false,
    SilentAlwaysOn     = false,
    SilentTargetPart   = "Head",
    SilentFOV          = 120,
    SilentHitChance    = 100,
    SilentVisibleCheck = false,
    SilentUseKey       = false,
    SilentKey          = Enum.KeyCode.Q,
    SilentShowFOV      = true,
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

    -- Hook Mouse's metatable __index — MM2 reads mouse.Hit for aim direction.
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
            warn("[N3X CrossHairx] Silent Aim: your executor doesn't support metatable hooks (getrawmetatable / hookfunction). Silent Aim will not work.")
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
        end    end))

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
-- SILENT AIM PAGE
-- ============================================================
do
    local sc = makeScroll(SilentPage)
    Section(sc, "SILENT AIM"); Divider(sc)

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
    Toggle(sc, "Show FOV Circle", true, function(on) settings.SilentShowFOV = on end)

    Divider(sc)
    Slider(sc, "FOV", 120, 5, 600, function(v) settings.SilentFOV = v end)
    Slider(sc, "Hit %", 100, 1, 100, function(v) settings.SilentHitChance = v end)

    Divider(sc)
    Section(sc, "PREDICTION"); Divider(sc)

    Toggle(sc, "Enable Prediction", true, function(on) settings.SilentPrediction = on end)
    Toggle(sc, "Ping Compensate", true, function(on) settings.SilentPingCompensate = on end)
    Slider(sc, "Predict X %", 160, 0, 300, function(v) settings.SilentPredictionX = v / 100 end)
    Slider(sc, "Predict Y %", 150, 0, 300, function(v) settings.SilentPredictionY = v / 100 end)

    Divider(sc)
    Section(sc, "TARGET"); Divider(sc)

    Dropdown(sc, "Part", { "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso", "Random", "Nearest" }, "Head", function(opt)
        settings.SilentTargetPart = opt
    end)

    Divider(sc)
    Section(sc, "CHECKS"); Divider(sc)

    Toggle(sc, "Visible Check", false, function(on) settings.SilentVisibleCheck = on end)
end

-- ============================================================
-- OPEN / CLOSE
-- ============================================================
local ToggleButton = Instance.new("TextButton")
ToggleButton.Parent = ScreenGui
ToggleButton.Position = UDim2.new(0, 20, 0, 20)
ToggleButton.Size = UDim2.new(0, 46, 0, 46)
ToggleButton.BackgroundColor3 = BG_HDR
ToggleButton.BorderSizePixel = 0
ToggleButton.AutoButtonColor = false
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Text = "⌖"
ToggleButton.TextColor3 = PINK
ToggleButton.TextSize = 26
ToggleButton.ZIndex = 50
ToggleButton.Active = true
corner(ToggleButton, 6)
stroke(ToggleButton, HOT_PINK, 1.5, 0.1)

local ToggleGlow = Instance.new("UIStroke")
ToggleGlow.Color = PINK
ToggleGlow.Thickness = 3
ToggleGlow.Transparency = 0.7
ToggleGlow.Parent = ToggleButton

ToggleButton.MouseEnter:Connect(function()
    TweenService:Create(ToggleButton, TweenInfo.new(0.12), { BackgroundColor3 = Color3.fromRGB(90, 10, 26) }):Play()
end)
ToggleButton.MouseLeave:Connect(function()
    TweenService:Create(ToggleButton, TweenInfo.new(0.12), { BackgroundColor3 = BG_HDR }):Play()
end)
ToggleButton.MouseButton1Click:Connect(function()
    Frame.Visible = not Frame.Visible
end)

keep(UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        Frame.Visible = not Frame.Visible
    end
end))

-- ============================================================
-- DRAG
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

print("[N3X CrossHairx] Silent Aim loaded — re-executing will NOT remove it.")
print("[N3X CrossHairx] Silent Aim uses Mouse.Hit hook (MM2 compatible).")
print("[N3X CrossHairx] If Silent Aim doesn't work, your executor lacks getrawmetatable/hookfunction.")
