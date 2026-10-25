--[[
    WAFFLE SCRIPT HUB v2
    Tabs: Player, Visuals, Teleport, Misc, Settings
    Tap the waffle button to show/hide. RightShift also works on PC.
    (No key screen - the Work.ink Lua Key System protects this script.)
]]

local HUB_NAME = "Waffle Script Hub"
local VERSION  = "v2.0"

-- ==============================
-- Stop duplicates if you run it twice
-- ==============================
local env = (getgenv and getgenv()) or _G
if env.WaffleHubUnload then pcall(env.WaffleHubUnload) end

-- ==============================
-- Services
-- ==============================
local Players         = game:GetService("Players")
local UIS             = game:GetService("UserInputService")
local RunService      = game:GetService("RunService")
local TweenService    = game:GetService("TweenService")
local HttpService     = game:GetService("HttpService")
local Lighting        = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local VirtualUser     = game:GetService("VirtualUser")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ==============================
-- Theme
-- ==============================
local THEME = {
    bg     = Color3.fromRGB(24, 24, 28),
    panel  = Color3.fromRGB(32, 32, 38),
    item   = Color3.fromRGB(42, 42, 50),
    track  = Color3.fromRGB(65, 65, 75),
    text   = Color3.fromRGB(240, 240, 245),
    sub    = Color3.fromRGB(160, 160, 172),
    accent = Color3.fromRGB(242, 178, 77),
    dark   = Color3.fromRGB(30, 20, 5),
    red    = Color3.fromRGB(220, 80, 80),
}

-- ==============================
-- Helpers
-- ==============================
local conns = {}
local function connect(signal, fn)
    local c = signal:Connect(fn)
    table.insert(conns, c)
    return c
end

local function new(class, props)
    local o = Instance.new(class)
    for k, v in pairs(props) do
        if k ~= "Parent" then o[k] = v end
    end
    if props.Parent then o.Parent = props.Parent end
    return o
end

local function corner(o, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 6)
    c.Parent = o
    return c
end

local function stroke(o, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color or THEME.track
    s.Thickness = thickness or 1
    s.Parent = o
    return s
end

local function getGuiParent()
    local ok, h = pcall(function() return gethui and gethui() end)
    if ok and h then return h end
    return player:WaitForChild("PlayerGui")
end

local gui = new("ScreenGui", {
    Name = "WaffleHubV2",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    Parent = getGuiParent(),
})

local function getHumanoid()
    local c = player.Character
    return c and c:FindFirstChildOfClass("Humanoid")
end

local function getRoot()
    local c = player.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

-- ==============================
-- Notifications
-- ==============================
local function notify(text)
    local n = new("TextLabel", {
        Size = UDim2.new(0, 250, 0, 34),
        Position = UDim2.new(0.5, -125, 0, -44),
        BackgroundColor3 = THEME.panel,
        TextColor3 = THEME.text,
        Text = text,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        BorderSizePixel = 0,
        ZIndex = 50,
        Parent = gui,
    })
    corner(n, 8)
    stroke(n, THEME.accent, 1)
    TweenService:Create(n, TweenInfo.new(0.25), { Position = UDim2.new(0.5, -125, 0, 14) }):Play()
    task.delay(2.4, function()
        if not n.Parent then return end
        local t = TweenService:Create(n, TweenInfo.new(0.25), { Position = UDim2.new(0.5, -125, 0, -44) })
        t:Play()
        t.Completed:Wait()
        n:Destroy()
    end)
end

-- ==============================
-- State
-- ==============================
local S = {
    speedOn = false, speed = 32,
    jumpOn = false, jump = 100,
    infJump = false,
    noclip = false,
    fly = false, flySpeed = 60,
    esp = false,
    fullbright = false,
    fovOn = false, fov = 70,
}

-- ==============================
-- Window
-- ==============================
local vp = camera.ViewportSize
local scaleValue = math.clamp(math.min(vp.X / 460, vp.Y / 340), 0.6, 1)

local win = new("Frame", {
    Name = "Window",
    Size = UDim2.new(0, 410, 0, 300),
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    BackgroundColor3 = THEME.bg,
    BorderSizePixel = 0,
    Parent = gui,
})
corner(win, 10)
stroke(win, THEME.track, 1)
local uiScale = new("UIScale", { Scale = scaleValue, Parent = win })

local titleBar = new("Frame", {
    Size = UDim2.new(1, 0, 0, 34),
    BackgroundColor3 = THEME.panel,
    BorderSizePixel = 0,
    Parent = win,
})
corner(titleBar, 10)
new("Frame", { -- square off the bottom corners of the title bar
    Size = UDim2.new(1, 0, 0, 10),
    Position = UDim2.new(0, 0, 1, -10),
    BackgroundColor3 = THEME.panel,
    BorderSizePixel = 0,
    Parent = titleBar,
})

new("TextLabel", {
    Size = UDim2.new(1, -90, 1, 0),
    Position = UDim2.new(0, 12, 0, 0),
    BackgroundTransparency = 1,
    Text = HUB_NAME .. "  " .. VERSION,
    TextColor3 = THEME.accent,
    Font = Enum.Font.GothamBold,
    TextSize = 14,
    TextXAlignment = Enum.TextXAlignment.Left,
    Parent = titleBar,
})

local function titleButton(text, xOffset, color, cb)
    local b = new("TextButton", {
        Size = UDim2.new(0, 26, 0, 22),
        Position = UDim2.new(1, xOffset, 0.5, -11),
        BackgroundColor3 = color,
        Text = text,
        TextColor3 = Color3.new(1, 1, 1),
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        BorderSizePixel = 0,
        Parent = titleBar,
    })
    corner(b, 6)
    b.MouseButton1Click:Connect(cb)
    return b
end

-- drag the window by its title bar
do
    local dragging, dragStart, startPos
    titleBar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = i.Position
            startPos = win.Position
        end
    end)
    connect(UIS.InputChanged, function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - dragStart
            win.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X / uiScale.Scale,
                startPos.Y.Scale, startPos.Y.Offset + d.Y / uiScale.Scale
            )
        end
    end)
    connect(UIS.InputEnded, function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- ==============================
-- Tabs
-- ==============================
local sidebar = new("Frame", {
    Size = UDim2.new(0, 92, 1, -34),
    Position = UDim2.new(0, 0, 0, 34),
    BackgroundTransparency = 1,
    Parent = win,
})
do
    local l = new("UIListLayout", { Padding = UDim.new(0, 4), Parent = sidebar })
    new("UIPadding", { PaddingTop = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 4), Parent = sidebar })
end

local pages, tabButtons = {}, {}

local function selectTab(name)
    for n, p in pairs(pages) do p.Visible = (n == name) end
    for n, b in pairs(tabButtons) do
        b.BackgroundColor3 = (n == name) and THEME.accent or THEME.item
        b.TextColor3 = (n == name) and THEME.dark or THEME.text
    end
end

local function createTab(name)
    local btn = new("TextButton", {
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = THEME.item,
        Text = name,
        TextColor3 = THEME.text,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        AutoButtonColor = false,
        BorderSizePixel = 0,
        Parent = sidebar,
    })
    corner(btn, 6)

    local page = new("ScrollingFrame", {
        Size = UDim2.new(1, -104, 1, -42),
        Position = UDim2.new(0, 98, 0, 40),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = THEME.accent,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new(),
        Visible = false,
        Parent = win,
    })
    new("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder, Parent = page })
    new("UIPadding", { PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6), Parent = page })

    pages[name] = page
    tabButtons[name] = btn
    btn.MouseButton1Click:Connect(function() selectTab(name) end)
    return page
end

-- ==============================
-- UI components
-- ==============================
local function addSection(page, text)
    return new("TextLabel", {
        Size = UDim2.new(1, 0, 0, 20),
        BackgroundTransparency = 1,
        Text = string.upper(text),
        TextColor3 = THEME.sub,
        Font = Enum.Font.GothamBold,
        TextSize = 11,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = page,
    })
end

local function addLabel(page, text)
    return new("TextLabel", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = THEME.sub,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = page,
    })
end

local function addButton(page, text, callback, inset)
    local b = new("TextButton", {
        Size = UDim2.new(1, inset or 0, 0, 32),
        BackgroundColor3 = THEME.item,
        Text = text,
        TextColor3 = THEME.text,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        AutoButtonColor = true,
        BorderSizePixel = 0,
        Parent = page,
    })
    corner(b, 6)
    b.MouseButton1Click:Connect(callback)
    return b
end

local function addToggle(page, text, default, callback)
    local state = default
    local btn = new("TextButton", {
        Size = UDim2.new(1, 0, 0, 34),
        BackgroundColor3 = THEME.item,
        Text = "",
        AutoButtonColor = false,
        BorderSizePixel = 0,
        Parent = page,
    })
    corner(btn, 6)
    new("TextLabel", {
        Size = UDim2.new(1, -60, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = THEME.text,
        Font = Enum.Font.Gotham,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = btn,
    })
    local pill = new("Frame", {
        Size = UDim2.new(0, 38, 0, 18),
        Position = UDim2.new(1, -48, 0.5, -9),
        BackgroundColor3 = THEME.track,
        BorderSizePixel = 0,
        Parent = btn,
    })
    corner(pill, 9)
    local knob = new("Frame", {
        Size = UDim2.new(0, 14, 0, 14),
        Position = UDim2.new(0, 2, 0.5, -7),
        BackgroundColor3 = Color3.new(1, 1, 1),
        BorderSizePixel = 0,
        Parent = pill,
    })
    corner(knob, 7)

    local function render()
        TweenService:Create(pill, TweenInfo.new(0.15), {
            BackgroundColor3 = state and THEME.accent or THEME.track
        }):Play()
        TweenService:Create(knob, TweenInfo.new(0.15), {
            Position = state and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)
        }):Play()
    end

    btn.MouseButton1Click:Connect(function()
        state = not state
        render()
        callback(state)
    end)
    render()

    return function(v)
        state = v
        render()
        callback(v)
    end
end

local function addSlider(page, text, min, max, default, callback)
    local holder = new("Frame", {
        Size = UDim2.new(1, 0, 0, 48),
        BackgroundColor3 = THEME.item,
        BorderSizePixel = 0,
        Parent = page,
    })
    corner(holder, 6)
    local label = new("TextLabel", {
        Size = UDim2.new(1, -16, 0, 20),
        Position = UDim2.new(0, 10, 0, 4),
        BackgroundTransparency = 1,
        Text = text .. ": " .. default,
        TextColor3 = THEME.text,
        Font = Enum.Font.Gotham,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = holder,
    })
    local track = new("Frame", {
        Size = UDim2.new(1, -20, 0, 6),
        Position = UDim2.new(0, 10, 0, 34),
        BackgroundColor3 = THEME.track,
        BorderSizePixel = 0,
        Parent = holder,
    })
    corner(track, 3)
    local fill = new("Frame", {
        Size = UDim2.new((default - min) / (max - min), 0, 1, 0),
        BackgroundColor3 = THEME.accent,
        BorderSizePixel = 0,
        Parent = track,
    })
    corner(fill, 3)

    local dragging = false
    local function setFromX(x)
        local rel = math.clamp((x - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        local val = math.floor(min + (max - min) * rel + 0.5)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        label.Text = text .. ": " .. val
        callback(val)
    end

    holder.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            setFromX(i.Position.X)
        end
    end)
    connect(UIS.InputChanged, function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            setFromX(i.Position.X)
        end
    end)
    connect(UIS.InputEnded, function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- ==============================
-- Features
-- ==============================

-- Fly
local flyBV, flyBG
local function stopFly()
    if flyBV then flyBV:Destroy() flyBV = nil end
    if flyBG then flyBG:Destroy() flyBG = nil end
    local hum = getHumanoid()
    if hum then hum.PlatformStand = false end
end

local function startFly()
    stopFly()
    local root, hum = getRoot(), getHumanoid()
    if not root or not hum then return end
    flyBV = new("BodyVelocity", {
        MaxForce = Vector3.new(1e5, 1e5, 1e5),
        Velocity = Vector3.zero,
        Parent = root,
    })
    flyBG = new("BodyGyro", {
        MaxTorque = Vector3.new(1e5, 1e5, 1e5),
        P = 1e4,
        CFrame = root.CFrame,
        Parent = root,
    })
    hum.PlatformStand = true
end

-- ESP
local espObjs, espLabels = {}, {}
local function clearESP()
    for _, o in pairs(espObjs) do pcall(function() o:Destroy() end) end
    espObjs, espLabels = {}, {}
end

local function refreshESP()
    clearESP()
    if not S.esp then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        local char = plr ~= player and plr.Character
        if char then
            local hl = new("Highlight", {
                FillColor = THEME.accent,
                OutlineColor = Color3.new(1, 1, 1),
                FillTransparency = 0.6,
                DepthMode = Enum.HighlightDepthMode.AlwaysOnTop,
                Parent = char,
            })
            table.insert(espObjs, hl)
            local head = char:FindFirstChild("Head")
            if head then
                local bb = new("BillboardGui", {
                    Size = UDim2.new(0, 120, 0, 28),
                    StudsOffset = Vector3.new(0, 2.5, 0),
                    AlwaysOnTop = true,
                    Parent = head,
                })
                local lbl = new("TextLabel", {
                    Size = UDim2.new(1, 0, 1, 0),
                    BackgroundTransparency = 1,
                    Text = plr.DisplayName,
                    TextColor3 = Color3.new(1, 1, 1),
                    TextStrokeTransparency = 0.4,
                    Font = Enum.Font.GothamBold,
                    TextSize = 13,
                    Parent = bb,
                })
                table.insert(espObjs, bb)
                espLabels[plr] = lbl
            end
        end
    end
end

local function hookPlayerESP(plr)
    if plr == player then return end
    connect(plr.CharacterAdded, function()
        task.wait(1)
        if S.esp then refreshESP() end
    end)
end

-- Fullbright
local savedLighting
local function setFullbright(on)
    if on then
        savedLighting = {
            Brightness = Lighting.Brightness,
            ClockTime = Lighting.ClockTime,
            FogEnd = Lighting.FogEnd,
            GlobalShadows = Lighting.GlobalShadows,
            Ambient = Lighting.Ambient,
            OutdoorAmbient = Lighting.OutdoorAmbient,
        }
        Lighting.Brightness = 2
        Lighting.ClockTime = 14
        Lighting.FogEnd = 1e6
        Lighting.GlobalShadows = false
        Lighting.Ambient = Color3.fromRGB(200, 200, 200)
        Lighting.OutdoorAmbient = Color3.fromRGB(200, 200, 200)
    elseif savedLighting then
        for k, v in pairs(savedLighting) do Lighting[k] = v end
    end
end

-- Misc functions
local function serverHop()
    notify("Searching for a server...")
    task.spawn(function()
        local ok, res = pcall(function()
            return game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
        end)
        if not ok then notify("Server hop failed") return end
        local data
        pcall(function() data = HttpService:JSONDecode(res) end)
        if data and data.data then
            for _, s in ipairs(data.data) do
                if s.id ~= game.JobId and s.playing < s.maxPlayers then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, player)
                    return
                end
            end
        end
        notify("No other server found")
    end)
end

local function fpsBoost()
    notify("FPS boost on")
    task.spawn(function()
        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
        Lighting.GlobalShadows = false
        for i, v in ipairs(workspace:GetDescendants()) do
            pcall(function()
                if v:IsA("BasePart") then
                    v.Material = Enum.Material.Plastic
                    v.Reflectance = 0
                elseif v:IsA("Decal") or v:IsA("Texture") then
                    v.Transparency = 1
                elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                    v.Enabled = false
                end
            end)
            if i % 300 == 0 then task.wait() end
        end
    end)
end

-- ==============================
-- Build tabs
-- ==============================
local pPlayer   = createTab("Player")
local pVisuals  = createTab("Visuals")
local pTeleport = createTab("Teleport")
local pMisc     = createTab("Misc")
local pSettings = createTab("Settings")

-- PLAYER
addSection(pPlayer, "Movement")
addToggle(pPlayer, "Custom speed", false, function(v) S.speedOn = v end)
addSlider(pPlayer, "Speed", 16, 200, S.speed, function(v) S.speed = v end)
addToggle(pPlayer, "Custom jump", false, function(v) S.jumpOn = v end)
addSlider(pPlayer, "Jump power", 50, 300, S.jump, function(v) S.jump = v end)
addToggle(pPlayer, "Infinite jump", false, function(v) S.infJump = v end)
addToggle(pPlayer, "Noclip", false, function(v) S.noclip = v end)
local setFly = addToggle(pPlayer, "Fly", false, function(v)
    S.fly = v
    if v then startFly() else stopFly() end
end)
addSlider(pPlayer, "Fly speed", 20, 250, S.flySpeed, function(v) S.flySpeed = v end)
addSection(pPlayer, "Character")
addButton(pPlayer, "Reset character", function()
    local hum = getHumanoid()
    if hum then hum.Health = 0 end
end)

-- VISUALS
addSection(pVisuals, "Players")
addToggle(pVisuals, "Player ESP", false, function(v)
    S.esp = v
    refreshESP()
end)
addSection(pVisuals, "World")
addToggle(pVisuals, "Fullbright", false, function(v)
    S.fullbright = v
    setFullbright(v)
end)
addToggle(pVisuals, "Custom field of view", false, function(v)
    S.fovOn = v
    if not v then camera.FieldOfView = 70 end
end)
addSlider(pVisuals, "FOV", 30, 120, S.fov, function(v) S.fov = v end)

-- TELEPORT
addSection(pTeleport, "Teleport to player")
local tpHolder = new("Frame", {
    Size = UDim2.new(1, 0, 0, 0),
    AutomaticSize = Enum.AutomaticSize.Y,
    BackgroundTransparency = 1,
    Parent = pTeleport,
})
new("UIListLayout", { Padding = UDim.new(0, 4), Parent = tpHolder })

local function refreshTeleportList()
    for _, c in ipairs(tpHolder:GetChildren()) do
        if c:IsA("TextButton") then c:Destroy() end
    end
    local count = 0
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player then
            count += 1
            addButton(tpHolder, plr.DisplayName .. "  (@" .. plr.Name .. ")", function()
                local myRoot = getRoot()
                local theirRoot = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                if myRoot and theirRoot then
                    myRoot.CFrame = theirRoot.CFrame * CFrame.new(0, 0, 3)
                    notify("Teleported to " .. plr.DisplayName)
                else
                    notify("Player not loaded")
                end
            end, 0)
        end
    end
    if count == 0 then
        addButton(tpHolder, "No other players", function() end, 0)
    end
end
addButton(pTeleport, "Refresh list", refreshTeleportList)
-- keep the refresh button above the list
tpHolder.LayoutOrder = 10
refreshTeleportList()

-- MISC
addSection(pMisc, "Utility")
local antiAfk = false
addToggle(pMisc, "Anti-AFK", false, function(v) antiAfk = v end)
connect(player.Idled, function()
    if antiAfk then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)
addButton(pMisc, "FPS boost", fpsBoost)
addSection(pMisc, "Server")
addButton(pMisc, "Rejoin server", function()
    notify("Rejoining...")
    TeleportService:Teleport(game.PlaceId, player)
end)
addButton(pMisc, "Server hop", serverHop)
addButton(pMisc, "Copy server ID", function()
    local ok = pcall(function() setclipboard(game.JobId) end)
    notify(ok and "Server ID copied" or "Copy not supported")
end)

-- SETTINGS (unload is assigned below)
local unload
addSection(pSettings, "Interface")
addLabel(pSettings, "Tap the waffle button to hide or show the hub. On PC you can also press RightShift.")
addSlider(pSettings, "Menu size %", 60, 120, math.floor(scaleValue * 100), function(v)
    uiScale.Scale = v / 100
end)
addSection(pSettings, "Danger zone")
addButton(pSettings, "Unload hub", function() unload() end)
addLabel(pSettings, HUB_NAME .. " " .. VERSION)

selectTab("Player")

-- ==============================
-- Title bar buttons
-- ==============================
local function toggleWindow()
    win.Visible = not win.Visible
end
titleButton("-", -62, THEME.track, function() win.Visible = false end)
titleButton("x", -32, THEME.red, function() unload() end)

-- ==============================
-- Waffle button (open / close)
-- ==============================
local logo = new("TextButton", {
    Name = "WaffleLogo",
    Size = UDim2.new(0, 56, 0, 56),
    Position = UDim2.new(0, 15, 0.5, -28),
    Text = "",
    AutoButtonColor = false,
    BackgroundColor3 = Color3.fromRGB(242, 178, 77),
    BorderSizePixel = 0,
    Active = true,
    Draggable = true,
    ZIndex = 10,
    Parent = gui,
})
corner(logo, 12)
stroke(logo, Color3.fromRGB(168, 102, 26), 2)
for row = 0, 2 do
    for col = 0, 2 do
        local p = new("Frame", {
            Size = UDim2.new(0, 12, 0, 12),
            Position = UDim2.new(0, 6 + col * 16, 0, 6 + row * 16),
            BackgroundColor3 = Color3.fromRGB(214, 138, 42),
            BorderSizePixel = 0,
            ZIndex = 11,
            Parent = logo,
        })
        corner(p, 3)
    end
end
local butter = new("Frame", {
    Size = UDim2.new(0, 14, 0, 10),
    Position = UDim2.new(0.5, -7, 0.5, -5),
    BackgroundColor3 = Color3.fromRGB(255, 229, 138),
    BorderSizePixel = 0,
    ZIndex = 12,
    Parent = logo,
})
corner(butter, 2)

do
    local startPos
    logo.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            startPos = logo.AbsolutePosition
        end
    end)
    logo.MouseButton1Click:Connect(function()
        if startPos and (logo.AbsolutePosition - startPos).Magnitude > 6 then return end
        toggleWindow()
    end)
end

connect(UIS.InputBegan, function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then toggleWindow() end
end)

-- ==============================
-- Main loops
-- ==============================
connect(RunService.Heartbeat, function()
    local hum = getHumanoid()
    if hum then
        if S.speedOn then hum.WalkSpeed = S.speed end
        if S.jumpOn then
            hum.UseJumpPower = true
            hum.JumpPower = S.jump
        end
    end

    if S.fly and flyBV and flyBG and hum then
        local cam = workspace.CurrentCamera
        local rel = cam.CFrame:VectorToObjectSpace(hum.MoveDirection)
        local vel = (cam.CFrame.RightVector * rel.X + cam.CFrame.LookVector * -rel.Z) * S.flySpeed
        flyBV.Velocity = vel
        flyBG.CFrame = cam.CFrame
    end
end)

connect(RunService.Stepped, function()
    if S.noclip and player.Character then
        for _, part in ipairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

connect(RunService.RenderStepped, function()
    if S.fovOn then workspace.CurrentCamera.FieldOfView = S.fov end
end)

connect(UIS.JumpRequest, function()
    if S.infJump then
        local hum = getHumanoid()
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- update ESP distance text twice a second
task.spawn(function()
    while gui.Parent do
        task.wait(0.5)
        if S.esp then
            local myRoot = getRoot()
            for plr, lbl in pairs(espLabels) do
                local r = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                if myRoot and r and lbl.Parent then
                    lbl.Text = plr.DisplayName .. " [" .. math.floor((r.Position - myRoot.Position).Magnitude) .. "]"
                end
            end
        end
    end
end)

-- players joining / leaving
for _, plr in ipairs(Players:GetPlayers()) do hookPlayerESP(plr) end
connect(Players.PlayerAdded, function(plr)
    hookPlayerESP(plr)
    refreshTeleportList()
end)
connect(Players.PlayerRemoving, function()
    task.wait(0.2)
    refreshTeleportList()
    if S.esp then refreshESP() end
end)

-- respawn handling
connect(player.CharacterAdded, function()
    task.wait(0.5)
    if S.fly then setFly(false) end
end)

-- ==============================
-- Unload
-- ==============================
unload = function()
    S.fly = false
    S.esp = false
    pcall(stopFly)
    pcall(clearESP)
    if S.fullbright then pcall(setFullbright, false) end
    if S.fovOn then camera.FieldOfView = 70 end
    for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
    gui:Destroy()
    env.WaffleHubUnload = nil
end
env.WaffleHubUnload = unload

notify("Waffle Hub loaded")
