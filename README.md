    if _G.RivalHubRunning then
    warn("[Rival Hub] Instancia previa detectada. Se recomienda rejoin si hay comportamiento errático.")
end
_G.RivalHubRunning = false
_G.__RivalHubMainOriginalPos = nil

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local HS = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

local LP = Players.LocalPlayer
if not LP then
    local _lpConn
    _lpConn = Players.PlayerAdded:Connect(function(p) LP = p end)
    local _t0 = tick()
    while not LP and tick() - _t0 < 15 do task.wait(0.1) end
    if _lpConn then _lpConn:Disconnect() end
end
if not LP then
    warn("[Rival Hub] LocalPlayer no disponible en 15s. Abortando.")
    return
end

local pgui = LP:WaitForChild("PlayerGui", 20)
if not pgui then
    warn("[Rival Hub] PlayerGui no cargó en 20s. Abortando.")
    return
end

local camera = Workspace.CurrentCamera

local SoundService = game:GetService("SoundService")


_G.RivalHubRunning = true
_G.RivalHubSession = (_G.RivalHubSession or 0) + 1
local _mySession = _G.RivalHubSession

-- Intro de execução removida conforme solicitado.

_G.__RivalOpenPingLagger = function()
-- https://discord.gg/SJnJsPZnW

if getgenv().SpeexPingLoaded then return end
getgenv().SpeexPingLoaded = true

local Players      = game:GetService("Players")
local CoreGui      = game:GetService("CoreGui")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local HttpService  = game:GetService("HttpService")
local player       = Players.LocalPlayer

local COL_BG      = Color3.fromRGB(252, 234, 234)
local COL_BG_TOP  = Color3.fromRGB(255, 240, 240)
local COL_ACCENT  = Color3.fromRGB(245, 140, 140)
local COL_ACCENT2 = Color3.fromRGB(240, 120, 120)
local COL_TEXT    = Color3.fromRGB(205, 110, 110)
local COL_STROKE  = Color3.fromRGB(255, 255, 255)
local COL_GREEN   = Color3.fromRGB(60, 200, 130)
local COL_RED     = Color3.fromRGB(200, 90, 100)
local COL_GREY    = Color3.fromRGB(180, 180, 180)

local CONFIG_FILE = "SpeexPingLagger_Config.json"
local DEFAULT_CFG = {
    power        = 100000,
    delay        = 0.125,
    keybind      = "F",
    autoBrainrot = true,
}
local cfg = {
    power        = DEFAULT_CFG.power,
    delay        = DEFAULT_CFG.delay,
    keybind      = DEFAULT_CFG.keybind,
    autoBrainrot = DEFAULT_CFG.autoBrainrot,
}

local function saveConfig()
    local ok, encoded = pcall(function() return HttpService:JSONEncode(cfg) end)
    if ok and encoded and writefile then pcall(writefile, CONFIG_FILE, encoded) end
end

local function loadConfig()
    if not (isfile and readfile and isfile(CONFIG_FILE)) then return end
    local ok, data = pcall(function() return HttpService:JSONDecode(readfile(CONFIG_FILE)) end)
    if not ok or type(data) ~= "table" then return end
    cfg.power        = tonumber(data.power) or DEFAULT_CFG.power
    cfg.delay        = tonumber(data.delay) or DEFAULT_CFG.delay
    cfg.keybind      = type(data.keybind) == "string" and data.keybind or DEFAULT_CFG.keybind
    cfg.autoBrainrot = type(data.autoBrainrot) == "boolean" and data.autoBrainrot or DEFAULT_CFG.autoBrainrot
end
loadConfig()

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeexPingMenu"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.DisplayOrder = 999
local okParent = pcall(function()
    if gethui then screenGui.Parent = gethui() else screenGui.Parent = CoreGui end
end)
if not okParent then screenGui.Parent = player:WaitForChild("PlayerGui") end

local mainFrame = Instance.new("Frame")
mainFrame.Name = "Main"
mainFrame.Size = UDim2.new(0, 320, 0, 363)
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -120)
mainFrame.BackgroundColor3 = COL_BG
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local grad = Instance.new("UIGradient")
grad.Rotation = 90
grad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, COL_BG_TOP),
    ColorSequenceKeypoint.new(1, COL_BG)
}
grad.Parent = mainFrame

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 18)
corner.Parent = mainFrame

local stroke = Instance.new("UIStroke")
stroke.Color = COL_STROKE
stroke.Thickness = 2
stroke.Transparency = 0.3
stroke.Parent = mainFrame

local dragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

local uiScale = Instance.new("UIScale")
uiScale.Scale = 0.5
uiScale.Parent = mainFrame

local function createLabel(text, size, pos, parent)
    local label = Instance.new("TextLabel")
    label.Size = size
    label.Position = pos
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = COL_TEXT
    label.Font = Enum.Font.GothamBold
    label.TextSize = 18
    label.Parent = parent
    return label
end

local function createButton(text, pos, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.8, 0, 0, 42)
    btn.Position = pos
    btn.BackgroundColor3 = COL_ACCENT
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Parent = parent
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 10); c.Parent = btn
    local g = Instance.new("UIGradient")
    g.Rotation = 90
    g.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 170, 170)),
        ColorSequenceKeypoint.new(1, COL_ACCENT2)
    }
    g.Parent = btn
    return btn
end

local title = createLabel("kira.vs", UDim2.new(0, 130, 0, 26), UDim2.new(0, 18, 0, 12), mainFrame)
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(230, 100, 100)

local subtitle = createLabel("speed bypass + ping lagger", UDim2.new(1, -70, 0, 18), UDim2.new(0, 18, 0, 36), mainFrame)
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.TextSize = 12
subtitle.Font = Enum.Font.Gotham
subtitle.TextColor3 = Color3.fromRGB(225, 140, 140)

local hideBtn = createLabel("Hide", UDim2.new(0, 60, 0, 30), UDim2.new(1, -70, 0, 12), mainFrame)
hideBtn.TextXAlignment = Enum.TextXAlignment.Right
hideBtn.TextSize = 14
hideBtn.Font = Enum.Font.Gotham

local clickableHide = Instance.new("TextButton")
clickableHide.Size = UDim2.new(0, 60, 0, 30)
clickableHide.Position = UDim2.new(1, -70, 0, 12)
clickableHide.BackgroundTransparency = 1
clickableHide.Text = ""
clickableHide.Parent = mainFrame

local MIN_SCALE, MAX_SCALE, SCALE_STEP = 0.5, 1.5, 0.1
local updateScaleUI

local function createScaleBtn(text, x)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 24, 0, 24)
    b.Position = UDim2.new(0, x, 0, 12)
    b.BackgroundColor3 = COL_ACCENT
    b.Text = text
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 16
    b.BorderSizePixel = 0
    b.AutoButtonColor = false
    b.Parent = mainFrame
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = b
    return b
end

local minusBtn = createScaleBtn("-", 150)
local plusBtn  = createScaleBtn("+", 220)
local scaleLabel = createLabel("100%", UDim2.new(0, 44, 0, 24), UDim2.new(0, 175, 0, 12), mainFrame)
scaleLabel.TextSize = 13
scaleLabel.Font = Enum.Font.Gotham

local function changeScale(delta)
    local newScale = math.clamp(uiScale.Scale + delta, MIN_SCALE, MAX_SCALE)
    uiScale.Scale = newScale
    scaleLabel.Text = math.floor(newScale * 100 + 0.5) .. "%"
    if updateScaleUI then updateScaleUI(newScale) end
end
minusBtn.MouseButton1Click:Connect(function() changeScale(-SCALE_STEP) end)
plusBtn.MouseButton1Click:Connect(function() changeScale(SCALE_STEP) end)

local currentVersion = "v1"

local versionLabel = createLabel("Version:", UDim2.new(0, 60, 0, 22), UDim2.new(0, 18, 0, 60), mainFrame)
versionLabel.TextSize = 13
versionLabel.Font = Enum.Font.Gotham
versionLabel.TextXAlignment = Enum.TextXAlignment.Left

local function createVersionBtn(text, x)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0, 52, 0, 24)
    b.Position = UDim2.new(0, x, 0, 58)
    b.BackgroundColor3 = COL_ACCENT
    b.Text = text
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BorderSizePixel = 0
    b.AutoButtonColor = false
    b.Parent = mainFrame
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = b
    return b
end

local v1Btn = createVersionBtn("V1", 80)
local v2Btn = createVersionBtn("V2", 140)

local function applyVersionUI()
    if currentVersion == "v1" then
        v1Btn.BackgroundColor3 = COL_GREEN
        v2Btn.BackgroundColor3 = COL_ACCENT
    else
        v1Btn.BackgroundColor3 = COL_ACCENT
        v2Btn.BackgroundColor3 = COL_GREEN
    end
end
applyVersionUI()

local profileCard = Instance.new("Frame")
profileCard.Size = UDim2.new(0.84, 0, 0, 68)
profileCard.Position = UDim2.new(0.08, 0, 0, 90)
profileCard.BackgroundColor3 = COL_ACCENT
profileCard.BackgroundTransparency = 0.55
profileCard.BorderSizePixel = 0
profileCard.Parent = mainFrame
local pcCorner = Instance.new("UICorner"); pcCorner.CornerRadius = UDim.new(0, 12); pcCorner.Parent = profileCard

local avatar = Instance.new("ImageLabel")
avatar.Size = UDim2.new(0, 52, 0, 52)
avatar.Position = UDim2.new(0, 8, 0.5, -26)
avatar.BackgroundColor3 = COL_BG_TOP
avatar.BorderSizePixel = 0
avatar.Parent = profileCard
local avCorner = Instance.new("UICorner"); avCorner.CornerRadius = UDim.new(1, 0); avCorner.Parent = avatar
local avStroke = Instance.new("UIStroke")
avStroke.Color = Color3.fromRGB(255, 255, 255)
avStroke.Thickness = 2
avStroke.Transparency = 0.2
avStroke.Parent = avatar

task.spawn(function()
    local ok, img = pcall(function()
        return Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100)
    end)
    if ok and img then avatar.Image = img end
end)

local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(1, -76, 0, 24)
nameLabel.Position = UDim2.new(0, 68, 0, 8)
nameLabel.BackgroundTransparency = 1
nameLabel.Text = string.upper(player.DisplayName)
nameLabel.TextColor3 = Color3.fromRGB(170, 70, 70)
nameLabel.Font = Enum.Font.GothamBold
nameLabel.TextSize = 17
nameLabel.TextXAlignment = Enum.TextXAlignment.Left
nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
nameLabel.Parent = profileCard

local subLabel = Instance.new("TextLabel")
subLabel.Size = UDim2.new(1, -76, 0, 20)
subLabel.Position = UDim2.new(0, 68, 0, 36)
subLabel.BackgroundTransparency = 1
subLabel.Text = "@" .. player.Name .. "  ·  ID: " .. player.UserId
subLabel.TextColor3 = Color3.fromRGB(200, 110, 110)
subLabel.Font = Enum.Font.Gotham
subLabel.TextSize = 12
subLabel.TextXAlignment = Enum.TextXAlignment.Left
subLabel.TextTruncate = Enum.TextTruncate.AtEnd
subLabel.Parent = profileCard

local discordLabel = Instance.new("TextLabel")
discordLabel.Size = UDim2.new(0.84, 0, 0, 22)
discordLabel.Position = UDim2.new(0.08, 0, 0, 326)
discordLabel.BackgroundTransparency = 1
discordLabel.Text = "discord.gg/SJnJsPZnW"
discordLabel.TextColor3 = Color3.fromRGB(195, 90, 90)
discordLabel.Font = Enum.Font.GothamBold
discordLabel.TextSize = 13
discordLabel.Parent = mainFrame

local discordLine = Instance.new("Frame")
discordLine.Size = UDim2.new(0, 150, 0, 2)
discordLine.Position = UDim2.new(0.5, -75, 0, 352)
discordLine.BackgroundColor3 = COL_ACCENT
discordLine.BorderSizePixel = 0
discordLine.Parent = mainFrame
local dlCorner = Instance.new("UICorner"); dlCorner.CornerRadius = UDim.new(0, 2); dlCorner.Parent = discordLine

local changeGuiBtn = createButton("Change GUI", UDim2.new(0.1, 0, 0, 178), mainFrame)
local statusBtn = createButton("DISABLED", UDim2.new(0.1, 0, 0, 238), mainFrame)

local popup = Instance.new("Frame")
popup.Size = UDim2.new(0, 260, 0, 330)
popup.Position = UDim2.new(0.5, -130, 0.5, -140)
popup.BackgroundColor3 = COL_BG
popup.BorderSizePixel = 0
popup.Visible = false
popup.Active = true
popup.Parent = screenGui

local popupScale = Instance.new("UIScale")
popupScale.Scale = 0.5
popupScale.Parent = popup

local pDragging, pDragStart, pStartPos = false, nil, nil
popup.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        pDragging = true
        pDragStart = input.Position
        pStartPos = popup.Position
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        pDragging = false
    end
end)
UIS.InputChanged:Connect(function(input)
    if pDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - pDragStart
        popup.Position = UDim2.new(
            pStartPos.X.Scale, pStartPos.X.Offset + delta.X,
            pStartPos.Y.Scale, pStartPos.Y.Offset + delta.Y
        )
    end
end)

local popupGrad = Instance.new("UIGradient")
popupGrad.Rotation = 90
popupGrad.Color = grad.Color
popupGrad.Parent = popup

local popupCorner = Instance.new("UICorner")
popupCorner.CornerRadius = UDim.new(0, 18)
popupCorner.Parent = popup

local popupStroke = Instance.new("UIStroke")
popupStroke.Color = COL_STROKE
popupStroke.Thickness = 2
popupStroke.Transparency = 0.3
popupStroke.Parent = popup

local popupTitle = createLabel("Change GUI", UDim2.new(1, -50, 0, 30), UDim2.new(0, 16, 0, 12), popup)
popupTitle.TextXAlignment = Enum.TextXAlignment.Left

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -38, 0, 10)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "X"
closeBtn.TextColor3 = COL_TEXT
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.Parent = popup
closeBtn.MouseButton1Click:Connect(function() popup.Visible = false end)

local sizeLabel = createLabel("GUI Size:", UDim2.new(0.55, 0, 0, 24), UDim2.new(0, 16, 0, 55), popup)
sizeLabel.TextXAlignment = Enum.TextXAlignment.Left
sizeLabel.TextSize = 14
sizeLabel.Font = Enum.Font.Gotham

local sizeValueLabel = createLabel("0.5x", UDim2.new(0, 60, 0, 24), UDim2.new(1, -76, 0, 55), popup)
sizeValueLabel.TextXAlignment = Enum.TextXAlignment.Right
sizeValueLabel.TextSize = 14
sizeValueLabel.Font = Enum.Font.Gotham

local TRACK_W = 228
local track = Instance.new("Frame")
track.Size = UDim2.new(0, TRACK_W, 0, 8)
track.Position = UDim2.new(0, 16, 0, 90)
track.BackgroundColor3 = COL_BG_TOP
track.BorderSizePixel = 0
track.Parent = popup
local trackCorner = Instance.new("UICorner"); trackCorner.CornerRadius = UDim.new(0, 4); trackCorner.Parent = track

local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 18, 0, 18)
knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
knob.BorderSizePixel = 0
knob.Parent = track
local knobCorner = Instance.new("UICorner"); knobCorner.CornerRadius = UDim.new(1, 0); knobCorner.Parent = knob
local knobStroke = Instance.new("UIStroke")
knobStroke.Color = COL_ACCENT2
knobStroke.Thickness = 2
knobStroke.Parent = knob

updateScaleUI = function(s)
    local t = (s - MIN_SCALE) / (MAX_SCALE - MIN_SCALE)
    knob.Position = UDim2.new(0, t * (TRACK_W - 18), 0.5, -9)
    scaleLabel.Text = math.floor(s * 100 + 0.5) .. "%"
    sizeValueLabel.Text = string.format("%.1fx", s)
    popupScale.Scale = s
end

local draggingSlider = false
local function setScaleFromInput(input)
    local x = input.Position.X - track.AbsolutePosition.X
    local t = math.clamp(x / (TRACK_W - 18), 0, 1)
    local s = MIN_SCALE + t * (MAX_SCALE - MIN_SCALE)
    uiScale.Scale = s
    popupScale.Scale = s
    updateScaleUI(s)
end

track.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
        setScaleFromInput(input)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
    end
end)
UIS.InputChanged:Connect(function(input)
    if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        setScaleFromInput(input)
    end
end)

updateScaleUI(uiScale.Scale)

local v1PopupContent = {}
local v2PopupContent = {}

local function createRowGeneric(labelText, valueText, y, contentTable)
    local l = createLabel(labelText, UDim2.new(0.55, 0, 0, 26), UDim2.new(0, 16, 0, y), popup)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.TextSize = 14
    l.Font = Enum.Font.Gotham
    table.insert(contentTable, l)

    local box = Instance.new("TextBox")
    box.Size = UDim2.new(0, 110, 0, 26)
    box.Position = UDim2.new(1, -126, 0, y)
    box.BackgroundColor3 = COL_BG_TOP
    box.Text = valueText
    box.TextColor3 = COL_TEXT
    box.Font = Enum.Font.GothamBold
    box.TextSize = 14
    box.BorderSizePixel = 0
    box.ClearTextOnFocus = false
    box.Parent = popup
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = box
    table.insert(contentTable, box)
    return box
end

local powerBox   = createRowGeneric("Power:", "100000", 120, v1PopupContent)
local delayBox   = createRowGeneric("Delay (secs):", "0.125", 155, v1PopupContent)
local keybindBox = createRowGeneric("Keybind:", "F", 190, v1PopupContent)

local speedEnabled = false
local speedToggle = Instance.new("TextButton")
speedToggle.Size = UDim2.new(0, 62, 0, 26)
speedToggle.Position = UDim2.new(1, -78, 0, 225)
speedToggle.BackgroundColor3 = COL_RED
speedToggle.Text = "OFF"
speedToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
speedToggle.Font = Enum.Font.GothamBold
speedToggle.TextSize = 14
speedToggle.BorderSizePixel = 0
speedToggle.AutoButtonColor = false
speedToggle.Parent = popup
local stCorner = Instance.new("UICorner"); stCorner.CornerRadius = UDim.new(0, 8); stCorner.Parent = speedToggle
table.insert(v1PopupContent, speedToggle)

local speedLabel = createLabel("Speed:", UDim2.new(0.55, 0, 0, 26), UDim2.new(0, 16, 0, 225), popup)
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.TextSize = 14
speedLabel.Font = Enum.Font.Gotham
table.insert(v1PopupContent, speedLabel)

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 60, 0, 26)
speedBox.Position = UDim2.new(1, -148, 0, 225)
speedBox.BackgroundColor3 = COL_BG_TOP
speedBox.Text = "49"
speedBox.TextColor3 = COL_TEXT
speedBox.Font = Enum.Font.GothamBold
speedBox.TextSize = 14
speedBox.BorderSizePixel = 0
speedBox.ClearTextOnFocus = false
speedBox.Parent = popup
local sbCorner = Instance.new("UICorner"); sbCorner.CornerRadius = UDim.new(0, 8); sbCorner.Parent = speedBox
table.insert(v1PopupContent, speedBox)

speedToggle.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    speedToggle.Text = speedEnabled and "ON" or "OFF"
    speedToggle.BackgroundColor3 = speedEnabled and COL_GREEN or COL_RED
end)

local autoBrainrot = false
local abLabel = createLabel("Auto Brainrot:", UDim2.new(0.55, 0, 0, 26), UDim2.new(0, 16, 0, 260), popup)
abLabel.TextXAlignment = Enum.TextXAlignment.Left
abLabel.TextSize = 14
abLabel.Font = Enum.Font.Gotham
table.insert(v1PopupContent, abLabel)

local abToggle = Instance.new("TextButton")
abToggle.Size = UDim2.new(0, 62, 0, 26)
abToggle.Position = UDim2.new(1, -78, 0, 260)
abToggle.BackgroundColor3 = COL_RED
abToggle.Text = "OFF"
abToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
abToggle.Font = Enum.Font.GothamBold
abToggle.TextSize = 14
abToggle.BorderSizePixel = 0
abToggle.AutoButtonColor = false
abToggle.Parent = popup
local abCorner = Instance.new("UICorner"); abCorner.CornerRadius = UDim.new(0, 8); abCorner.Parent = abToggle
table.insert(v1PopupContent, abToggle)

abToggle.MouseButton1Click:Connect(function()
    autoBrainrot = not autoBrainrot
    abToggle.Text = autoBrainrot and "ON" or "OFF"
    abToggle.BackgroundColor3 = autoBrainrot and COL_GREEN or COL_RED
end)

local v2PowerBox = createRowGeneric("Power:", tostring(cfg.power), 120, v2PopupContent)
v2PowerBox.PlaceholderText = "100000"
v2PowerBox.FocusLost:Connect(function()
    local num = tonumber(v2PowerBox.Text)
    if num and num >= 1 then cfg.power = math.floor(num) saveConfig() end
    v2PowerBox.Text = tostring(cfg.power)
end)

local v2DelayBox = createRowGeneric("Delay (secs):", string.format("%.3f", cfg.delay), 155, v2PopupContent)
v2DelayBox.PlaceholderText = "0.125"
v2DelayBox.FocusLost:Connect(function()
    local num = tonumber(v2DelayBox.Text)
    if num and num >= 0.01 then cfg.delay = num saveConfig() end
    v2DelayBox.Text = string.format("%.3f", cfg.delay)
end)

local v2KeybindBox = createRowGeneric("Keybind:", cfg.keybind, 190, v2PopupContent)
v2KeybindBox.FocusLost:Connect(function()
    local txt = v2KeybindBox.Text:upper()
    local ok, kc = pcall(function() return Enum.KeyCode[txt] end)
    if ok and kc then cfg.keybind = txt saveConfig() end
    v2KeybindBox.Text = cfg.keybind
end)

local v2AutoBrainrotLabel = createLabel("Auto Brainrot:", UDim2.new(0.55, 0, 0, 26), UDim2.new(0, 16, 0, 225), popup)
v2AutoBrainrotLabel.TextXAlignment = Enum.TextXAlignment.Left
v2AutoBrainrotLabel.TextSize = 14
v2AutoBrainrotLabel.Font = Enum.Font.Gotham
table.insert(v2PopupContent, v2AutoBrainrotLabel)

local v2AbToggle = Instance.new("TextButton")
v2AbToggle.Size = UDim2.new(0, 62, 0, 26)
v2AbToggle.Position = UDim2.new(1, -78, 0, 225)
v2AbToggle.BackgroundColor3 = cfg.autoBrainrot and COL_GREEN or COL_RED
v2AbToggle.Text = cfg.autoBrainrot and "ON" or "OFF"
v2AbToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
v2AbToggle.Font = Enum.Font.GothamBold
v2AbToggle.TextSize = 14
v2AbToggle.BorderSizePixel = 0
v2AbToggle.AutoButtonColor = false
v2AbToggle.Parent = popup
local v2AbCorner = Instance.new("UICorner"); v2AbCorner.CornerRadius = UDim.new(0, 8); v2AbCorner.Parent = v2AbToggle
table.insert(v2PopupContent, v2AbToggle)

v2AbToggle.MouseButton1Click:Connect(function()
    cfg.autoBrainrot = not cfg.autoBrainrot
    v2AbToggle.Text = cfg.autoBrainrot and "ON" or "OFF"
    v2AbToggle.BackgroundColor3 = cfg.autoBrainrot and COL_GREEN or COL_RED
    saveConfig()
    if not cfg.autoBrainrot then
        v2BrainrotMode = false
        v2LastBrainrotState = false
    end
end)

local function applyPopupVersionUI()
    local showV1 = (currentVersion == "v1")
    for _, el in ipairs(v1PopupContent) do
        if el and el.Parent then el.Visible = showV1 end
    end
    for _, el in ipairs(v2PopupContent) do
        if el and el.Parent then el.Visible = not showV1 end
    end
    popupTitle.Text = showV1 and "Change GUI · V1" or "Change GUI · V2 Ping"
end
applyPopupVersionUI()

local miniStatusBtn = Instance.new("TextButton")
miniStatusBtn.Size = UDim2.new(0.84, 0, 0, 32)
miniStatusBtn.Position = UDim2.new(0.08, 0, 0, 56)
miniStatusBtn.BackgroundColor3 = COL_ACCENT
miniStatusBtn.Text = "DISABLE"
miniStatusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
miniStatusBtn.Font = Enum.Font.GothamBold
miniStatusBtn.TextSize = 15
miniStatusBtn.BorderSizePixel = 0
miniStatusBtn.AutoButtonColor = false
miniStatusBtn.Visible = false
miniStatusBtn.Parent = mainFrame
local miniCorner = Instance.new("UICorner"); miniCorner.CornerRadius = UDim.new(0, 8); miniCorner.Parent = miniStatusBtn
local miniGrad = Instance.new("UIGradient")
miniGrad.Rotation = 90
miniGrad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 170, 170)),
    ColorSequenceKeypoint.new(1, COL_ACCENT2)
}
miniGrad.Parent = miniStatusBtn

local remote = nil

local function findRemote()
    local rrs = game:FindFirstChild("RobloxReplicatedStorage")
    if not rrs then return nil end
    for _, name in ipairs({"SetPlayerBlockList", "UpdatePlayerBlockList", "SetBlockList", "UpdateBlockList"}) do
        local found = rrs:FindFirstChild(name)
        if found and found:IsA("RemoteEvent") then return found end
    end
    for _, c in ipairs(rrs:GetChildren()) do
        if c:IsA("RemoteEvent") and c.Name:find("Block") then return c end
    end
    return nil
end

remote = findRemote()

local function buildPayload(power)
    local main = {}
    local nested = {{}}
    local current = nested[1]
    for _ = 1, 186 do
        local n = {}
        table.insert(current, n)
        current = n
    end
    local maxRep = math.min(math.floor(power / 188), 10000)
    for _ = 1, maxRep do
        table.insert(main, nested)
    end
    return main
end

local enabled = false

local function runPingLoop()
    local delay = tonumber(delayBox.Text) or 0.125
    while enabled and remote and currentVersion == "v1" do
        local power = tonumber(powerBox.Text) or 100000
        local payload = buildPayload(power)
        local ok = pcall(function() remote:FireServer(payload) end)
        if not ok then delay = math.min(delay * 1.5, 0.5)
        else delay = math.max(delay * 0.995, 0.05) end
        local nd = tonumber(delayBox.Text)
        if nd and nd > 0 then delay = math.max(nd, 0.01) end
        task.wait(delay)
    end
end

local function setEnabled(state)
    enabled = state
    if enabled then
        if not remote then remote = findRemote() end
        if not remote then
            enabled = false
            statusBtn.Text = "NO REMOTE"
            statusBtn.BackgroundColor3 = COL_RED
            return
        end
        statusBtn.Text = "ENABLED"
        statusBtn.BackgroundColor3 = COL_GREEN
        miniStatusBtn.Text = "ENABLE"
        miniStatusBtn.BackgroundColor3 = COL_GREEN
        miniGrad.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(110, 225, 165)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(45, 185, 115))
        }
        task.spawn(runPingLoop)
    else
        statusBtn.Text = "DISABLED"
        statusBtn.BackgroundColor3 = COL_ACCENT
        miniStatusBtn.Text = "DISABLE"
        miniStatusBtn.BackgroundColor3 = COL_ACCENT
        miniGrad.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 170, 170)),
            ColorSequenceKeypoint.new(1, COL_ACCENT2)
        }
    end
end

local v2Enabled = false
local v2BrainrotMode = false
local v2LastBrainrotState = false

local function v2RunPingLoop()
    while v2Enabled and currentVersion == "v2" do
        if not remote then remote = findRemote() end
        if not remote then
            v2Enabled = false
            statusBtn.Text = "NO REMOTE"
            statusBtn.BackgroundColor3 = COL_RED
            break
        end
        local power = tonumber(v2PowerBox.Text) or cfg.power
        local delay = tonumber(v2DelayBox.Text) or cfg.delay
        local payload = buildPayload(power)
        local ok = pcall(function() remote:FireServer(payload) end)
        if not ok then delay = math.min(delay * 1.5, 0.5) end
        task.wait(math.max(delay, 0.01))
    end
end

local function v2SetActive(state)
    v2Enabled = state
    if state then
        if not remote then remote = findRemote() end
        if not remote then
            v2Enabled = false
            statusBtn.Text = "NO REMOTE"
            statusBtn.BackgroundColor3 = COL_RED
            return
        end
        statusBtn.Text = "DEACTIVATE"
        statusBtn.BackgroundColor3 = COL_GREEN
        miniStatusBtn.Text = "DEACTIVATE"
        miniStatusBtn.BackgroundColor3 = COL_GREEN
        miniGrad.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(110, 225, 165)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(45, 185, 115))
        }
        task.spawn(v2RunPingLoop)
    else
        statusBtn.Text = "ACTIVATE"
        statusBtn.BackgroundColor3 = COL_ACCENT
        miniStatusBtn.Text = "ACTIVATE"
        miniStatusBtn.BackgroundColor3 = COL_ACCENT
        miniGrad.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(250, 170, 170)),
            ColorSequenceKeypoint.new(1, COL_ACCENT2)
        }
    end
end

statusBtn.MouseButton1Click:Connect(function()
    if currentVersion == "v1" then
        setEnabled(not enabled)
    else
        v2SetActive(not v2Enabled)
    end
end)

miniStatusBtn.MouseButton1Click:Connect(function()
    if currentVersion == "v1" then
        setEnabled(not enabled)
    else
        v2SetActive(not v2Enabled)
    end
end)

v1Btn.MouseButton1Click:Connect(function()
    if currentVersion == "v1" then return end
    v2SetActive(false)
    currentVersion = "v1"
    applyVersionUI()
    applyPopupVersionUI()
    setEnabled(false)
end)

v2Btn.MouseButton1Click:Connect(function()
    if currentVersion == "v2" then return end
    setEnabled(false)
    currentVersion = "v2"
    applyVersionUI()
    applyPopupVersionUI()
    v2SetActive(false)
end)

local lastMoveDir = Vector3.new(0,0,0)
local rng = Random.new()

RunService.RenderStepped:Connect(function()
    if currentVersion ~= "v1" then return end
    if not speedEnabled then return end
    if not enabled then return end

    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end

    local st = hum:GetState()
    if hum.PlatformStand
       or st == Enum.HumanoidStateType.Physics
       or st == Enum.HumanoidStateType.Ragdoll
       or st == Enum.HumanoidStateType.FallingDown
       or st == Enum.HumanoidStateType.Dead then
        return
    end

    local targetSpeed = tonumber(speedBox.Text) or 49
    if targetSpeed < 10 then targetSpeed = 10 end
    if targetSpeed > 500 then targetSpeed = 500 end

    local md = hum.MoveDirection
    if md.Magnitude > 0.05 then lastMoveDir = md end

    local dir = nil
    if md.Magnitude > 0.05 then
        dir = md
    elseif lastMoveDir.Magnitude > 0 then
        local keys = {Enum.KeyCode.W, Enum.KeyCode.A, Enum.KeyCode.S, Enum.KeyCode.D,
                      Enum.KeyCode.Up, Enum.KeyCode.Down, Enum.KeyCode.Left, Enum.KeyCode.Right}
        for _, k in ipairs(keys) do
            if UIS:IsKeyDown(k) then dir = lastMoveDir break end
        end
    end

    pcall(function() root:SetNetworkOwner(player) end)

    local j = rng:NextNumber(-0.1, 0.1)
    local vy = root.AssemblyLinearVelocity.Y

    if dir and dir.Magnitude > 0.05 then
        local u = dir.Unit
        root.AssemblyLinearVelocity = Vector3.new(
            u.X * targetSpeed + j,
            vy,
            u.Z * targetSpeed + j
        )
    else
        root.AssemblyLinearVelocity = Vector3.new(0, vy, 0)
    end
end)

RunService.Heartbeat:Connect(function()
    if currentVersion ~= "v1" then return end
    if not speedEnabled or not enabled then return end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and hum.WalkSpeed ~= 16 then hum.WalkSpeed = 16 end
end)

local lastBrainrotState = false
RunService.Heartbeat:Connect(function()
    if currentVersion ~= "v1" then lastBrainrotState = false return end
    if not autoBrainrot then lastBrainrotState = false return end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local slowed = hum.WalkSpeed < 25
    if slowed and not lastBrainrotState then
        lastBrainrotState = true
        if not enabled then setEnabled(true) end
    elseif not slowed and lastBrainrotState then
        lastBrainrotState = false
        if enabled then setEnabled(false) end
    end
end)

RunService.Heartbeat:Connect(function()
    if currentVersion ~= "v2" then v2LastBrainrotState = false return end
    if not cfg.autoBrainrot then v2LastBrainrotState = false return end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local slowed = hum.WalkSpeed < 25
    if slowed and not v2LastBrainrotState then
        v2LastBrainrotState = true
        if not v2Enabled then v2SetActive(true) end
    elseif not slowed and v2LastBrainrotState then
        v2LastBrainrotState = false
        if v2Enabled then v2SetActive(false) end
    end
end)

changeGuiBtn.MouseButton1Click:Connect(function()
    popup.Visible = not popup.Visible
end)

UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    local kc = input.KeyCode

    if currentVersion == "v1" then
        local keyName = keybindBox.Text:upper()
        local ok, keyCode = pcall(function() return Enum.KeyCode[keyName] end)
        if ok and keyCode and kc == keyCode then
            setEnabled(not enabled)
        end
    else
        local ok, keyCode = pcall(function() return Enum.KeyCode[cfg.keybind] end)
        if ok and keyCode and kc == keyCode then
            v2SetActive(not v2Enabled)
        end
    end
end)

local hidden = false
clickableHide.MouseButton1Click:Connect(function()
    hidden = not hidden
    if hidden then
        mainFrame:TweenSize(UDim2.new(0, 320, 0, 100), "Out", "Quad", 0.3, true)
        changeGuiBtn.Visible = false
        statusBtn.Visible = false
        subtitle.Visible = false
        profileCard.Visible = false
        discordLabel.Visible = false
        discordLine.Visible = false
        versionLabel.Visible = false
        v1Btn.Visible = false
        v2Btn.Visible = false
        miniStatusBtn.Visible = true
        hideBtn.Text = "Show"
    else
        mainFrame:TweenSize(UDim2.new(0, 320, 0, 363), "Out", "Quad", 0.3, true)
        miniStatusBtn.Visible = false
        task.delay(0.3, function()
            changeGuiBtn.Visible = true
            statusBtn.Visible = true
            subtitle.Visible = true
            profileCard.Visible = true
            discordLabel.Visible = true
            discordLine.Visible = true
            versionLabel.Visible = true
            v1Btn.Visible = true
            v2Btn.Visible = true
        end)
        hideBtn.Text = "Hide"
    end
end)

for _, btn in ipairs({changeGuiBtn, statusBtn, minusBtn, plusBtn, miniStatusBtn}) do
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = COL_ACCENT2 end)
    btn.MouseLeave:Connect(function()
        if btn == statusBtn then
            if currentVersion == "v1" and enabled then btn.BackgroundColor3 = COL_GREEN
            elseif currentVersion == "v2" and v2Enabled then btn.BackgroundColor3 = COL_GREEN
            else btn.BackgroundColor3 = COL_ACCENT end
        elseif btn == miniStatusBtn then
            if currentVersion == "v1" and enabled then btn.BackgroundColor3 = COL_GREEN
            elseif currentVersion == "v2" and v2Enabled then btn.BackgroundColor3 = COL_GREEN
            else btn.BackgroundColor3 = COL_ACCENT end
        else
            btn.BackgroundColor3 = COL_ACCENT
        end
    end)
end

end

local RIVALHUB_LANGUAGE = "en"

local RIVALHUB_TRANSLATIONS = {
    es = { ["RESET ALL SETTINGS"]="RESTABLECER TODO", ["SETTINGS RESET"]="AJUSTES RESTABLECIDOS", ["CONFIRM?"]="¿CONFIRMAR?", ["ERROR"]="ERROR", ["RESET"]="RESTABLECER", ["Keybinds"]="TECLAS", ["Carry Mode"]="MODO CARGAR", ["Lagger Mode"]="MODO LAG", ["Auto Left"]="AUTO IZQUIERDA", ["Auto Right"]="AUTO DERECHA", ["Auto Bat"]="AUTO BATE", ["TP Down"]="TP ABAJO", ["Drop Brainrot"]="SOLTAR BRAINROT", ["Hide GUI"]="OCULTAR MENÚ", ["Normal Speed"]="VELOCIDAD NORMAL", ["Carry Speed"]="VELOCIDAD CARGAR", ["Current Mode"]="MODO ACTUAL", ["ACTIVATE"]="ACTIVAR" },
    pt = { ["RESET ALL SETTINGS"]="REDEFINIR TUDO", ["SETTINGS RESET"]="CONFIGURAÇÕES REDEFINIDAS", ["CONFIRM?"]="CONFIRMAR?", ["ERROR"]="ERRO", ["RESET"]="REDEFINIR", ["Keybinds"]="TECLAS", ["Carry Mode"]="MODO CARREGAR", ["Auto Left"]="AUTO ESQUERDA", ["Auto Right"]="AUTO DIREITA", ["Hide GUI"]="OCULTAR MENU", ["Normal Speed"]="VELOCIDADE NORMAL", ["Carry Speed"]="VELOCIDADE CARREGAR", ["ACTIVATE"]="ATIVAR" },
    fr = { ["RESET ALL SETTINGS"]="RÉINITIALISER", ["SETTINGS RESET"]="PARAMÈTRES RÉINITIALISÉS", ["CONFIRM?"]="CONFIRMER ?", ["ERROR"]="ERREUR", ["RESET"]="RÉINITIALISER", ["Keybinds"]="RACCOURCIS", ["Carry Mode"]="MODE PORTER", ["Auto Left"]="AUTO GAUCHE", ["Auto Right"]="AUTO DROITE", ["Hide GUI"]="MASQUER LE MENU", ["Normal Speed"]="VITESSE NORMALE", ["Carry Speed"]="VITESSE PORTER", ["ACTIVATE"]="ACTIVER" },
    de = { ["RESET ALL SETTINGS"]="ALLES ZURÜCKSETZEN", ["SETTINGS RESET"]="EINSTELLUNGEN ZURÜCKGESETZT", ["CONFIRM?"]="BESTÄTIGEN?", ["ERROR"]="FEHLER", ["RESET"]="ZURÜCKSETZEN", ["Keybinds"]="TASTEN", ["Carry Mode"]="TRAGEMODUS", ["Auto Left"]="AUTO LINKS", ["Auto Right"]="AUTO RECHTS", ["Hide GUI"]="MENÜ AUSBLENDEN", ["Normal Speed"]="NORMALE GESCHWINDIGKEIT", ["Carry Speed"]="TRAGEGESCHWINDIGKEIT", ["ACTIVATE"]="AKTIVIEREN" },
}

local function localizeRivalHubInterface(root)
    local dictionary = RIVALHUB_TRANSLATIONS[RIVALHUB_LANGUAGE]
    if not dictionary or not root then return end
    for _, object in ipairs(root:GetDescendants()) do
        if object:IsA("TextLabel") or object:IsA("TextButton") then
            local source = object:GetAttribute("RivalHubSourceText") or object.Text
            object:SetAttribute("RivalHubSourceText", source)
            local translated = dictionary[source]
            if translated then object.Text = translated end
        end
    end
end

local function rivalHubTranslate(source)
    local dictionary = RIVALHUB_TRANSLATIONS[RIVALHUB_LANGUAGE]
    return (dictionary and dictionary[source]) or source
end

local _tick          = tick
local _clamp         = math.clamp
local _floor         = math.floor
local _abs           = math.abs
local _huge          = math.huge
local _sqrt          = math.sqrt
local _V3new         = Vector3.new
local _V3zero        = Vector3.zero
local _CFnew         = CFrame.new
local _CFlookAt      = CFrame.lookAt
local _RayParams_new = RaycastParams.new

local _GetPlayersCached
do
    local cache, cacheTime = nil, 0
    _GetPlayersCached = function()
        local now = _tick()
        if cache and now - cacheTime < 0.03 then return cache end
        cache = Players:GetPlayers()
        cacheTime = now
        return cache
    end
end

local function waitForCharReady(char, timeout)
    timeout = timeout or 5
    local deadline = _tick() + timeout
    while (not char) or (not char.Parent)
          or (not char:FindFirstChild("HumanoidRootPart"))
          or (not char:FindFirstChildOfClass("Humanoid")) do
        if _tick() > deadline then return false end
        task.wait(0.05)
    end
    return true
end

NS = 60
CS = 29
LAGGER_SPEED = 15
LAGGER_CARRY_SPEED = 24.5
BAT_AIMBOT_SPEED = 55
BYPASS_AIMBOT_SPEED = 55
local State = {
    bypassBatEnabled = false,
    bypassBatSpeed = BYPASS_AIMBOT_SPEED,
}
CONFIG_FILE = "kira_" .. tostring(LP.UserId) .. ".json"
BAT_V2_HIT_DIST = 4.5
_isDraggingButton = false

-- ==================== KIRA BACKGROUNDS (VERÃO, NATAL, HALLOWEEN) ====================
local _KIRA_CUSTOM_BG_ASSETS = {}
local _KIRA_BG_B64S = {
    "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",
}

local function decodeKiraBase64(encoded)
    if not encoded or encoded == "" then return "" end
    if crypt and crypt.base64decode then
        local ok, decoded = pcall(crypt.base64decode, encoded)
        if ok and decoded then return decoded end
    end
    if crypt and crypt.base64_decode then
        local ok, decoded = pcall(crypt.base64_decode, encoded)
        if ok and decoded then return decoded end
    end
    if syn and syn.crypt and syn.crypt.base64 and syn.crypt.base64.decode then
        local ok, decoded = pcall(syn.crypt.base64.decode, encoded)
        if ok and decoded then return decoded end
    end
    if base64_decode then
        local ok, decoded = pcall(base64_decode, encoded)
        if ok and decoded then return decoded end
    end
    local chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    local map = {}
    for i = 1, #chars do map[chars:byte(i)] = i - 1 end
    local out, buffer, bits = {}, 0, 0
    for i = 1, #encoded do
        local byte = encoded:byte(i)
        if byte == 61 then break end
        local value = map[byte]
        if value then
            buffer = buffer * 64 + value
            bits = bits + 6
            if bits >= 8 then
                bits = bits - 8
                out[#out + 1] = string.char(math.floor(buffer / (2 ^ bits)))
                buffer = buffer % (2 ^ bits)
            end
        end
    end
    return table.concat(out)
end

local function initKiraBackgrounds()
    local loader = getcustomasset or getsynasset or (syn and syn.crypt and syn.crypt.customasset)
    local write_fn, is_file = writefile, isfile
    if type(loader) ~= "function" or type(write_fn) ~= "function" then return end
    for i, encoded in ipairs(_KIRA_BG_B64S) do
        local ok, asset = pcall(function()
            local filename = "kira_custom_bg_" .. tostring(i) .. ".jpg"
            if not (type(is_file) == "function" and is_file(filename)) then
                write_fn(filename, decodeKiraBase64(encoded))
            end
            return loader(filename)
        end)
        if ok and type(asset) == "string" and asset ~= "" then
            _KIRA_CUSTOM_BG_ASSETS[i] = asset
        end
    end
end
pcall(initKiraBackgrounds)

function getBackgroundAsset(idx)
    idx = idx or 1
    if _KIRA_CUSTOM_BG_ASSETS[idx] then
        return _KIRA_CUSTOM_BG_ASSETS[idx]
    end
    local raw = backgroundImages[idx] or backgroundImages[1]
    if type(raw) == "string" and (raw:sub(1, 13) == "rbxassetid://" or raw:sub(1, 11) == "rbxthumb://") then
        return raw
    end
    return "rbxassetid://" .. tostring(raw)
end

backgroundIndex = 1
backgroundImages = {"73963271683049", "87425565124554", "93776108213038"}
kiraUI = {effects = true, animated = {}, cards = {}, tweens = {}, backgroundRevision = 0}
useVenixCarryEngine = false

backgroundImageTransparency = 0
backgroundMode = "Kira Verão"
backgroundSelectorLabel = nil
miniBackgroundImage = nil
floatingButtonScale = 1
_floatingUIScales = {}

BACKGROUND_OPTIONS = {
    "None",
    "Kira Verão",
    "Kira Natal",
    "Kira Halloween",
}
TPBatState = TPBatState or {
    enabled = false,
    connection = nil,
    hitCooldown = false,
    antiDieConnections = {},
}
TPBatState.enabled = false

function applyBackgroundMode(mode)
    if mode == "None" then
        backgroundMode = "None"
    else
        local targetIdx = nil
        if mode == "Kira Verão" or mode == "Background 1" or mode == "1" then
            targetIdx = 1
        elseif mode == "Kira Natal" or mode == "Background 2" or mode == "2" then
            targetIdx = 2
        elseif mode == "Kira Halloween" or mode == "Background 3" or mode == "3" then
            targetIdx = 3
        else
            local n = tonumber(string.match(tostring(mode), "(%d+)")) or backgroundIndex
            targetIdx = _clamp(_floor(n), 1, #backgroundImages)
        end
        backgroundIndex = targetIdx
        backgroundMode = BACKGROUND_OPTIONS[backgroundIndex + 1] or ("Background " .. backgroundIndex)
    end
    kiraUI.backgroundRevision = kiraUI.backgroundRevision + 1
    local revision = kiraUI.backgroundRevision
    if main then
        main.BackgroundColor3 = Color3.fromRGB(9, 12, 16)
        main.BackgroundTransparency = 0
        local bg = main:FindFirstChild("BackgroundImage")
        local overlay = main:FindFirstChild("BackgroundIncoming")
        if overlay then overlay:Destroy() end
        if kiraUI.backgroundTween then kiraUI.backgroundTween:Cancel() end
        if bg then
            local asset = getBackgroundAsset(backgroundIndex)
            local opacity = backgroundMode == "None" and 1 or (backgroundImageTransparency or 0)
            if backgroundMode == "None" or not kiraUI.effects or bg.Image == asset then
                bg.Image = (backgroundMode == "None" and "" or asset)
                bg.ImageTransparency = opacity
                bg.Visible = (backgroundMode ~= "None")
            else
                local incoming = bg:Clone()
                incoming.Name = "BackgroundIncoming"
                incoming.Image = asset
                incoming.ImageTransparency = 1
                incoming.ZIndex = 1
                incoming.Visible = true
                incoming.Parent = main
                kiraUI.backgroundTween = TS:Create(incoming, TweenInfo.new(0.28, Enum.EasingStyle.Quad), {ImageTransparency = opacity})
                kiraUI.backgroundTween.Completed:Connect(function()
                    if revision == kiraUI.backgroundRevision and bg.Parent then
                        bg.Image = asset
                        bg.ImageTransparency = opacity
                        bg.Visible = true
                    end
                    if incoming.Parent then incoming:Destroy() end
                end)
                kiraUI.backgroundTween:Play()
            end
        end
    end
    if kiraUI.art and kiraUI.art.Parent then
        kiraUI.art.Image = getBackgroundAsset(backgroundIndex)
        kiraUI.art.Visible = backgroundMode ~= "None"
    end
    if miniBackgroundImage then miniBackgroundImage.Visible = false end
    if backgroundSelectorLabel then backgroundSelectorLabel.Text = backgroundMode end
    for i, card in ipairs(kiraUI.cards) do
        if card.Parent then
            local stroke = card:FindFirstChildOfClass("UIStroke")
            if stroke then stroke.Transparency = backgroundMode ~= "None" and i == backgroundIndex and 0 or 0.72 end
            local badge = card:FindFirstChild("Selected")
            if badge then badge.Visible = backgroundMode ~= "None" and i == backgroundIndex end
        end
    end
end

local COLOR_THEMES = {
    ["Red"]     = Color3.fromRGB(245, 60, 60),      -- Vermelho
    ["Orange"]  = Color3.fromRGB(255, 140, 30),     -- Laranja
    ["Yellow"]  = Color3.fromRGB(255, 225, 40),     -- Amarelo
    ["Lime"]    = Color3.fromRGB(120, 230, 50),     -- Verde Claro
    ["Green"]   = Color3.fromRGB(50, 215, 90),      -- Verde
    ["Cyan"]    = Color3.fromRGB(40, 220, 240),     -- Ciano
    ["Blue"]    = Color3.fromRGB(50, 130, 255),     -- Azul
    ["Indigo"]  = Color3.fromRGB(80, 75, 225),      -- Índigo
    ["Violet"]  = Color3.fromRGB(165, 60, 255),     -- Violeta
    ["Purple"]  = Color3.fromRGB(195, 45, 230),     -- Roxo
    ["Pink"]    = Color3.fromRGB(255, 90, 180),     -- Rosa
    ["White"]   = Color3.fromRGB(248, 250, 255),    -- Branco
    ["Rainbow"] = Color3.fromRGB(245, 60, 60),      -- Arco-Íris animado
}

local COLOR_DISPLAY_NAMES = {
    ["Red"]     = "Vermelho",
    ["Orange"]  = "Laranja",
    ["Yellow"]  = "Amarelo",
    ["Lime"]    = "Verde Claro",
    ["Green"]   = "Verde",
    ["Cyan"]    = "Ciano",
    ["Blue"]    = "Azul",
    ["Indigo"]  = "Índigo",
    ["Violet"]  = "Violeta",
    ["Purple"]  = "Roxo",
    ["Pink"]    = "Rosa",
    ["White"]   = "Branco",
    ["Rainbow"] = "Arco-Íris",
}

local COLOR_THEME_KEYS = {
    "Red", "Orange", "Yellow", "Lime", "Green", "Cyan",
    "Blue", "Indigo", "Violet", "Purple", "Pink", "White", "Rainbow"
}

customColorEnabled = true
local _customColorToggles = {}

function setCustomColorEnabled(enabled)
    customColorEnabled = enabled == true
    for _, setter in ipairs(_customColorToggles) do
        pcall(setter, customColorEnabled)
    end
    updateAllUIThemeColors(getThemeColor())
    if saveAllSettings then pcall(saveAllSettings) end
end

RIVAL_TITLE_THEMES = {
    ["Orange"] = { top = Color3.fromRGB(255, 245, 227), bottom = Color3.fromRGB(229, 181, 118) },
    ["Blue"]  = { top = Color3.fromRGB(60, 130, 255),  bottom = Color3.fromRGB(210, 220, 235) },
    ["Red"]   = { top = Color3.fromRGB(255, 70, 70),   bottom = Color3.fromRGB(215, 215, 215) },
    ["Green"] = { top = Color3.fromRGB(50, 220, 110),  bottom = Color3.fromRGB(200, 240, 210) },
    ["Black"] = { top = Color3.fromRGB(35, 35, 40),    bottom = Color3.fromRGB(115, 115, 125) },
    ["Gray"]  = { top = Color3.fromRGB(200, 200, 200), bottom = Color3.fromRGB(120, 120, 130) },
}
currentRivalTitleTheme = "Orange"
scriptLogoRef = nil
rivalTitleSelectorLabel = nil

function applyRivalTitleTheme(themeName)
    local t = RIVAL_TITLE_THEMES[themeName]
    if not t then return end
    currentRivalTitleTheme = themeName
    if scriptLogoRef then
        local grad = scriptLogoRef:FindFirstChildOfClass("UIGradient")
        if grad then
            grad.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, t.top),
                ColorSequenceKeypoint.new(1, t.bottom),
            })
        end
    end
    if rivalTitleSelectorLabel then
        rivalTitleSelectorLabel.Text = themeName
    end
    if saveAllSettings then pcall(saveAllSettings) end
end

currentColorTheme = "Orange"
selectedColor = COLOR_THEMES["Orange"]

function getThemeColor() return selectedColor end

function rivalHubGradient(c)
    return ColorSequence.new({
        ColorSequenceKeypoint.new(0.00, c:Lerp(Color3.new(1,1,1), 0.55)),
        ColorSequenceKeypoint.new(0.45, c),
        ColorSequenceKeypoint.new(1.00, c:Lerp(Color3.new(0,0,0), 0.35)),
    })
end

function rivalHubGradientSoft(c)
    return ColorSequence.new({
        ColorSequenceKeypoint.new(0.00, c:Lerp(Color3.new(1,1,1), 0.75)),
        ColorSequenceKeypoint.new(0.50, c:Lerp(Color3.new(1,1,1), 0.25)),
        ColorSequenceKeypoint.new(1.00, c:Lerp(Color3.new(0,0,0), 0.15)),
    })
end

function rivalHubGradientDark(c)
    return ColorSequence.new({
        ColorSequenceKeypoint.new(0.00, c:Lerp(Color3.fromRGB(0,0,0), 0.35)),
        ColorSequenceKeypoint.new(0.55, c:Lerp(Color3.fromRGB(0,0,0), 0.65)),
        ColorSequenceKeypoint.new(1.00, c:Lerp(Color3.fromRGB(0,0,0), 0.80)),
    })
end

function applyRivalHubGradientToLabel(label, c)
    if not label then return end
    local grad = label:FindFirstChildOfClass("UIGradient")
    if not grad then
        grad = Instance.new("UIGradient", label)
    end
    grad.Rotation = 0
    grad.Color = rivalHubGradient(c)
end

local _lastThemeUpdate = 0
local _lastThemeColor = nil

function applyColorTheme(themeName)
    local color = COLOR_THEMES[themeName]
    if not color and themeName ~= "Rainbow" then return end
    currentColorTheme = themeName
    if themeName == "Rainbow" then
        local now = _tick()
        local hue = (now * 0.16) % 1
        color = Color3.fromHSV(hue, 0.85, 1)
    end
    selectedColor = color or Color3.fromRGB(255, 140, 30)
    updateAllUIThemeColors(selectedColor)
    if _G.__refreshColorButtonSelection then pcall(_G.__refreshColorButtonSelection) end
    if colorSelectorLabel then
        colorSelectorLabel.Text = COLOR_DISPLAY_NAMES[currentColorTheme] or currentColorTheme
        colorSelectorLabel.TextColor3 = (currentColorTheme == "Rainbow") and Color3.fromRGB(255, 215, 0) or selectedColor
    end
    if saveAllSettings then pcall(saveAllSettings) end
end

_uicStroke = nil
_uicAvatarStroke = nil
_uicHandle = nil
_uicLine = nil

function updateAllUIThemeColors(color)
    local now = _tick()
    if color == _lastThemeColor and now - _lastThemeUpdate < 0.1 then return end
    _lastThemeUpdate = now
    _lastThemeColor = color

    if progressFill then
        progressFill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        local grad = progressFill:FindFirstChildOfClass("UIGradient")
        if grad then
            grad.Color = rivalHubGradient(color)
            grad.Rotation = 0
        end
    end
    if pbFrame then
        local border = pbFrame:FindFirstChild("OuterStroke")
        local track = pbFrame:FindFirstChild("StealProgressTrack")
        local divider = pbFrame:FindFirstChild("StatsDivider")
        local fpsNeon = pbFrame:FindFirstChild("FPSNeon", true)
        local pingNeon = pbFrame:FindFirstChild("PingNeon", true)

        if customColorEnabled then
            pbFrame.BackgroundColor3 = color:Lerp(Color3.fromRGB(15, 15, 22), 0.65)
            if border then
                border.Color = color
                border.Transparency = 0.15
            end
            if track then
                track.BackgroundColor3 = color:Lerp(Color3.fromRGB(6, 6, 10), 0.82)
            end
            if progressFill then
                progressFill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
                local grad = progressFill:FindFirstChildOfClass("UIGradient")
                if grad then
                    grad.Color = rivalHubGradient(color)
                    grad.Rotation = 0
                end
            end
            if progressPct then
                progressPct.TextColor3 = Color3.fromRGB(255, 255, 255)
            end
            if progressStatus then
                progressStatus.TextColor3 = color:Lerp(Color3.fromRGB(255, 255, 255), 0.65)
            end
            if divider then divider.BackgroundColor3 = color end
            if stealHud.stats then
                stealHud.stats.TextColor3 = color:Lerp(Color3.fromRGB(255, 255, 255), 0.82)
            end
            if fpsNeon then fpsNeon.TextColor3 = color end
            if pingNeon then pingNeon.TextColor3 = color end
        else
            pbFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
            if border then
                border.Color = Color3.fromRGB(91, 77, 92)
                border.Transparency = 0.45
            end
            if track then
                track.BackgroundColor3 = Color3.fromRGB(34, 30, 39)
            end
            if progressFill then
                progressFill.BackgroundColor3 = Color3.fromRGB(255, 35, 35)
                local grad = progressFill:FindFirstChildOfClass("UIGradient")
                if grad then
                    grad.Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(150, 0, 0)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 35, 35))
                    })
                end
            end
            if progressPct then progressPct.TextColor3 = Color3.fromRGB(255, 255, 255) end
            if progressStatus then progressStatus.TextColor3 = Color3.fromRGB(255, 255, 255) end
            if divider then divider.BackgroundColor3 = Color3.fromRGB(112, 106, 121) end
            if stealHud.stats then stealHud.stats.TextColor3 = Color3.fromRGB(242, 239, 247) end
        end
    end

    if kiraUI.speedStroke and kiraUI.speedStroke.Parent then kiraUI.speedStroke.Color = color end
    if kiraUI.discordLabel and kiraUI.discordLabel.Parent then kiraUI.discordLabel.TextColor3 = color end

    if _uicStroke then _uicStroke.Color = color end
    if _uicAvatarStroke then _uicAvatarStroke.Color = color end
    if _uicHandle then _uicHandle.TextColor3 = Color3.fromRGB(255, 140, 0) end
    if _uicLine then _uicLine.BackgroundColor3 = color end

    local function searchAndUpdateText(parent)
        for _, child in ipairs(parent:GetDescendants()) do
            if child:IsA("TextLabel") then
                if child.Name == "DiscordText" then
                    child.TextColor3 = color
                    local grad = child:FindFirstChildOfClass("UIGradient")
                    if grad then
                        grad.Color = ColorSequence.new({
                            ColorSequenceKeypoint.new(0.00, color),
                            ColorSequenceKeypoint.new(0.30, Color3.fromRGB(200,200,200)),
                            ColorSequenceKeypoint.new(0.50, Color3.fromRGB(255,255,255)),
                            ColorSequenceKeypoint.new(0.70, Color3.fromRGB(200,200,200)),
                            ColorSequenceKeypoint.new(1.00, color),
                        })
                    end
                elseif child.Name == "SpeedLabel" then
                    child.TextColor3 = Color3.fromRGB(245, 45, 45)
                    child.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                    child.TextStrokeTransparency = 0
                    child.Font = Enum.Font.GothamBlack
                    local grad = child:FindFirstChildOfClass("UIGradient")
                    if grad then grad:Destroy() end
                end
            end
            if child:IsA("UIStroke") then
                if child.Color == Color3.fromRGB(180, 180, 190) then child.Color = color end
            end
        end
    end
    if gui then searchAndUpdateText(gui) end
    for _, tab in ipairs(tabButtons or {}) do
        local active = tab:FindFirstChild("Underline") and tab.Underline.Visible
        local underline = tab:FindFirstChild("Underline")
        if underline then underline.BackgroundColor3 = color end
        local stroke = tab:FindFirstChild("TabStroke")
        if stroke then stroke.Color = color; stroke.Transparency = active and 0.42 or 1 end
        tab.BackgroundColor3 = active and Color3.fromRGB(53, 37, 24) or Color3.fromRGB(15, 18, 23)
        tab.BackgroundTransparency = active and 0.05 or 1
        local caption = tab:FindFirstChild("Caption")
        if caption then caption.TextColor3 = active and Color3.fromRGB(249, 242, 229) or Color3.fromRGB(155, 157, 156) end
        if kiraUI.colorSectionIcon then kiraUI.colorSectionIcon(tab:FindFirstChild("SectionIcon"), active and color or Color3.fromRGB(144, 147, 150)) end
    end
    if _G.__RivalHubRefreshESPTheme then pcall(_G.__RivalHubRefreshESPTheme, color) end
    if main then
        local titleFrame = main:FindFirstChild("Frame")
        if titleFrame then
            for _, child in ipairs(titleFrame:GetDescendants()) do
                if child:IsA("UIStroke") and child.Color == Color3.fromRGB(180, 180, 190) then
                    child.Color = color
                end
            end
        end
    end
    if colorSelectorLabel then
        colorSelectorLabel.Text = COLOR_DISPLAY_NAMES[currentColorTheme] or currentColorTheme
        colorSelectorLabel.TextColor3 = (currentColorTheme == "Rainbow") and Color3.fromRGB(255, 215, 0) or color
    end

    if _G._rivalHubStealSlide then
        _G._rivalHubStealSlide.BackgroundColor3 = color
        local sg = _G._rivalHubStealSlide:FindFirstChildOfClass("UIGradient")
        if sg then sg.Color = rivalHubGradient(color) end
        local ss = _G._rivalHubStealSlide:FindFirstChildOfClass("UIStroke")
        if ss then ss.Color = color end
    end

    if miniBtn then
        
        local stroke = miniBtn:FindFirstChildOfClass("UIStroke")
        if stroke then stroke.Color = Color3.fromRGB(255, 140, 0) end
    end

    if MobilePanel then
        for _, btn in ipairs(MobilePanel:GetChildren()) do
            if btn:IsA("TextButton") and btn:FindFirstChild("BtnGrad") then
                paintFloatingBtn(btn, btn:GetAttribute("MobActive") == true)
            end
        end
    end
end

task.spawn(function()
    while true do
        if currentColorTheme == "Rainbow" and customColorEnabled then
            local now = _tick()
            local hue = (now * 0.16) % 1
            local dynamicColor = Color3.fromHSV(hue, 0.85, 1)
            selectedColor = dynamicColor
            updateAllUIThemeColors(dynamicColor)
        end
        task.wait(0.06)
    end
end)

local HOMERO_CFG = {
    body = { 10725826963, 86500008, 86500054, 86500036, 86500064, 86500078 },
    aplicarCuerpo = true,
    items = {
        { id = 103227700418, offset = _CFnew(0,0,0) },
        { id = 84952305140948,   offset = _CFnew(0,0,0) },
        { id = 122465238537030,  offset = _CFnew(0,0,0) },
    },
    shirt = nil,
    pants = "rbxassetid://78591690208112",
    skinColor = Color3.fromRGB(234,184,146),
    headColor = Color3.fromRGB(0,0,0),
}

local TAG = "LocalOutfit_"

local function loadObjects(id)
    local ok, res = pcall(function()
        return game:GetObjects("rbxassetid://" .. tostring(id))
    end)
    if ok and typeof(res) == "table" and #res > 0 then return res end
    ok, res = pcall(function()
        return game:GetService("InsertService"):LoadAsset(id)
    end)
    if ok and res then return { res } end
    return nil
end

local function collectParts(objs)
    local out = {}
    for _, o in ipairs(objs) do
        if o:IsA("BasePart") then out[#out + 1] = o end
        for _, d in ipairs(o:GetDescendants()) do
            if d:IsA("BasePart") then out[#out + 1] = d end
        end
    end
    return out
end

local function findAtt(char, name)
    for _, p in ipairs(char:GetDescendants()) do
        if p:IsA("Attachment") and p.Name == name and p.Parent:IsA("BasePart") then
            return p
        end
    end
end

local function applyBody(char)
    local total = 0
    for _, id in ipairs(HOMERO_CFG.body) do
        local objs = loadObjects(id)
        if objs then
            for _, mp in ipairs(collectParts(objs)) do
                local orig = char:FindFirstChild(mp.Name)
                if orig and orig:IsA("BasePart") then
                    local c = mp:Clone()
                    c.Name = TAG .. "body_" .. mp.Name
                    c.CanCollide = false
                    c.Anchored = false
                    c.Massless = true
                    c.Size = orig.Size
                    c.CFrame = orig.CFrame
                    c.Parent = char
                    local w = Instance.new("WeldConstraint")
                    w.Part0 = orig
                    w.Part1 = c
                    w.Parent = c
                    if orig:GetAttribute("RivalHubOutfitOriginalTransparency") == nil then
                        orig:SetAttribute("RivalHubOutfitOriginalTransparency", orig.Transparency)
                    end
                    orig.Transparency = 1
                    total = total + 1
                end
            end
            for _, o in ipairs(objs) do pcall(function() o:Destroy() end) end
        end
    end
    return total
end

local function attachItem(char, entry)
    local objs = loadObjects(entry.id)
    if not objs then return false end

    local handle
    for _, p in ipairs(collectParts(objs)) do
        if p.Name == "Handle" then handle = p; break end
        if not handle then handle = p end
    end
    if not handle then
        for _, o in ipairs(objs) do pcall(function() o:Destroy() end) end
        return false
    end

    local H = handle:Clone()
    for _, o in ipairs(objs) do pcall(function() o:Destroy() end) end

    H.Name = TAG .. "item_" .. tostring(entry.id)
    H.CanCollide = false
    H.Anchored = false
    H.Massless = true

    local wrap = H:FindFirstChildWhichIsA("WrapLayer")
    local target, c0, c1

    if wrap then
        target = char:FindFirstChild("UpperTorso")
              or char:FindFirstChild("Torso")
              or char:FindFirstChild("HumanoidRootPart")
        c0 = entry.offset or _CFnew()
        c1 = _CFnew()
    else
        local hAtt = H:FindFirstChildOfClass("Attachment")
        local bAtt = hAtt and findAtt(char, hAtt.Name)
        if bAtt then
            target = bAtt.Parent
            c0 = bAtt.CFrame * (entry.offset or _CFnew())
            c1 = hAtt.CFrame
        else
            target = char:FindFirstChild("Head")
            c0 = entry.offset or _CFnew(0, 1.4, 0)
            c1 = _CFnew()
        end
    end

    if not target then H:Destroy(); return false end

    H.CFrame = target.CFrame * c0 * c1:Inverse()
    H.Parent = char

    local w = Instance.new("Weld")
    w.Part0 = target
    w.Part1 = H
    w.C0 = c0
    w.C1 = c1
    w.Parent = H
    return true
end

function applyHomeroOutfit(char)
    if not char then char = LP.Character end
    if not char then return end
    char:WaitForChild("Humanoid", 10)
    char:WaitForChild("Head", 10)
    task.wait(0.4)

    for _, d in ipairs(char:GetChildren()) do
        if d.Name:sub(1, #TAG) == TAG then pcall(function() d:Destroy() end) end
    end

    if HOMERO_CFG.skinColor then
        local bc = char:FindFirstChildWhichIsA("BodyColors") or Instance.new("BodyColors")
        bc.HeadColor3 = HOMERO_CFG.headColor or HOMERO_CFG.skinColor
        bc.TorsoColor3 = HOMERO_CFG.skinColor
        bc.LeftArmColor3, bc.RightArmColor3 = HOMERO_CFG.skinColor, HOMERO_CFG.skinColor
        bc.LeftLegColor3, bc.RightLegColor3 = HOMERO_CFG.skinColor, HOMERO_CFG.skinColor
        bc.Parent = char
    end

    for _, a in ipairs(char:GetChildren()) do
        if a:IsA("Accessory") then
            local h = a:FindFirstChild("Handle")
            if h then h.Transparency = 1 end
        end
    end

    if HOMERO_CFG.shirt then
        local s = char:FindFirstChildWhichIsA("Shirt") or Instance.new("Shirt")
        s.Name = "Shirt"
        s.ShirtTemplate = HOMERO_CFG.shirt
        s.Parent = char
    end
    if HOMERO_CFG.pants then
        local p = char:FindFirstChildWhichIsA("Pants") or Instance.new("Pants")
        p.Name = "Pants"
        p.PantsTemplate = HOMERO_CFG.pants
        p.Parent = char
    end
    if HOMERO_CFG.aplicarCuerpo then applyBody(char) end
    for _, e in ipairs(HOMERO_CFG.items) do attachItem(char, e) end
end

local _originalAppearance = nil

local function captureOriginalAppearance(char)
    if not char or _originalAppearance then return end
    local data = {
        bodyColors = nil,
        shirt = nil,
        pants = nil,
        headMesh = nil,
        headTexture = nil,
    }
    local bc = char:FindFirstChildWhichIsA("BodyColors")
    if bc then
        data.bodyColors = {
            head = bc.HeadColor3,
            torso = bc.TorsoColor3,
            leftArm = bc.LeftArmColor3,
            rightArm = bc.RightArmColor3,
            leftLeg = bc.LeftLegColor3,
            rightLeg = bc.RightLegColor3,
        }
    end
    local sh = char:FindFirstChildWhichIsA("Shirt")
    if sh then data.shirt = sh.ShirtTemplate end
    local pa = char:FindFirstChildWhichIsA("Pants")
    if pa then data.pants = pa.PantsTemplate end
    local head = char:FindFirstChild("Head")
    if head then
        local sm = head:FindFirstChildWhichIsA("SpecialMesh")
        if sm and sm.MeshType == Enum.MeshType.FileMesh then
            data.headMesh = sm.MeshId
            data.headTexture = sm.TextureId
        end
    end
    _originalAppearance = data
end

local function clearPreviousOutfitAssets(char)
    if not char then return end
    for _, child in ipairs(char:GetChildren()) do
        if child.Name:sub(1, #TAG) == TAG
        or child.Name == "AuFfitAccessory"
        or child.Name == "BubbleSkinChanger_Hair"
        or child.Name == "Korblox_RightLeg" then
            pcall(function() child:Destroy() end)
        elseif child:IsA("CharacterMesh") and child.BodyPart == Enum.BodyPart.Head then
            pcall(function() child:Destroy() end)
        end
    end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            local originalTransparency = part:GetAttribute("RivalHubOutfitOriginalTransparency")
            if originalTransparency ~= nil then
                part.Transparency = originalTransparency
                part:SetAttribute("RivalHubOutfitOriginalTransparency", nil)
            end
            local originalLocalTransparency = part:GetAttribute("RivalHubOutfitOriginalLocalTransparency")
            if originalLocalTransparency ~= nil then
                part.LocalTransparencyModifier = originalLocalTransparency
                part:SetAttribute("RivalHubOutfitOriginalLocalTransparency", nil)
            end
        end
    end
    if _originalAppearance and _originalAppearance.bodyColors then
        local bc = char:FindFirstChildWhichIsA("BodyColors") or Instance.new("BodyColors")
        local o = _originalAppearance.bodyColors
        bc.HeadColor3, bc.TorsoColor3 = o.head, o.torso
        bc.LeftArmColor3, bc.RightArmColor3 = o.leftArm, o.rightArm
        bc.LeftLegColor3, bc.RightLegColor3 = o.leftLeg, o.rightLeg
        bc.Parent = char
    end
    local head = char:FindFirstChild("Head")
    if head and head:IsA("MeshPart") and head:GetAttribute("RivalHubOutfitModifiedMeshPart") then
        pcall(function()
            head.MeshId = head:GetAttribute("RivalHubOutfitOriginalMeshId") or ""
            head.TextureID = head:GetAttribute("RivalHubOutfitOriginalTextureId") or ""
        end)
        head:SetAttribute("RivalHubOutfitModifiedMeshPart", nil)
        head:SetAttribute("RivalHubOutfitOriginalMeshId", nil)
        head:SetAttribute("RivalHubOutfitOriginalTextureId", nil)
    elseif head then
        local specialMesh = head:FindFirstChildWhichIsA("SpecialMesh")
        if specialMesh and specialMesh:GetAttribute("RivalHubOutfitCreatedMesh") then
            specialMesh:Destroy()
        elseif specialMesh and specialMesh:GetAttribute("RivalHubOutfitModifiedMesh") then
            specialMesh.MeshId = specialMesh:GetAttribute("RivalHubOutfitOriginalMeshId") or ""
            specialMesh.TextureId = specialMesh:GetAttribute("RivalHubOutfitOriginalTextureId") or ""
            specialMesh:SetAttribute("RivalHubOutfitModifiedMesh", nil)
            specialMesh:SetAttribute("RivalHubOutfitOriginalMeshId", nil)
            specialMesh:SetAttribute("RivalHubOutfitOriginalTextureId", nil)
        end
    end
end

local function applyNoOutfit(char)
    if not char then char = LP.Character end
    if not char then return end
    for _, d in ipairs(char:GetChildren()) do
        if d.Name:sub(1, #TAG) == TAG then pcall(function() d:Destroy() end) end
    end
    local oldAcc = char:FindFirstChild("AuFfitAccessory")
    if oldAcc then pcall(function() oldAcc:Destroy() end) end
    local oldKorblox = char:FindFirstChild("Korblox_RightLeg")
    if oldKorblox then pcall(function() oldKorblox:Destroy() end) end
    for _, d in ipairs(char:GetChildren()) do
        if d:IsA("CharacterMesh") and d.BodyPart == Enum.BodyPart.Head then
            pcall(function() d:Destroy() end)
        end
    end
    local head = char:FindFirstChild("Head")
    if head then
        head.Transparency = 0
        head.CanCollide = true
        head.LocalTransparencyModifier = 0
        local face = head:FindFirstChild("face")
        if face then face.Transparency = 0 end
        local sm = head:FindFirstChildWhichIsA("SpecialMesh")
        if sm then
            if _originalAppearance and _originalAppearance.headMesh then
                sm.MeshId = _originalAppearance.headMesh
                sm.TextureId = _originalAppearance.headTexture or ""
            else
                pcall(function() sm:Destroy() end)
            end
        end
    end
    pcall(function()
        local neck = char:FindFirstChild("Neck")
        if neck then neck.Enabled = true end
    end)
    for _, partName in ipairs({"RightUpperLeg", "RightLowerLeg", "RightFoot"}) do
        local limb = char:FindFirstChild(partName)
        if limb then limb.Transparency = 0 end
    end
    if _originalAppearance and _originalAppearance.bodyColors then
        local bc = char:FindFirstChildWhichIsA("BodyColors") or Instance.new("BodyColors")
        local o = _originalAppearance.bodyColors
        bc.HeadColor3 = o.head
        bc.TorsoColor3 = o.torso
        bc.LeftArmColor3 = o.leftArm
        bc.RightArmColor3 = o.rightArm
        bc.LeftLegColor3 = o.leftLeg
        bc.RightLegColor3 = o.rightLeg
        bc.Parent = char
    end
    if _originalAppearance and _originalAppearance.shirt then
        local s = char:FindFirstChildWhichIsA("Shirt") or Instance.new("Shirt")
        s.ShirtTemplate = _originalAppearance.shirt
        s.Parent = char
    end
    if _originalAppearance and _originalAppearance.pants then
        local p = char:FindFirstChildWhichIsA("Pants") or Instance.new("Pants")
        p.PantsTemplate = _originalAppearance.pants
        p.Parent = char
    end
    for _, a in ipairs(char:GetChildren()) do
        if a:IsA("Accessory") then
            local h = a:FindFirstChild("Handle")
            if h then h.Transparency = 0 end
        end
    end
    for _, item in ipairs(char:GetChildren()) do
        if item:IsA("Accessory") then
            for _, part in ipairs(item:GetDescendants()) do
                if part:IsA("BasePart") then
                    local original = part:GetAttribute("RivalHubOriginalTransparency")
                    if original ~= nil then part.Transparency = original; part:SetAttribute("RivalHubOriginalTransparency", nil) end
                    local originalLocal = part:GetAttribute("RivalHubOriginalLocalTransparency")
                    if originalLocal ~= nil then part.LocalTransparencyModifier = originalLocal; part:SetAttribute("RivalHubOriginalLocalTransparency", nil) end
                end
            end
        elseif item:IsA("Shirt") then
            local original = item:GetAttribute("RivalHubOriginalTemplate")
            if original ~= nil then item.ShirtTemplate = original; item:SetAttribute("RivalHubOriginalTemplate", nil) end
        elseif item:IsA("Pants") then
            local original = item:GetAttribute("RivalHubOriginalTemplate")
            if original ~= nil then item.PantsTemplate = original; item:SetAttribute("RivalHubOriginalTemplate", nil) end
        elseif item:IsA("ShirtGraphic") then
            local original = item:GetAttribute("RivalHubOriginalGraphic")
            if original ~= nil then item.Graphic = original; item:SetAttribute("RivalHubOriginalGraphic", nil) end
        end
    end
end

local OUTFITS = {
    { label = "OFF", customApply = applyNoOutfit },
    -- Presets de skin copiados do KiraVS.
    { label = "KIRA V1", bubbleShirt = 74707712629633, bubblePants = 12405320750, bubbleHair = 140188532534398 },
    { label = "KIRA V2", bubbleShirt = 101796619834594, bubblePants = 18975891159, bubbleHair = 115520061093937 },
    { label = "KIRA V3", bubbleShirt = 18552805597, bubblePants = 5414143509, bubbleHair = 84008082880128 },
}
local currentOutfitIndex = 1
local outfitSelectorLabel = nil

VX7A = {
    index = 0, mod = nil, loading = false, busy = false, pending = nil, label = nil,
    labels = {"OFF"},
    src = "return { apply = function() return false end, remove = function() end }",
}

function VX7A.load()
    if VX7A.mod then return VX7A.mod end
    if VX7A.loading or type(loadstring) ~= "function" then return nil end
    VX7A.loading = true
    local ok, fn = pcall(loadstring, VX7A.src)
    if ok and type(fn) == "function" then
        local ok2, res = pcall(fn)
        if ok2 and type(res) == "table" then VX7A.mod = res end
    end
    VX7A.loading = false
    return VX7A.mod
end

function VX7A.setLabel()
    if VX7A.label and VX7A.label.Parent then
        VX7A.label.Text = VX7A.labels[VX7A.index + 1] or "OFF"
    end
end

function VX7A.stop()
    VX7A.pending = nil
    if VX7A.index ~= 0 then
        VX7A.index = 0
        local m = VX7A.mod
        if m and m.remove then pcall(m.remove) end
    end
    VX7A.setLabel()
end

function VX7A.apply(index)
    index = math.clamp(math.floor(tonumber(index) or 0), 0, #VX7A.labels - 1)
    if index == 0 then VX7A.stop() return end
    local m = VX7A.load()
    if not m or type(m.apply) ~= "function" then
        warn("[Rival Hub] No se pudo cargar el modulo de avatares VX7 (loadstring no disponible).")
        VX7A.setLabel()
        return
    end
    if VX7A.busy then VX7A.pending = index; VX7A.index = index; VX7A.setLabel() return end
    if currentOutfitIndex ~= 1 then
        currentOutfitIndex = 1
        pcall(function() applyOutfitByIndex(1) end)
        if outfitSelectorLabel then outfitSelectorLabel.Text = OUTFITS[1].label end
    end
    VX7A.index = index
    VX7A.setLabel()
    VX7A.busy = true
    task.spawn(function()
        pcall(m.apply, index)
        VX7A.busy = false
        local nxt = VX7A.pending
        VX7A.pending = nil
        if nxt and nxt ~= 0 and VX7A.index == nxt then VX7A.apply(nxt) end
    end)
end

pcall(function()
    LP.CharacterAdded:Connect(function()
        if _G.RivalHubSession ~= _mySession or VX7A.index == 0 then return end
        task.delay(1.5, function()
            if _G.RivalHubSession == _mySession and VX7A.index ~= 0 then VX7A.apply(VX7A.index) end
        end)
    end)
end)

local function loadObjectsStd(id)
    local ok, res = pcall(function() return game:GetObjects("rbxassetid://" .. tostring(id)) end)
    if ok and typeof(res) == "table" and #res > 0 then return res end
    ok, res = pcall(function() return game:GetService("InsertService"):LoadAsset(id) end)
    if ok and res then return {res} end
    return nil
end
local function resolveBubbleTemplate(id, className, propertyName)
    local fallback = "rbxassetid://" .. tostring(id)
    local ok, objects = pcall(function() return game:GetObjects(fallback) end)
    if ok and objects and objects[1] then
        local item = objects[1]
        local obj = item:IsA(className) and item or item:FindFirstChildWhichIsA(className, true)
        if obj and obj[propertyName] and obj[propertyName] ~= "" then fallback = obj[propertyName] end
        for _, v in ipairs(objects) do pcall(function() v:Destroy() end) end
    end
    return fallback
end
local function addBubbleHair(char, assetId)
    local head = char and char:FindFirstChild("Head")
    if not head or not head:IsA("BasePart") then
        warn("[Rival Hub] KiraVS hair: Head nao encontrado.")
        return false
    end

    local objects = loadObjectsStd(assetId)
    if not objects or not objects[1] then
        warn("[Rival Hub] Nao foi possivel carregar o cabelo: " .. tostring(assetId))
        return false
    end

    local accessory
    for _, obj in ipairs(objects) do
        if obj:IsA("Accessory") then
            accessory = obj
            break
        end
        accessory = obj:FindFirstChildWhichIsA("Accessory", true)
        if accessory then break end
    end
    if not accessory then
        for _, obj in ipairs(objects) do pcall(function() obj:Destroy() end) end
        warn("[Rival Hub] O asset de cabelo nao contem um Accessory: " .. tostring(assetId))
        return false
    end

    local hair = accessory:Clone()
    for _, obj in ipairs(objects) do pcall(function() obj:Destroy() end) end
    hair.Name = "BubbleSkinChanger_Hair"

    local handle = hair:FindFirstChild("Handle")
    if not handle or not handle:IsA("BasePart") then
        pcall(function() hair:Destroy() end)
        warn("[Rival Hub] O cabelo nao possui Handle: " .. tostring(assetId))
        return false
    end

    for _, part in ipairs(hair:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Anchored = false
            part.CanCollide = false
            part.Massless = true
            part.LocalTransparencyModifier = 0
            part.Transparency = 0
        end
    end
    handle.Transparency = 0
    handle.LocalTransparencyModifier = 0

    -- Posiciona o cabelo manualmente usando os Attachments, como no KiraVS original.
    local handleAttachment, headAttachment
    for _, att in ipairs(handle:GetChildren()) do
        if att:IsA("Attachment") then
            local target = head:FindFirstChild(att.Name)
            if target and target:IsA("Attachment") then
                handleAttachment, headAttachment = att, target
                break
            end
        end
    end

    local c0, c1
    if handleAttachment and headAttachment then
        c0 = headAttachment.CFrame
        c1 = handleAttachment.CFrame
    else
        c0 = CFrame.new(0, head.Size.Y * 0.5, 0)
        c1 = CFrame.new()
    end

    -- Remove welds criados automaticamente pelo Roblox antes de alinhar o acessorio.
    for _, obj in ipairs(handle:GetChildren()) do
        if obj:IsA("Weld") or obj:IsA("WeldConstraint") or obj:IsA("Motor6D") then
            obj:Destroy()
        end
    end
    hair.Parent = char
    handle.CFrame = head.CFrame * c0 * c1:Inverse()

    local weld = Instance.new("Weld")
    weld.Name = "AccessoryWeld"
    weld.Part0 = handle
    weld.Part1 = head
    weld.C0 = c1
    weld.C1 = c0
    weld.Parent = handle
    return true
end

local function applyHeadlessKorblox(char)
    if not char then return end
    pcall(function() LP.CharacterAvatarType = Enum.AvatarType.R6 end)
    local head = char:FindFirstChild("Head")
    if head then
        if head:GetAttribute("RivalHubOutfitOriginalTransparency") == nil then
            head:SetAttribute("RivalHubOutfitOriginalTransparency", head.Transparency)
            head:SetAttribute("RivalHubOutfitOriginalLocalTransparency", head.LocalTransparencyModifier)
        end
        head.Transparency = 1
        head.CanCollide = false
        head.LocalTransparencyModifier = 1
        local face = head:FindFirstChild("face")
        if face then face.Transparency = 1 end
    end
    pcall(function()
        local neck = char:FindFirstChild("Neck")
        if neck then neck.Enabled = false end
    end)
    for _, v in pairs(char:GetChildren()) do
        if v:IsA("Accessory") then
            local w = v:FindFirstChildWhichIsA("Weld") or v:FindFirstChildWhichIsA("WeldConstraint") or v:FindFirstChildWhichIsA("Motor6D")
            if w then
                local p0, p1 = w.Part0, w.Part1
                if (p0 and p0.Name == "Head") or (p1 and p1.Name == "Head") then
                    for _, part in ipairs(v:GetDescendants()) do
                        if part:IsA("BasePart") then
                            if part:GetAttribute("RivalHubOutfitOriginalTransparency") == nil then
                                part:SetAttribute("RivalHubOutfitOriginalTransparency", part.Transparency)
                                part:SetAttribute("RivalHubOutfitOriginalLocalTransparency", part.LocalTransparencyModifier)
                            end
                            part.Transparency = 1
                            part.LocalTransparencyModifier = 1
                        end
                    end
                end
            end
        end
    end
    local rightLegConfig = { id = "rbxassetid://139607718", targetBodyPart = "RightUpperLeg", partsToHide = {"RightUpperLeg", "RightLowerLeg", "RightFoot"}, scale = _V3new(1, 1, 1), offset = _CFnew(0, 0, 0) }
    local targetPart = char:FindFirstChild(rightLegConfig.targetBodyPart)
    if targetPart then
        local oldAsset = char:FindFirstChild("Korblox_RightLeg")
        if oldAsset then oldAsset:Destroy() end
        for _, partName in ipairs(rightLegConfig.partsToHide) do
            local limb = char:FindFirstChild(partName)
            if limb and limb:IsA("BasePart") then
                if limb:GetAttribute("RivalHubOutfitOriginalTransparency") == nil then
                    limb:SetAttribute("RivalHubOutfitOriginalTransparency", limb.Transparency)
                end
                limb.Transparency = 1
            end
        end
        local success, objects = pcall(function() return game:GetObjects(rightLegConfig.id) end)
        if success and objects and #objects > 0 then
            local assetModel = objects[1]
            assetModel.Name = "Korblox_RightLeg"
            local mainMesh = assetModel:IsA("BasePart") and assetModel or assetModel:FindFirstChildWhichIsA("BasePart", true)
            if mainMesh then
                mainMesh.Size = mainMesh.Size * rightLegConfig.scale
                mainMesh.CanCollide = false
                mainMesh.CFrame = targetPart.CFrame * rightLegConfig.offset
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = targetPart
                weld.Part1 = mainMesh
                weld.Parent = mainMesh
                assetModel.Parent = char
            end
        end
    end
end

function applyOutfitByIndex(index)
    local cfg = OUTFITS[index]
    if not cfg then return end
    local char = LP.Character
    if not char then return end
    clearPreviousOutfitAssets(char)
    for _, item in ipairs(char:GetChildren()) do
        if item:IsA("Accessory") and item.Name:sub(1, #TAG) ~= TAG then
            for _, part in ipairs(item:GetDescendants()) do
                if part:IsA("BasePart") then
                    if part:GetAttribute("RivalHubOriginalTransparency") == nil then
                        part:SetAttribute("RivalHubOriginalTransparency", part.Transparency)
                        part:SetAttribute("RivalHubOriginalLocalTransparency", part.LocalTransparencyModifier)
                    end
                    part.Transparency = 1
                    part.LocalTransparencyModifier = 1
                end
            end
        elseif item:IsA("Shirt") then
            if item:GetAttribute("RivalHubOriginalTemplate") == nil then item:SetAttribute("RivalHubOriginalTemplate", item.ShirtTemplate) end
            item.ShirtTemplate = ""
        elseif item:IsA("Pants") then
            if item:GetAttribute("RivalHubOriginalTemplate") == nil then item:SetAttribute("RivalHubOriginalTemplate", item.PantsTemplate) end
            item.PantsTemplate = ""
        elseif item:IsA("ShirtGraphic") then
            if item:GetAttribute("RivalHubOriginalGraphic") == nil then item:SetAttribute("RivalHubOriginalGraphic", item.Graphic) end
            item.Graphic = ""
        end
    end

    if cfg.customApply then
        cfg.customApply(char)
        if outfitSelectorLabel then outfitSelectorLabel.Text = cfg.label end
        return
    end
    if cfg.bubbleShirt then
        local s = char:FindFirstChildWhichIsA("Shirt") or Instance.new("Shirt")
        s.Name = "BubbleSkinChanger_Shirt"
        s.ShirtTemplate = resolveBubbleTemplate(cfg.bubbleShirt, "Shirt", "ShirtTemplate")
        s.Parent = char
        local p = char:FindFirstChildWhichIsA("Pants") or Instance.new("Pants")
        p.Name = "BubbleSkinChanger_Pants"
        p.PantsTemplate = resolveBubbleTemplate(cfg.bubblePants, "Pants", "PantsTemplate")
        p.Parent = char
        addBubbleHair(char, cfg.bubbleHair)
        if outfitSelectorLabel then outfitSelectorLabel.Text = cfg.label end
        return
    end
    char:WaitForChild("Head", 5)
    local head = char:FindFirstChild("Head")
    if not head then return end
    for _, d in ipairs(char:GetChildren()) do
        if d:IsA("CharacterMesh") and d.BodyPart == Enum.BodyPart.Head then
            pcall(function() d:Destroy() end)
        end
    end
    local done = false
    if head:IsA("MeshPart") then
        done = pcall(function()
            if not head:GetAttribute("RivalHubOutfitModifiedMeshPart") then
                head:SetAttribute("RivalHubOutfitOriginalMeshId", head.MeshId)
                head:SetAttribute("RivalHubOutfitOriginalTextureId", head.TextureID)
                head:SetAttribute("RivalHubOutfitModifiedMeshPart", true)
            end
            head.MeshId = cfg.headMesh
            head.TextureID = cfg.headTexture or ""
        end)
    end
    if not done then
        local sm = head:FindFirstChildWhichIsA("SpecialMesh")
        if not sm then
            sm = Instance.new("SpecialMesh")
            sm:SetAttribute("RivalHubOutfitCreatedMesh", true)
        elseif not sm:GetAttribute("RivalHubOutfitModifiedMesh") then
            sm:SetAttribute("RivalHubOutfitOriginalMeshId", sm.MeshId)
            sm:SetAttribute("RivalHubOutfitOriginalTextureId", sm.TextureId)
            sm:SetAttribute("RivalHubOutfitModifiedMesh", true)
        end
        sm.Parent = head
        sm.MeshType = Enum.MeshType.FileMesh
        sm.MeshId = cfg.headMesh
        sm.TextureId = cfg.headTexture or ""
    end
    if cfg.shirt then
        local s = char:FindFirstChildWhichIsA("Shirt") or Instance.new("Shirt")
        s.Name = "Shirt"
        s.ShirtTemplate = cfg.shirt
        s.Parent = char
    end
    if cfg.pants then
        local p = char:FindFirstChildWhichIsA("Pants") or Instance.new("Pants")
        p.Name = "Pants"
        p.PantsTemplate = cfg.pants
        p.Parent = char
    end
    local old = char:FindFirstChild("AuFfitAccessory")
    if old then old:Destroy() end
    if cfg.accessory and head then
        local objs = loadObjectsStd(cfg.accessory)
        if objs then
            local handle
            for _, o in ipairs(objs) do
                if o:IsA("BasePart") then handle = o; break end
                local f = o:FindFirstChildWhichIsA("BasePart", true)
                if f then handle = f; break end
            end
            if handle then
                local h = handle:Clone()
                h.Name = "AuFfitAccessory"
                h.CanCollide = false
                h.Anchored = false
                h.Massless = true
                h.Parent = char
                local weld = Instance.new("Weld")
                weld.Part0 = head
                weld.Part1 = h
                weld.C0 = _CFnew(cfg.offset)
                weld.Parent = h
            end
            for _, o in ipairs(objs) do pcall(function() o:Destroy() end) end
        end
    end
    if cfg.headlessKorblox then
        applyHeadlessKorblox(char)
    else
        if char then
            local head2 = char:FindFirstChild("Head")
            if head2 then
                head2.Transparency = 0
                head2.CanCollide = true
                head2.LocalTransparencyModifier = 0
                local face2 = head2:FindFirstChild("face")
                if face2 then face2.Transparency = 0 end
            end
            pcall(function()
                local neck = char:FindFirstChild("Neck")
                if neck then neck.Enabled = true end
            end)
            for _, partName in ipairs({"RightUpperLeg", "RightLowerLeg", "RightFoot"}) do
                local limb = char:FindFirstChild(partName)
                if limb then limb.Transparency = 0 end
            end
            local oldKorblox = char:FindFirstChild("Korblox_RightLeg")
            if oldKorblox then oldKorblox:Destroy() end
        end
    end
    if outfitSelectorLabel then outfitSelectorLabel.Text = cfg.label end
end

speedMode = false
antiRagdollMode = "off"
antiDieEnabled = false
antiBatEnabled = false
antiFlingEnabled = false
jumpEnabled = false
laggerToggled = false
laggerCarryToggled = false
batCounterEnabled = false
unwalkEnabled = false
autoLeftEnabled = false
autoRightEnabled = false
autoBatEnabled = false
dropMode = 1
antiLagEnabled = false
removeAccessoriesEnabled = false
stretchEnabled = false
stretchFOV = 120
uiLocked = false
mobileButtonsLocked = false
uiScaleValue = 90
espEnabled = false
espLineEnabled = false

vividGraphicsEnabled = false
_vividEffects = {}
setVividVisual = nil

hideButtonsEnabled = false
setHideButtonsVisual = nil

bodyLockEnabled = false
bodyLockRange = 20
bodyLockRangeBox = nil
_bodyLockConn = nil
_blSuppressCount = 0
_blWasEnabled = false
_blRestoreTimer = nil
_blSmoothRestore = false

savedProgressBarPos = nil
savedButtonPositions = {}
savedMobilePanelPos = nil

neonWeatherEnabled = false
skyTheme = "Off"
skySelectorLabel = nil
_originalLighting = nil
setNeonWeatherVisual = nil

currentAnimPack = "Off"
originalTryardAnims = nil
tryardHeartbeatConn = nil
animSelectorLabel = nil

lastMoveDir = _V3zero

local CoreGui = game:GetService("CoreGui")

local InfiniteJump = {
    enabled = false,
    mode = "hold",
    jumpPower = 55,
    minVelocity = 35,
    fallClamp = -120,
    jumpConn = nil,
    heartbeatConn = nil,
    presimConn = nil,
}

local function applyInfJumpBoost(root)
    if not root then return end
    local velocity = root.AssemblyLinearVelocity
    if velocity.Y < InfiniteJump.minVelocity then
        root.AssemblyLinearVelocity = _V3new(velocity.X, InfiniteJump.jumpPower, velocity.Z)
    end
    if velocity.Y < InfiniteJump.fallClamp then
        root.AssemblyLinearVelocity = _V3new(velocity.X, InfiniteJump.fallClamp, velocity.Z)
    end
end

local function onJumpRequest()
    if not InfiniteJump.enabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then
        pcall(function()
            root.AssemblyLinearVelocity = _V3new(
                root.AssemblyLinearVelocity.X,
                InfiniteJump.jumpPower,
                root.AssemblyLinearVelocity.Z
            )
        end)
    end
end

local function onPreSimulation()
    if not InfiniteJump.enabled then return end
    if InfiniteJump.mode ~= "hold" then return end
    local char = LP.Character
    if not char then return end
    local hum  = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end

    local held = UIS:IsKeyDown(Enum.KeyCode.Space)
        or UIS:IsKeyDown(Enum.KeyCode.ButtonA)
        or (hum.Jump == true)

    if held then
        applyInfJumpBoost(root)
    else
        local v = root.AssemblyLinearVelocity
        if v.Y < InfiniteJump.fallClamp then
            root.AssemblyLinearVelocity = _V3new(v.X, InfiniteJump.fallClamp, v.Z)
        end
    end
end

local function connectEvents()
    if InfiniteJump.jumpConn then InfiniteJump.jumpConn:Disconnect() end
    if InfiniteJump.heartbeatConn then InfiniteJump.heartbeatConn:Disconnect() end
    if InfiniteJump.presimConn then InfiniteJump.presimConn:Disconnect() end
    InfiniteJump.jumpConn = UIS.JumpRequest:Connect(onJumpRequest)
    InfiniteJump.presimConn = RunService.PreSimulation:Connect(onPreSimulation)
end

function InfiniteJump.start()
    if InfiniteJump.enabled then return end
    InfiniteJump.enabled = true
    connectEvents()
end

function InfiniteJump.stop()
    InfiniteJump.enabled = false
    if InfiniteJump.jumpConn then InfiniteJump.jumpConn:Disconnect(); InfiniteJump.jumpConn = nil end
    if InfiniteJump.heartbeatConn then InfiniteJump.heartbeatConn:Disconnect(); InfiniteJump.heartbeatConn = nil end
    if InfiniteJump.presimConn then InfiniteJump.presimConn:Disconnect(); InfiniteJump.presimConn = nil end
end

function InfiniteJump.setJumpPower(power)
    power = tonumber(power) or 55
    InfiniteJump.jumpPower = _clamp(power, 10, 200)
end

function InfiniteJump.setMode(mode)
    InfiniteJump.mode = mode == "manual" and "manual" or "hold"
end

function InfiniteJump.isRunning() return InfiniteJump.enabled == true end

InfiniteJump.stop()

function getActiveMoveSpeed()
    if useVenixCarryEngine and kiraVenixCarry and kiraVenixCarry:isRunning() then return kiraVenixCarry:getActiveSpeed() end
    if laggerCarryToggled then return LAGGER_CARRY_SPEED
    elseif laggerToggled then return LAGGER_SPEED
    elseif speedMode then return CS
    else return NS end
end

function getSpeedModeName()
    if useVenixCarryEngine and kiraVenixCarry and kiraVenixCarry.softStealEnabled and kiraVenixCarry.softStealLatched then return "AUTO CARRY" end
    if laggerToggled or laggerCarryToggled then return "LAGGER"
    elseif speedMode then return "CARRY"
    else return "NORMAL" end
end

local _s2VelState = {root = nil, v = _V3zero}
local _velChecked = setmetatable({}, {__mode = "k"})
local _hookedVelParts = {}
local _velocityHook = _G.__kiraVelocityHook or {hooked = false, roots = setmetatable({}, {__mode = "k"})}
_G.__kiraVelocityHook = _velocityHook
local function _hookVelHRP(root)
    if not root then return end
    _s2VelState.root = root
    _velChecked[root] = true
    _velocityHook.roots[root] = true
    if _velocityHook.hooked then return end
    if type(getrawmetatable) ~= "function" or type(setreadonly) ~= "function"
        or type(newcclosure) ~= "function" or type(checkcaller) ~= "function" then return end
    pcall(function()
        local mt = getrawmetatable(root)
        if not mt then return end
        local oldIndex = rawget(mt, "__index")
        if type(oldIndex) ~= "function" and type(oldIndex) ~= "table" then return end
        local function original(obj, key)
            if type(oldIndex) == "function" then return oldIndex(obj, key) end
            return oldIndex[key]
        end
        setreadonly(mt, false)
        mt.__index = newcclosure(function(obj, key)
            if not checkcaller() and _velocityHook.roots[obj]
                and (key == "AssemblyLinearVelocity" or key == "Velocity") then
                local real = original(obj, key)
                if real and real.Magnitude > 20 then return real.Unit * 20 end
                return real
            end
            return original(obj, key)
        end)
        setreadonly(mt, true)
        _velocityHook.hooked = true
    end)
end
local function _setupVelChecked(character)
    _velocityHook.roots = setmetatable({}, {__mode = "k"})
    _velChecked = setmetatable({}, {__mode = "k"})
    if not character then _s2VelState.root = nil; return nil end
    local root = character:WaitForChild("HumanoidRootPart", 5)
    if root then _s2VelState.root = root; _velChecked[root] = true; _velocityHook.roots[root] = true end
    return root
end
if LP.Character then _hookVelHRP(_setupVelChecked(LP.Character)) end
local function _isRagdollState(hum)
    if not hum then return true end
    local state = hum:GetState()
    return hum.PlatformStand or state == Enum.HumanoidStateType.Physics
        or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown
end
local function _applyVelocitySpeed(direction, speed, root)
    if not root or not root.Parent then return end
    if autoBatEnabled or TPBatState.enabled or (_G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2()) then return end
    if type(speed) ~= "number" or speed ~= speed or speed <= 0 or speed == math.huge then return end
    local y = root.AssemblyLinearVelocity.Y
    if direction and direction.Magnitude > 0.05 then
        pcall(function() root:SetNetworkOwner(LP) end)
        local unit = direction.Unit
        root.AssemblyLinearVelocity = _V3new(unit.X * speed, y, unit.Z * speed)
    else
        root.AssemblyLinearVelocity = _V3new(0, y, 0)
    end
    _G.__RivalHubLiveSpeed = {t = os.clock(), v = speed}
end
function getAutoPathSpeed()
    if laggerCarryToggled or laggerToggled then return LAGGER_SPEED end
    return NS
end

        kiraVenixCarry = {
            normalSpeed = NS,
            carrySpeed = CS,
            laggerSpeed = LAGGER_SPEED,
            laggerCarrySpeed = LAGGER_CARRY_SPEED,
            speedToggled = false,
            laggerMode = 0,
            softStealEnabled = false,
            softStealRadius = 10,
            softStealSpeed = 30,
            softStealLatched = false,
            _isCarrying = false,
            _lastCarryCheck = 0,
            _lvBoost = nil,
            _lvAtt = nil,
            _blockedTime = 0,
            _maxForce = 2200,
            _freeForce = 500,
            _heartbeatConn = nil,
            _softStealScanner = nil,
            _softStealAnimals = {},
            _softStealScanning = false,
            _state = nil,
            _dropInProgress = false,
            _batAimbotToggled = false,
            _rayParams = nil,
            _rayFilter = nil,
            _rayFilterTime = 0,
            isCarrying = function(self)
                local now = _tick()
                if now - (self._lastCarryCheck or 0) < 0.1 then
                    return self._isCarrying
                end
                self._lastCarryCheck = now
                local character = LP.Character
                if not character then
                    self._isCarrying = false
                    return false
                end
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                local walkSpeed = humanoid and humanoid.WalkSpeed or 16
                local isCarrying = walkSpeed < 25 and walkSpeed > 0
                local ok, result = pcall(function()
                    return LP:GetAttribute("Stealing")
                end)
                ok = ok and result == true
                local isEnabled = false
                if ok then
                    isEnabled = true
                end
                local ok2, result2 = pcall(function()
                    return character:GetAttribute("Stealing")
                end)
                if ok2 and result2 == true then
                    isEnabled = true
                end
                if not isEnabled then
                    for _, name in ipairs({
                        "Carrying",
                        "IsCarrying",
                        "Grabbed",
                        "Holding",
                        "StealHold",
                        "HasGrab",
                    }) do
                        local instance = character:FindFirstChild(name)
                        if instance then
                            if
                                instance:IsA("BoolValue") and instance.Value
                                or instance:IsA("ObjectValue") and instance.Value
                                or instance:IsA("StringValue") and instance.Value ~= ""
                            then
                                isEnabled = true
                                break
                            end
                        end
                    end
                end
                self._isCarrying = isCarrying or isEnabled
                return self._isCarrying
            end,
            getActiveSpeed = function(self)
                if self._state and (self._state.autoLeftEnabled or self._state.autoRightEnabled) then
                    return self.normalSpeed
                end
                if self.softStealEnabled then
                    local _, distance = self:getNearestSoftStealAnimal(self.softStealRadius)
                    if distance and distance <= self.softStealRadius then
                        self.softStealLatched = true
                        return self.softStealSpeed
                    end
                    if self.softStealLatched and self:isCarrying() then
                        return self.softStealSpeed
                    end
                    self.softStealLatched = false
                end
                if self.laggerMode == 1 then
                    return self.laggerSpeed
                end
                if self.laggerMode == 2 then
                    return self.laggerCarrySpeed
                end
                if self.speedToggled then
                    return self.carrySpeed
                end
                return self.normalSpeed
            end,
            getStatus = function(self)
                if not (self._state and (self._state.autoLeftEnabled or self._state.autoRightEnabled)) then
                    if self.softStealEnabled then
                        local _, distance = self:getNearestSoftStealAnimal(self.softStealRadius)
                        if
                            distance and distance <= self.softStealRadius
                            or self.softStealLatched and self:isCarrying()
                        then
                            return "AUTO CARRY", self.softStealSpeed
                        end
                    end
                    if self.laggerMode == 1 then
                        return "LAGGER", self.laggerSpeed
                    end
                    if self.laggerMode == 2 then
                        return "LAGGER CARRY", self.laggerCarrySpeed
                    end
                    if self.speedToggled then
                        return "CARRY", self.carrySpeed
                    end
                    return "NORMAL", self.normalSpeed
                end
                return "NORMAL", self.normalSpeed
            end,
        }
        do
            local function destroyLV()
                if kiraVenixCarry._lvBoost and kiraVenixCarry._lvBoost.Parent then
                    pcall(function()
                        kiraVenixCarry._lvBoost:Destroy()
                    end)
                end
                if kiraVenixCarry._lvAtt and kiraVenixCarry._lvAtt.Parent then
                    pcall(function()
                        kiraVenixCarry._lvAtt:Destroy()
                    end)
                end
                kiraVenixCarry._lvBoost = nil
                kiraVenixCarry._lvAtt = nil
            end
            local function setupLV(parent)
                if kiraVenixCarry._lvBoost and kiraVenixCarry._lvBoost.Parent == parent then
                    return
                end
                destroyLV()
                local attachment = Instance.new("Attachment")
                attachment.Parent = parent
                local linearVelocity = Instance.new("LinearVelocity")
                linearVelocity.Name = "kiraCarryBoost"
                attachment.Name = "kiraCarryAttachment"
                linearVelocity.Attachment0 = attachment
                linearVelocity.VelocityConstraintMode = Enum.VelocityConstraintMode.Plane
                linearVelocity.PrimaryTangentAxis = _V3new(1, 0, 0)
                linearVelocity.SecondaryTangentAxis = _V3new(0, 0, 1)
                linearVelocity.MaxForce = kiraVenixCarry._maxForce
                linearVelocity.PlaneVelocity = Vector2.zero
                linearVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
                linearVelocity.Parent = parent
                kiraVenixCarry._lvAtt = attachment
                kiraVenixCarry._lvBoost = linearVelocity
                pcall(function()
                    parent:SetNetworkOwner(LP)
                end)
            end
            kiraVenixCarry.scanSoftStealAnimals = function(self)
                self._softStealAnimals = {}
                local plots = Workspace:FindFirstChild("Plots")
                if not plots then
                    return
                end
                for _, child in ipairs(plots:GetChildren()) do
                    if child:IsA("Model") then
                        local animalPodiums = child:FindFirstChild("AnimalPodiums")
                        if animalPodiums then
                            for _, child2 in ipairs(animalPodiums:GetChildren()) do
                                if child2:IsA("Model") then
                                    local base = child2:FindFirstChild("Base")
                                    base = base and base:FindFirstChild("Spawn")
                                    if base then
                                        table.insert(self._softStealAnimals, {
                                            plot = child.Name,
                                            slot = child2.Name,
                                            worldPosition = base.Position,
                                            uid = child.Name .. "_" .. child2.Name,
                                        })
                                    end
                                end
                            end
                        end
                    end
                end
            end
            kiraVenixCarry.startSoftStealScanner = function(callback)
                if callback._softStealScanner then
                    return
                end
                callback._softStealScanning = true
                callback:scanSoftStealAnimals()
                local accumulator = 0
                callback._softStealScanner = RunService.Heartbeat:Connect(function(dt)
                    accumulator = accumulator + dt
                    if accumulator < 0.2 then return end
                    accumulator = 0
                    if not callback._softStealScanning then
                        return
                    end
                    callback:scanSoftStealAnimals()
                end)
            end
            kiraVenixCarry.stopSoftStealScanner = function(self)
                self._softStealScanning = false
                if self._softStealScanner then
                    self._softStealScanner:Disconnect()
                    self._softStealScanner = nil
                end
            end
            kiraVenixCarry.getNearestSoftStealAnimal = function(self, radius)
                local character = LP.Character
                if character then
                    character = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso")
                end
                if not character then
                    return nil, _huge
                end
                local softStealAnimals = self._softStealAnimals
                local position = character.Position
                local bestDistance = _huge
                local animals = nil
                for i = 1, #softStealAnimals do
                    local rootPart = softStealAnimals[i]
                    if rootPart.worldPosition then
                        local dx = position.X - rootPart.worldPosition.X
                        local dy = position.Y - rootPart.worldPosition.Y
                        local dz = position.Z - rootPart.worldPosition.Z
                        local distance = _sqrt(dx * dx + dy * dy + dz * dz)
                        if distance < bestDistance then
                            bestDistance = distance
                            animals = rootPart
                        end
                    end
                end
                if radius and bestDistance > radius then
                    return nil, bestDistance
                end
                return animals, bestDistance
            end
            kiraVenixCarry.updateMovement = function(self, deltaTime)
                local character = LP.Character
                if not character then
                    return
                end
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if not humanoidRootPart or not humanoid or humanoid.Health <= 0 then
                    return
                end
                local speed = self:getActiveSpeed()
                local moveDirection = humanoid.MoveDirection
                local moving = moveDirection.Magnitude > 0.1
                local unit = nil
                if moving then
                    if not self._rayParams then
                        self._rayParams = _RayParams_new()
                        self._rayParams.FilterType = Enum.RaycastFilterType.Exclude
                        self._rayFilter = {}
                        self._rayFilterTime = 0
                    end
                    local now = _tick()
                    if now - self._rayFilterTime > 1 then
                        self._rayFilterTime = now
                        local rayFilter = self._rayFilter
                        while #rayFilter > 0 do
                            rayFilter[#rayFilter] = nil
                        end
                        rayFilter[1] = character
                        local plist = _GetPlayersCached()
                        for i = 1, #plist do
                            local p = plist[i]
                            if p.Character then
                                rayFilter[#rayFilter + 1] = p.Character
                            end
                        end
                        self._rayParams.FilterDescendantsInstances = rayFilter
                    else
                        local rayFilter = self._rayFilter
                        local found = false
                        for i = 1, #rayFilter do
                            if rayFilter[i] == character then
                                found = true
                                break
                            end
                        end
                        if not found then
                            rayFilter[1] = character
                            self._rayParams.FilterDescendantsInstances = rayFilter
                        end
                    end
                    local unit2 = _V3new(moveDirection.X, 0, moveDirection.Z).Unit
                    local rayParams = self._rayParams
                    local hit =
                        Workspace:Raycast(humanoidRootPart.Position + _V3new(0, 1, 0), unit2 * 2.5, rayParams)
                    local canCollide = hit and hit.Instance and hit.Instance.CanCollide
                    unit = nil
                    if canCollide then
                        local nf = _V3new(hit.Normal.X, 0, hit.Normal.Z)
                        unit = nil
                        if nf.Magnitude > 0.7 then
                            unit = nf.Unit
                        end
                    end
                end
                local hVelocity =
                    _V3new(humanoidRootPart.AssemblyLinearVelocity.X, 0, humanoidRootPart.AssemblyLinearVelocity.Z)
                local blocked = unit ~= nil or moving and hVelocity.Magnitude < 2
                if blocked then
                    for _, child in ipairs(character:GetChildren()) do
                        if child:IsA("Model") then
                            for _, descendant in ipairs(child:GetDescendants()) do
                                if descendant:IsA("BasePart") and descendant.CanCollide then
                                    descendant.CanCollide = false
                                end
                            end
                        end
                    end
                    for _, name in ipairs({
                        "Carrying",
                        "IsCarrying",
                        "Grabbed",
                        "Holding",
                        "StealHold",
                        "HasGrab",
                    }) do
                        local instance = character:FindFirstChild(name)
                        if
                            instance
                            and instance:IsA("ObjectValue")
                            and instance.Value
                            and instance.Value:IsA("Model")
                        then
                            for _, descendant in ipairs(instance.Value:GetDescendants()) do
                                if descendant:IsA("BasePart") and descendant.CanCollide then
                                    descendant.CanCollide = false
                                end
                            end
                        end
                    end
                end
                local state = humanoid:GetState()
                if
                    not (
                        state == Enum.HumanoidStateType.Physics
                        or state == Enum.HumanoidStateType.Ragdoll
                        or state == Enum.HumanoidStateType.FallingDown
                        or self._dropInProgress
                        or self._batAimbotToggled
                    )
                then
                    if not kiraVenixCarry._lvBoost or kiraVenixCarry._lvBoost.Parent ~= humanoidRootPart then
                        setupLV(humanoidRootPart)
                    end
                    local lvBoost = kiraVenixCarry._lvBoost
                    if lvBoost then
                        if not lvBoost.Enabled then
                            lvBoost.Enabled = true
                        end
                        if 0.1 < moveDirection.Magnitude then
                            local unit2 = _V3new(moveDirection.X, 0, moveDirection.Z).Unit
                            if unit then
                                local wanted = unit2 * speed
                                local along = wanted - unit * wanted:Dot(unit)
                                if along.Magnitude < 0.5 then
                                    lvBoost.PlaneVelocity = Vector2.zero
                                else
                                    lvBoost.PlaneVelocity = Vector2.new(along.X, along.Z)
                                end
                            else
                                lvBoost.PlaneVelocity = Vector2.new(unit2.X * speed, unit2.Z * speed)
                            end
                        else
                            lvBoost.PlaneVelocity = Vector2.zero
                        end
                        if blocked then
                            self._blockedTime = self._blockedTime + (deltaTime or 0.016)
                        else
                            self._blockedTime = 0
                        end
                        if self._blockedTime > 0.35 then
                            if lvBoost.MaxForce ~= self._freeForce then
                                lvBoost.MaxForce = self._freeForce
                            end
                        elseif lvBoost.MaxForce ~= self._maxForce then
                            lvBoost.MaxForce = self._maxForce
                        end
                    end
                elseif kiraVenixCarry._lvBoost then
                    kiraVenixCarry._lvBoost.PlaneVelocity = Vector2.zero
                    if kiraVenixCarry._lvBoost.Enabled then
                        kiraVenixCarry._lvBoost.Enabled = false
                    end
                end
            end
            kiraVenixCarry.start = function(deltaTime236)
                if deltaTime236._heartbeatConn then
                    return
                end
                deltaTime236._heartbeatConn = RunService.Heartbeat:Connect(function(deltaTime)
                    if _mySession ~= _G.RivalHubSession then deltaTime236:stop(); return end
                    deltaTime236.normalSpeed = NS
                    deltaTime236.carrySpeed = CS
                    deltaTime236.laggerSpeed = LAGGER_SPEED
                    deltaTime236.laggerCarrySpeed = LAGGER_CARRY_SPEED
                    deltaTime236.speedToggled = speedMode
                    deltaTime236.laggerMode = laggerCarryToggled and 2 or (laggerToggled and 1 or 0)
                    deltaTime236._state = {autoLeftEnabled = autoLeftEnabled, autoRightEnabled = autoRightEnabled}
                    deltaTime236._dropInProgress = dropActive or _G.IsDropping or autoLeftEnabled or autoRightEnabled
                    deltaTime236._batAimbotToggled = autoBatEnabled or TPBatState.enabled
                        or (_G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2())
                    deltaTime236:updateMovement(deltaTime)
                end)
                if deltaTime236.softStealEnabled then
                    deltaTime236:startSoftStealScanner()
                end
                print("[CarrySystem] Activado")
            end
            kiraVenixCarry.stop = function(self)
                if self._heartbeatConn then
                    self._heartbeatConn:Disconnect()
                    self._heartbeatConn = nil
                end
                self:stopSoftStealScanner()
                destroyLV()
                self.softStealLatched = false
                print("[CarrySystem] Desactivado")
            end
        end
    kiraVenixCarry.setNormalSpeed = function(self, value)
        self.normalSpeed = _clamp(value, 1, 500)
    end
    kiraVenixCarry.setCarrySpeed = function(self, value)
        self.carrySpeed = _clamp(value, 1, 500)
    end
    kiraVenixCarry.setLaggerSpeed = function(self, value)
        self.laggerSpeed = _clamp(value, 0.1, 500)
    end
    kiraVenixCarry.setLaggerCarrySpeed = function(self, value)
        self.laggerCarrySpeed = _clamp(value, 0.1, 500)
    end
    kiraVenixCarry.setSoftStealSpeed = function(self, value)
        self.softStealSpeed = _clamp(value, 1, 500)
    end
    kiraVenixCarry.setSoftStealRadius = function(self, value)
        self.softStealRadius = _clamp(value, 1, 200)
    end
    kiraVenixCarry.toggleCarryMode = function(self)
        self.speedToggled = not self.speedToggled
    end
    kiraVenixCarry.setLaggerMode = function(self, mode)
        if mode == 0 then
            self.laggerMode = 0
        elseif mode == 1 then
            self.laggerMode = 1
        elseif mode == 2 then
            self.laggerMode = 2
        end
    end
    kiraVenixCarry.toggleLaggerMode = function(self)
        if self.laggerMode == 0 then
            self.laggerMode = 1
        elseif self.laggerMode == 1 then
            self.laggerMode = 2
        else
            self.laggerMode = 0
        end
    end
    kiraVenixCarry.setSoftStealEnabled = function(self, softStealEnabled)
        self.softStealEnabled = softStealEnabled
        if softStealEnabled then
            self:startSoftStealScanner()
        else
            self:stopSoftStealScanner()
            self.softStealLatched = false
        end
    end
    kiraVenixCarry.toggleSoftSteal = function(self)
        self:setSoftStealEnabled(not self.softStealEnabled)
    end
    kiraVenixCarry.isRunning = function(self)
        return self._heartbeatConn ~= nil
    end
    kiraVenixCarry.getCurrentSpeed = function(self)
        return self:getActiveSpeed()
    end

ANIM_PACKS = {
    ["Zombie"] = { idle1="rbxassetid://616158929", idle2="rbxassetid://616160636", walk="rbxassetid://616168032", run="rbxassetid://616163682", jump="rbxassetid://616161997", fall="rbxassetid://616157476", climb="rbxassetid://616156119", swim="rbxassetid://616165109", swimidle="rbxassetid://616166655" },
    ["Ninja"] = { idle1="rbxassetid://656117400", idle2="rbxassetid://656117400", walk="rbxassetid://656121766", run="rbxassetid://656118852", jump="rbxassetid://656117878", fall="rbxassetid://656115606", climb="rbxassetid://656114359", swim="rbxassetid://656117400", swimidle="rbxassetid://656117400" },
    ["Knight"] = { idle1="rbxassetid://657595757", idle2="rbxassetid://657595757", walk="rbxassetid://657552124", run="rbxassetid://657564596", jump="rbxassetid://658409194", fall="rbxassetid://657600338", climb="rbxassetid://658360781", swim="rbxassetid://657595757", swimidle="rbxassetid://657595757" },
    ["Elder"] = { idle1="rbxassetid://845397899", idle2="rbxassetid://845397899", walk="rbxassetid://845403856", run="rbxassetid://845386501", jump="rbxassetid://845398858", fall="rbxassetid://845397673", climb="rbxassetid://845392038", swim="rbxassetid://845397899", swimidle="rbxassetid://845397899" },
    ["Levitate"] = { idle1="rbxassetid://616006778", idle2="rbxassetid://616006778", walk="rbxassetid://616013216", run="rbxassetid://616013216", jump="rbxassetid://616008936", fall="rbxassetid://616005863", climb="rbxassetid://616003713", swim="rbxassetid://616006778", swimidle="rbxassetid://616006778" },
    ["Astronaut"] = { idle1="rbxassetid://891621366", idle2="rbxassetid://891621366", walk="rbxassetid://891636393", run="rbxassetid://891636393", jump="rbxassetid://891627522", fall="rbxassetid://891617961", climb="rbxassetid://891609353", swim="rbxassetid://891621366", swimidle="rbxassetid://891621366" },
    ["Pirate"] = { idle1="rbxassetid://750781874", idle2="rbxassetid://750781874", walk="rbxassetid://750785693", run="rbxassetid://750783738", jump="rbxassetid://750782230", fall="rbxassetid://750780242", climb="rbxassetid://750779899", swim="rbxassetid://750781874", swimidle="rbxassetid://750781874" },
    ["Toy"] = { idle1="rbxassetid://782841498", idle2="rbxassetid://782841498", walk="rbxassetid://782843345", run="rbxassetid://782842708", jump="rbxassetid://782847020", fall="rbxassetid://782846423", climb="rbxassetid://782843869", swim="rbxassetid://782841498", swimidle="rbxassetid://782841498" },
    ["Vampire"] = { idle1="rbxassetid://1083445855", idle2="rbxassetid://1083445855", walk="rbxassetid://1083473930", run="rbxassetid://1083462077", jump="rbxassetid://1083455352", fall="rbxassetid://1083443587", climb="rbxassetid://1083439238", swim="rbxassetid://1083445855", swimidle="rbxassetid://1083445855" },
    ["Werewolf"] = { idle1="rbxassetid://1083195517", idle2="rbxassetid://1083195517", walk="rbxassetid://1083178339", run="rbxassetid://1083216690", jump="rbxassetid://1083218792", fall="rbxassetid://1083189019", climb="rbxassetid://1083182000", swim="rbxassetid://1083195517", swimidle="rbxassetid://1083195517" },
    ["Rthro"] = { idle1="rbxassetid://2510196951", idle2="rbxassetid://2510196951", walk="rbxassetid://2510202577", run="rbxassetid://2510198475", jump="rbxassetid://2510197830", fall="rbxassetid://2510195892", climb="rbxassetid://2510192778", swim="rbxassetid://2510196951", swimidle="rbxassetid://2510196951" },
    ["Stylish"] = { idle1="rbxassetid://616136790", idle2="rbxassetid://616136790", walk="rbxassetid://616146177", run="rbxassetid://616140816", jump="rbxassetid://616139451", fall="rbxassetid://616134815", climb="rbxassetid://616133594", swim="rbxassetid://616136790", swimidle="rbxassetid://616136790" },
}

ANIM_PACK_ORDER = {{"Off", "Off"}, {"Zombie", "Zombie"}, {"Ninja", "Ninja"}, {"Knight", "Knight"}, {"Elder", "Elder"}, {"Levitate", "Levitate"}, {"Astronaut", "Astronaut"}, {"Pirate", "Pirate"}, {"Toy", "Toy"}, {"Vampire", "Vampire"}, {"Werewolf", "Werewolf"}, {"Rthro", "Rthro"}, {"Stylish", "Stylish"}}

local function isPackAnim(id)
    for _, pack in pairs(ANIM_PACKS) do
        for _, v in pairs(pack) do
            if v == id then return true end
        end
    end
    return false
end

local function saveOriginalAnims(char)
    local animate = char:FindFirstChild("Animate")
    if not animate then return end
    local function g(obj) return obj and obj.AnimationId or nil end
    local ids = {
        idle1 = g(animate.idle and animate.idle.Animation1),
        idle2 = g(animate.idle and animate.idle.Animation2),
        walk  = g(animate.walk and animate.walk.WalkAnim),
        run   = g(animate.run  and animate.run.RunAnim),
        jump  = g(animate.jump and animate.jump.JumpAnim),
        fall  = g(animate.fall and animate.fall.FallAnim),
        climb = g(animate.climb and animate.climb.ClimbAnim),
        swim  = g(animate.swim and animate.swim.Swim),
        swimidle = g(animate.swimidle and animate.swimidle.SwimIdle),
    }
    if not isPackAnim(ids.walk) then originalTryardAnims = ids end
end

local function applyAnimPack(packName)
    currentAnimPack = packName
    if animSelectorLabel then animSelectorLabel.Text = packName end
    if packName == "Off" then
        if originalTryardAnims and LP.Character then
            local animate = LP.Character:FindFirstChild("Animate")
            if animate then
                local function s(obj,id) if obj then obj.AnimationId = id end end
                s(animate.idle and animate.idle.Animation1, originalTryardAnims.idle1)
                s(animate.idle and animate.idle.Animation2, originalTryardAnims.idle2)
                s(animate.walk and animate.walk.WalkAnim, originalTryardAnims.walk)
                s(animate.run  and animate.run.RunAnim,   originalTryardAnims.run)
                s(animate.jump and animate.jump.JumpAnim, originalTryardAnims.jump)
                s(animate.fall and animate.fall.FallAnim, originalTryardAnims.fall)
                s(animate.climb and animate.climb.ClimbAnim, originalTryardAnims.climb)
                s(animate.swim and animate.swim.Swim, originalTryardAnims.swim)
                s(animate.swimidle and animate.swimidle.SwimIdle, originalTryardAnims.swimidle)
            end
        end
        if tryardHeartbeatConn then tryardHeartbeatConn:Disconnect(); tryardHeartbeatConn = nil end
        return
    end
    local pack = ANIM_PACKS[packName]
    if not pack then return end
    if tryardHeartbeatConn then tryardHeartbeatConn:Disconnect() end
    tryardHeartbeatConn = RunService.Heartbeat:Connect(function()
        local c = LP.Character
        if not c then return end
        local animate = c:FindFirstChild("Animate")
        if not animate then return end
        local function s(obj,id) if obj then obj.AnimationId = id end end
        s(animate.idle and animate.idle.Animation1, pack.idle1)
        s(animate.idle and animate.idle.Animation2, pack.idle2)
        s(animate.walk and animate.walk.WalkAnim, pack.walk)
        s(animate.run  and animate.run.RunAnim,   pack.run)
        s(animate.jump and animate.jump.JumpAnim, pack.jump)
        s(animate.fall and animate.fall.FallAnim, pack.fall)
        s(animate.climb and animate.climb.ClimbAnim, pack.climb)
        s(animate.swim and animate.swim.Swim, pack.swim)
        s(animate.swimidle and animate.swimidle.SwimIdle, pack.swimidle)
    end)
end

local function startAnimPack(packName)
    local char = LP.Character
    if char then
        saveOriginalAnims(char)
        applyAnimPack(packName)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            for _, track in ipairs(hum:GetPlayingAnimationTracks()) do track:Stop(0) end
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    else
        applyAnimPack(packName)
    end
    currentAnimPack = packName
end

local function stopAnimPack()
    currentAnimPack = "Off"
    if animSelectorLabel then animSelectorLabel.Text = "Off" end
    applyAnimPack("Off")
end

DEFAULT_KB = {
    DropBrainrot = {kb = Enum.KeyCode.R, gp = nil},
    AutoLeft     = {kb = Enum.KeyCode.Z, gp = nil},
    AutoRight    = {kb = Enum.KeyCode.C, gp = nil},
    AutoBat      = {kb = Enum.KeyCode.X, gp = nil},
    TPFloor      = {kb = Enum.KeyCode.E, gp = nil},
    GuiHide      = {kb = Enum.KeyCode.T, gp = nil},
    CarryToggle  = {kb = Enum.KeyCode.Q, gp = nil},
    LaggerMode   = {kb = Enum.KeyCode.F, gp = nil},
    BatV2        = {kb = Enum.KeyCode.Four, gp = nil},
    InstaReset   = {kb = Enum.KeyCode.G, gp = nil},
}

KB = {
    DropBrainrot = {kb = DEFAULT_KB.DropBrainrot.kb, gp = DEFAULT_KB.DropBrainrot.gp},
    AutoLeft     = {kb = DEFAULT_KB.AutoLeft.kb, gp = DEFAULT_KB.AutoLeft.gp},
    AutoRight    = {kb = DEFAULT_KB.AutoRight.kb, gp = DEFAULT_KB.AutoRight.gp},
    AutoBat      = {kb = DEFAULT_KB.AutoBat.kb, gp = DEFAULT_KB.AutoBat.gp},
    TPFloor      = {kb = DEFAULT_KB.TPFloor.kb, gp = DEFAULT_KB.TPFloor.gp},
    GuiHide      = {kb = DEFAULT_KB.GuiHide.kb, gp = DEFAULT_KB.GuiHide.gp},
    CarryToggle  = {kb = DEFAULT_KB.CarryToggle.kb, gp = DEFAULT_KB.CarryToggle.gp},
    LaggerMode   = {kb = DEFAULT_KB.LaggerMode.kb, gp = DEFAULT_KB.LaggerMode.gp},
    BatV2        = {kb = DEFAULT_KB.BatV2.kb, gp = DEFAULT_KB.BatV2.gp},
    InstaReset   = {kb = DEFAULT_KB.InstaReset.kb, gp = DEFAULT_KB.InstaReset.gp},
}

_isResetting = false
_lastSavedJSON = nil
_isLoading = false

CONFIG = {
    AUTO_STEAL_ENABLED = false,
    STEAL_RANGE = 61,
    HOLD_MIN = 0.05,
    HOLD_MAX = 0.15,
    ENTRY_DELAY = 0.1,
    COOLDOWN = 0.2,
    PRIME_RANGE = 60,
}

local plots = workspace:FindFirstChild("Plots")
local stealConnection = nil

local Steal = {
    AutoStealEnabled = false,
    StealRadius = CONFIG.STEAL_RANGE,
    StealDuration = 1.3,
    StealDelay = 0.25,
    Data = {}
}

local isStealing = false
local autoStealMode = "75"
local autoStealVariant = 1
local autoStealGeneration = 0
autoStealStartedAt = 0
autoStealLastActivity = _tick()
autoStealSupervisorConn = nil
local AUTO_STEAL_VARIANT_NAMES = {"75", "80", "85", "90", "V3", "Semi"}
local function setAutoStealMode(mode)
    autoStealVariant = 1
    if mode == "V2" then mode = "80" elseif mode == "V1" then mode = "75" end
    for i, name in ipairs(AUTO_STEAL_VARIANT_NAMES) do
        if name == tostring(mode) then autoStealVariant = i; break end
    end
    autoStealMode = AUTO_STEAL_VARIANT_NAMES[autoStealVariant]
end
local function getAutoStealDuration()
    local factors = {0.75, 0.80, 0.85, 0.90, 1, 1}
    return Steal.StealDuration * factors[autoStealVariant]
end
local autoGrabSetDelayRadius = 9
local autoGrabStopTime = 0.96
local autoGrabStopEnabled = true

local _plotsCache = nil
local _plotsCacheTime = 0
local function getPlotsRoot()
    local now = _tick()
    if _plotsCache and now - _plotsCacheTime < 2 and _plotsCache.Parent then
        return _plotsCache
    end
    _plotsCache = workspace:FindFirstChild("Plots")
    _plotsCacheTime = now
    return _plotsCache
end

local function isMyPlotByName(plotName)
    local plotsRoot = getPlotsRoot()
    if not plotsRoot then return false end
    local plot = plotsRoot:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then
            return yb.Enabled == true
        end
    end
    return false
end

local function findNearestPrompt()
    local char = LP.Character
    if not char then return nil, nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil, nil end
    local plotsRoot = getPlotsRoot()
    if not plotsRoot then return nil, nil end
    local nearestPrompt, nearestDist, nearestName = nil, _huge, nil
    local rpos = root.Position
    for _, plot in ipairs(plotsRoot:GetChildren()) do
        if not isMyPlotByName(plot.Name) then
            local pods = plot:FindFirstChild("AnimalPodiums")
            if pods then
                for _, pod in ipairs(pods:GetChildren()) do
                    pcall(function()
                        local base = pod:FindFirstChild("Base")
                        local spawn = base and base:FindFirstChild("Spawn")
                        if spawn then
                            local sp = spawn.Position
                            local dx = sp.X - rpos.X
                            local dy = sp.Y - rpos.Y
                            local dz = sp.Z - rpos.Z
                            local dist = _sqrt(dx*dx + dy*dy + dz*dz)
                            if dist < nearestDist and dist <= Steal.StealRadius then
                                local att = spawn:FindFirstChild("PromptAttachment")
                                if att then
                                    for _, child in ipairs(att:GetChildren()) do
                                        if child:IsA("ProximityPrompt") and child.ActionText and child.ActionText:find("Steal") then
                                            nearestPrompt = child
                                            nearestDist = dist
                                            nearestName = pod.Name
                                            break
                                        end
                                    end
                                end
                            end
                        end
                    end)
                end
            end
        end
    end
    return nearestPrompt, nearestName
end

local function executeSteal(prompt, podName)
    if isStealing or not prompt or not prompt.Parent then return end
    for old in pairs(Steal.Data) do if not old.Parent then Steal.Data[old] = nil end end
    local data = Steal.Data[prompt]
    if not data then
        data = {hold = {}, trigger = {}, ready = true}
        Steal.Data[prompt] = data
        pcall(function()
            if type(getconnections) ~= "function" then return end
            for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                if c.Function then table.insert(data.hold, c.Function) end
            end
            for _, c in ipairs(getconnections(prompt.Triggered)) do
                if c.Function then table.insert(data.trigger, c.Function) end
            end
        end)
    end
    if not data.ready then return end
    local generation = autoStealGeneration
    data.ready = false
    isStealing = true
    autoStealStartedAt = _tick()
    autoStealLastActivity = autoStealStartedAt
    local function active()
        return generation == autoStealGeneration and isStealing and Steal.AutoStealEnabled
            and _mySession == _G.RivalHubSession
    end
    local function distance()
        if not prompt.Parent or not prompt.Parent.Parent then return nil end
        local char = LP.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not root or not hum or hum.Health <= 0 then return nil end
        local parent = prompt.Parent
        local part = parent:IsA("BasePart") and parent or parent.Parent
        if not part or not part:IsA("BasePart") then return nil end
        return (root.Position - part.Position).Magnitude
    end
    local function valid()
        if not active() then return false end
        local d = distance()
        return d ~= nil and d <= Steal.StealRadius
    end
    task.spawn(function()
        local success, err = pcall(function()
            updateStealProgress(0, "STEALING")
            for _, fn in ipairs(data.hold) do task.spawn(fn) end
            local started = _tick()
            local duration = Steal.StealDuration
            local stopTime = getAutoStealDuration()
            while valid() and _tick() - started < stopTime do
                updateStealProgress((_tick() - started) / duration, "STEALING")
                task.wait()
            end
            if not valid() then return end
            local stopProgress = _clamp(stopTime / duration, 0, 1)
            updateStealProgress(stopProgress, "APPROACH")
            local deadline = _tick() + math.max(2.99 - stopTime - math.max(duration - stopTime, 0), 0.05)
            while valid() do
                local d = distance()
                if d and d <= 9 then break end
                if _tick() >= deadline then return end
                task.wait()
            end
            if not valid() then return end
            local fillStart = _tick()
            local fillDuration = math.max(duration - stopTime, 0.05)
            while valid() do
                local fp = _clamp((_tick() - fillStart) / fillDuration, 0, 1)
                updateStealProgress(stopProgress + fp * (1 - stopProgress), "STEALING")
                if fp >= 1 then break end
                task.wait()
            end
            if not valid() then return end
            for _, fn in ipairs(data.trigger) do task.spawn(fn) end
            local remote = ReplicatedStorage:FindFirstChild("StealAnimal")
            if remote and remote:IsA("RemoteEvent") and podName then pcall(function() remote:FireServer(podName) end) end
            if #data.trigger == 0 then
                if type(fireproximityprompt) == "function" then pcall(fireproximityprompt, prompt)
                else pcall(function() prompt:InputHoldBegin(); prompt:InputHoldEnd() end) end
            end
            updateStealProgress(1, "COMPLETE")
            task.wait(0.12)
        end)
        data.ready = true
        if generation == autoStealGeneration then
            isStealing = false
            autoStealStartedAt = 0
            autoStealLastActivity = _tick()
            updateStealProgress(0, "READY")
        end
        if not success then warn("[kira Auto Steal] " .. tostring(err)) end
    end)
end
function startAutoSteal()
    Steal.StealRadius = autoStealVariant == 6 and 10 or CONFIG.STEAL_RANGE
    Steal.AutoStealEnabled = true
    CONFIG.AUTO_STEAL_ENABLED = true
    autoStealLastActivity = _tick()

    if stealConnection and not stealConnection.Connected then
        stealConnection = nil
    end
    if stealConnection then return true end

    stealConnection = RunService.Heartbeat:Connect(function()
        if _mySession ~= _G.RivalHubSession then
            stopAutoSteal()
            return
        end
        if not CONFIG.AUTO_STEAL_ENABLED then return end

        if not Steal.AutoStealEnabled then
            Steal.AutoStealEnabled = true
        end

        if isStealing then
            if autoStealStartedAt > 0 and (_tick() - autoStealStartedAt) > 6 then
                autoStealGeneration = autoStealGeneration + 1
                isStealing = false
                autoStealStartedAt = 0
                for _, data in pairs(Steal.Data) do
                    data.ready = true
                end
                updateStealProgress(0, "READY")
            end
            return
        end

        local p, name = findNearestPrompt()
        if p then
            autoStealLastActivity = _tick()
            executeSteal(p, name)
        end
    end)
    return true
end

function stopAutoSteal()
    autoStealGeneration = autoStealGeneration + 1
    if stealConnection then
        pcall(function() stealConnection:Disconnect() end)
        stealConnection = nil
    end
    isStealing = false
    autoStealStartedAt = 0
    autoStealLastActivity = _tick()
    Steal.AutoStealEnabled = false
    CONFIG.AUTO_STEAL_ENABLED = false
    for _, data in pairs(Steal.Data) do data.ready = true end
    updateStealProgress(0, "READY")
end

if autoStealSupervisorConn then
    pcall(function() autoStealSupervisorConn:Disconnect() end)
end
autoStealSupervisorElapsed = 0
do
    autoStealSupervisorConn = RunService.Heartbeat:Connect(function(dt)
        if _mySession ~= _G.RivalHubSession then
            if autoStealSupervisorConn then
                pcall(function() autoStealSupervisorConn:Disconnect() end)
                autoStealSupervisorConn = nil
            end
            return
        end

        autoStealSupervisorElapsed = autoStealSupervisorElapsed + dt
        if autoStealSupervisorElapsed < 0.5 then return end
        autoStealSupervisorElapsed = 0

        if CONFIG.AUTO_STEAL_ENABLED then
            local alive = stealConnection and stealConnection.Connected
            if not alive or not Steal.AutoStealEnabled then
                pcall(startAutoSteal)
            elseif isStealing and autoStealStartedAt > 0 and (_tick() - autoStealStartedAt) > 6 then
                autoStealGeneration = autoStealGeneration + 1
                isStealing = false
                autoStealStartedAt = 0
                for _, data in pairs(Steal.Data) do
                    data.ready = true
                end
                updateStealProgress(0, "READY")
            end
        end
    end)
end

dropActive = false
lastDropTime = 0
lastMoveDir = _V3new(0,0,0)
origFOV = nil
fovEnabled = false
fovValue = 70
customFovConn = nil

_anyKeyListening = false
_aimbotConn = nil
_prevAutoRotate = nil
tpBatConn = nil
tpBatPrevAutoRotate = nil
tpBatHitCD = false
TP_BAT_SWING_CD = 0.08

enemySpeedConn = nil
movementLoop = nil
steppedConn = nil
alConn = nil
arConn = nil
infJumpConn = nil
stretchConn = nil
stretchFovConn = nil
antiLagDescConn = nil
dropConnections = {}
enemySpeedLabels = {}
Conns = {autoSteal = nil, batCounter = nil, anchor = {}, progress = nil, autoLeft = nil, autoRight = nil}
keyButtonRefs = {}
progressFill = nil
progressPct = nil
progressStatus = nil
progressTween = nil
pbFrame = nil
stealHud = {value = 0, state = "READY", statsText = "-- FPS  ·  PING: -- MS"}
speedLabel = nil
modeValLbl = nil
normalBox, carryBox, laggerBox, lagger2Box, radInput, batSpeedBox, uiScaleBox = nil, nil, nil, nil, nil, nil, nil
modeSelectBtn, dropModeBtnRef = nil, nil
autoBatSetVisual, autoLeftSetVisual, autoRightSetVisual, setBatCounterVisual = nil, nil, nil, nil
setAntiRagVisual, setUnwalkVisual, setAntiLagVisual, setLockUIVisual, setInstaGrab = nil, nil, nil, nil, nil
infJumpSetVisual = nil
infJumpModeBtn = nil
autoStealModeBtn = nil
setAntiDieVisual = nil
setAntiBatVisual = nil
setAntiFlingVisual = nil
setESPVIsual = nil
setESPLineVisual = nil
mobSetAutoBat, mobSetAutoLeft, mobSetAutoRight, mobSetDropBR, mobSetTpDown, mobSetCarry, mobSetLagger1, mobSetLagger2, mobSetBatV2 = nil, nil, nil, nil, nil, nil, nil, nil, nil
miniBtn, main, gui = nil, nil, nil
MobilePanel = nil
showGui = nil
hideGui = nil
mainUIScale = nil
animSelectorLabel = nil
pbScale = nil
tabButtons = nil
colorSelectorLabel = nil

GAMEPAD_KEYS = {
    [Enum.KeyCode.ButtonA] = true, [Enum.KeyCode.ButtonB] = true,
    [Enum.KeyCode.ButtonX] = true, [Enum.KeyCode.ButtonY] = true,
    [Enum.KeyCode.ButtonL1] = true, [Enum.KeyCode.ButtonR1] = true,
    [Enum.KeyCode.ButtonL2] = true, [Enum.KeyCode.ButtonR2] = true,
    [Enum.KeyCode.ButtonL3] = true, [Enum.KeyCode.ButtonR3] = true,
    [Enum.KeyCode.ButtonStart] = true, [Enum.KeyCode.ButtonSelect] = true,
    [Enum.KeyCode.DPadUp] = true, [Enum.KeyCode.DPadDown] = true,
    [Enum.KeyCode.DPadLeft] = true, [Enum.KeyCode.DPadRight] = true,
}

MOVE_KEYS = {
    [Enum.KeyCode.W] = true, [Enum.KeyCode.A] = true,
    [Enum.KeyCode.S] = true, [Enum.KeyCode.D] = true,
    [Enum.KeyCode.Up] = true, [Enum.KeyCode.Left] = true,
    [Enum.KeyCode.Down] = true, [Enum.KeyCode.Right] = true,
}

BAT_COUNTER_SLAP_LIST = {
    "Bat", "Slap", "Iron Slap", "Gold Slap", "Diamond Slap",
    "Emerald Slap", "Ruby Slap", "Dark Matter Slap", "Flame Slap",
    "Nuclear Slap", "Galaxy Slap", "Glitched Slap"
}

AP = {
    L1 = _V3new(-476.48, -6.28, 92.73),
    L2 = _V3new(-483.12, -4.95, 94.80),
    L_FACE = _V3new(-482.25, -4.96, 92.09),
    R1 = _V3new(-476.16, -6.52, 25.62),
    R2 = _V3new(-483.06, -5.03, 25.48),
    R_FACE = _V3new(-482.06, -6.93, 35.47),
}

function isGamepadInput(inp)
    return inp and inp.UserInputType and inp.UserInputType.Name:match("^Gamepad") ~= nil
end

function isBindableInput(inp)
    if not inp or inp.KeyCode == Enum.KeyCode.Unknown then return false end
    if inp.UserInputType == Enum.UserInputType.Keyboard then return true end
    return isGamepadInput(inp) and GAMEPAD_KEYS[inp.KeyCode] == true
end

function kbMatch(entry, kc)
    return kc and (kc == entry.kb or (entry.gp and kc == entry.gp))
end

function updateStealProgress(value, state)
    value = _clamp(tonumber(value) or 0, 0, 1)
    state = state or "READY"
    stealHud.value, stealHud.state = value, state
    if progressFill then
        progressFill.Size = UDim2.new(value, 0, 1, 0)
        progressFill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    end
    if progressPct then progressPct.Text = _floor(value * 100 + 0.5) .. "%" end
    if progressStatus then
        progressStatus.Text = "STEAL"
        progressStatus.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
end

function resetProgressBar()
    updateStealProgress(0, "READY")
end

local function doTpDown()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local _, yaw = root.CFrame:ToEulerAnglesYXZ()
        root.CFrame = _CFnew(root.Position.X, -7, root.Position.Z) * CFrame.Angles(0, yaw, 0)
        root.AssemblyLinearVelocity = _V3zero
        root.AssemblyAngularVelocity = _V3zero
    end)
end

_G._VynxRunTPDown = doTpDown
_G._VynxTPDownIsAutoOn = function() return false end

local AntiRagdollV1 = {}
AntiRagdollV1.__index = AntiRagdollV1

local BOOST_SPEED = 400
local AR_DEFAULT_SPEED = 16

local stateV1 = {
    active = false,
    isBoosting = false,
    cachedChar = nil,
    ragdollConnections = {},
}

local function disconnectAllV1()
    for _, conn in ipairs(stateV1.ragdollConnections) do
        pcall(function() conn:Disconnect() end)
    end
    stateV1.ragdollConnections = {}
end

local function cacheCharacterV1()
    local char = LP.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return false end
    stateV1.cachedChar = { character = char, humanoid = hum, root = root }
    return true
end

local function isRagdolledV1()
    if not stateV1.cachedChar or not stateV1.cachedChar.humanoid then return false end
    local hum = stateV1.cachedChar.humanoid
    local st = hum:GetState()
    local ragdollStates = {
        [Enum.HumanoidStateType.Physics] = true,
        [Enum.HumanoidStateType.Ragdoll] = true,
        [Enum.HumanoidStateType.FallingDown] = true,
    }
    return ragdollStates[st] or false
end

local function forceExitRagdollV1()
    if not stateV1.cachedChar or not stateV1.cachedChar.humanoid or not stateV1.cachedChar.root then return end
    local hum = stateV1.cachedChar.humanoid
    local root = stateV1.cachedChar.root
    pcall(function()
        LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow())
    end)
    for _, descendant in ipairs(stateV1.cachedChar.character:GetDescendants()) do
        if descendant:IsA("BallSocketConstraint") or
           (descendant:IsA("Attachment") and descendant.Name:find("RagdollAttachment")) then
            descendant:Destroy()
        end
    end
    if not stateV1.isBoosting then
        stateV1.isBoosting = true
        hum.WalkSpeed = BOOST_SPEED
    end
    if hum.Health > 0 then
        hum:ChangeState(Enum.HumanoidStateType.Running)
    end
    root.Anchored = false
end

local function heartbeatLoopV1()
    while stateV1.active do
        task.wait()
        if isRagdolledV1() then
            forceExitRagdollV1()
        elseif stateV1.isBoosting and not isRagdolledV1() then
            stateV1.isBoosting = false
            if stateV1.cachedChar and stateV1.cachedChar.humanoid then
                stateV1.cachedChar.humanoid.WalkSpeed = AR_DEFAULT_SPEED
            end
        end
    end
end

function AntiRagdollV1.start()
    if stateV1.active then return end
    AntiRagdollV1.stop()
    if not cacheCharacterV1() then
        warn("[AntiRagdollV1] No se pudo cachear el personaje")
        return
    end
    stateV1.active = true
    stateV1.isBoosting = false
    local camConn = RunService.RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        if cam and stateV1.cachedChar and stateV1.cachedChar.humanoid then
            cam.CameraSubject = stateV1.cachedChar.humanoid
        end
    end)
    table.insert(stateV1.ragdollConnections, camConn)
    local respawnConn = LP.CharacterAdded:Connect(function()
        stateV1.isBoosting = false
        task.wait(0.5)
        cacheCharacterV1()
    end)
    table.insert(stateV1.ragdollConnections, respawnConn)
    task.spawn(heartbeatLoopV1)
    print("[AntiRagdollV1] Activado")
end

function AntiRagdollV1.stop()
    stateV1.active = false
    if stateV1.isBoosting and stateV1.cachedChar and stateV1.cachedChar.humanoid then
        stateV1.cachedChar.humanoid.WalkSpeed = AR_DEFAULT_SPEED
    end
    stateV1.isBoosting = false
    disconnectAllV1()
    stateV1.cachedChar = nil
    print("[AntiRagdollV1] Desactivado")
end

function AntiRagdollV1.isRunning() return stateV1.active end

local AntiRagdollV2 = {
    Enabled = false,
    Connection = nil,
    ResetCooldown = 0,
}

local function startAntiRagdollV2()
    if AntiRagdollV2.Connection then return end
    AntiRagdollV2.Enabled = true
    AntiRagdollV2.Connection = RunService.Heartbeat:Connect(function()
        if not AntiRagdollV2.Enabled then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root then return end
        if hum.Health <= 0 or hum:GetState() == Enum.HumanoidStateType.Dead then return end
        local state = hum:GetState()
        local now = _tick()
        if state == Enum.HumanoidStateType.Physics or
           state == Enum.HumanoidStateType.Ragdoll or
           state == Enum.HumanoidStateType.FallingDown then
            if now - AntiRagdollV2.ResetCooldown > 0.15 then
                AntiRagdollV2.ResetCooldown = now
                pcall(function()
                    if hum:GetState() == Enum.HumanoidStateType.GettingUp then return end
                    if hum.Health <= 0 or hum:GetState() == Enum.HumanoidStateType.Dead then return end
                    hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                    root.Velocity = _V3zero
                    root.RotVelocity = _V3zero
                    root.AssemblyLinearVelocity = _V3zero
                    root.AssemblyAngularVelocity = _V3zero
                    for _, obj in ipairs(char:GetDescendants()) do
                        if obj:IsA("Motor6D") then obj.Enabled = true end
                        if obj:IsA("Constraint") then obj.Enabled = true end
                    end
                    workspace.CurrentCamera.CameraSubject = hum
                    local PM = LP.PlayerScripts:FindFirstChild("PlayerModule")
                    if PM then
                        local CM = require(PM:FindFirstChild("ControlModule"))
                        if CM then CM:Enable() end
                    end
                    hum.AutoRotate = true
                    hum.PlatformStand = false
                    hum.Sit = false
                end)
            end
        end
    end)
end

local function stopAntiRagdollV2()
    AntiRagdollV2.Enabled = false
    if AntiRagdollV2.Connection then
        AntiRagdollV2.Connection:Disconnect()
        AntiRagdollV2.Connection = nil
    end
    AntiRagdollV2.ResetCooldown = 0
end

function setAntiRagdollMode(mode)
    if AntiRagdollV1.isRunning() then AntiRagdollV1.stop() end
    if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
    antiRagdollMode = mode
    if mode == "v1" then AntiRagdollV1.start()
    elseif mode == "v2" then startAntiRagdollV2() end
    if _G.updateAntiRagdollUI then _G.updateAntiRagdollUI(mode) end
    saveAllSettings()
end

_antiDieEnabled = false
_antiDieStopped = false
_antiDieSources = { toggle = false, autobat = false }

_antiDie = {
    enabled = false, loop = nil, healthConn = nil, charConn = nil,
    lastHealTime = 0, invincibleUntil = 0,
    config = {
        healthThreshold = 50,
        invincibilityFrames = 0.75,
        fallDamageProtection = false,
        ragdollProtection = true,
        autoRevive = true,
    },
}

function _antiDie.SuperHeal(hum)
    if not hum or not hum.Parent then return end
    local maxHealth = hum.MaxHealth or 100
    if maxHealth <= 0 or maxHealth == math.huge then maxHealth = 100 end
    pcall(function()
        hum.Health = maxHealth
        if hum.MaxHealth < maxHealth then hum.MaxHealth = maxHealth end
    end)
    _antiDie.invincibleUntil = _tick() + _antiDie.config.invincibilityFrames
    _antiDie.lastHealTime = _tick()
    pcall(function()
        local char = hum.Parent
        if not char then return end
        for _, child in ipairs(char:GetChildren()) do
            if child:IsA("NumberValue") then
                local name = child.Name:lower()
                if name:find("health") or name:find("hp") or name:find("life") then
                    child.Value = maxHealth
                end
            end
            if child:IsA("BoolValue") and child.Name:lower():find("dead") then
                child.Value = false
            end
        end
    end)
end

function _antiDie.PreventDamage(root, hum)
    if not hum then return end
    if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
    if _tick() < _antiDie.invincibleUntil then
        if hum.Health < (hum.MaxHealth or 100) then
            hum.Health = hum.MaxHealth or 100
        end
    end
    if _antiDie.config.ragdollProtection then
        local state = hum:GetState()
        if state == Enum.HumanoidStateType.Physics or
           state == Enum.HumanoidStateType.Ragdoll or
           state == Enum.HumanoidStateType.FallingDown or
           state == Enum.HumanoidStateType.Dead then
            pcall(function()
                hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end)
            _antiDie.SuperHeal(hum)
            if root then
                pcall(function() root.AssemblyAngularVelocity = Vector3.zero end)
            end
        end
    end
    if hum.Health <= 0 then
        _antiDie.SuperHeal(hum)
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
        if root then
            pcall(function()
                root.CFrame = CFrame.new(root.Position + Vector3.new(0, 2, 0))
                root.AssemblyLinearVelocity = Vector3.zero
            end)
        end
    end
end

function _antiDie.AutoRevive()
    if not _antiDie.config.autoRevive then return end
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum then return end
    if hum.Health <= 0 then
        _antiDie.SuperHeal(hum)
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
        if root then
            pcall(function()
                root.CFrame = CFrame.new(root.Position + Vector3.new(0, 3, 0))
                root.AssemblyLinearVelocity = Vector3.zero
            end)
        end
    end
end

function _antiDie.AttachHealth(char)
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then hum = char and char:WaitForChild("Humanoid", 3) end
    if not hum then return end
    _antiDie.healthConn = hum:GetPropertyChangedSignal("Health"):Connect(function()
        if not _antiDie.enabled then return end
        if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
        if hum.Health <= 0 then _antiDie.AutoRevive() end
    end)
    if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
end

function _antiDie.StartEngine()
    if _antiDie.enabled and _antiDie.loop then return end
    _antiDie.enabled = true
    _antiDieEnabled = true
    antiDieEnabled = true
    if _antiDie.loop then _antiDie.loop:Disconnect(); _antiDie.loop = nil end
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    _antiDie.loop = RunService.Heartbeat:Connect(function()
        if not _antiDie.enabled then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum then return end
        if hum.Health <= 0 then _antiDie.AutoRevive()
        elseif hum.Health <= _antiDie.config.healthThreshold then _antiDie.SuperHeal(hum)
        elseif hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
        _antiDie.PreventDamage(root, hum)
    end)
    if LP.Character then _antiDie.AttachHealth(LP.Character) end
    if _antiDie.charConn then _antiDie.charConn:Disconnect(); _antiDie.charConn = nil end
    _antiDie.charConn = LP.CharacterAdded:Connect(function(char)
        if not _antiDie.enabled then return end
        task.wait(0.05)
        _antiDie.AttachHealth(char)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then _antiDie.SuperHeal(hum) end
    end)
end

function _antiDie.StopEngine()
    _antiDie.enabled = false
    _antiDieEnabled = false
    antiDieEnabled = false
    if _antiDie.loop then _antiDie.loop:Disconnect(); _antiDie.loop = nil end
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    if _antiDie.charConn then _antiDie.charConn:Disconnect(); _antiDie.charConn = nil end
    pcall(function()
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
        hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        hum.BreakJointsOnDeath = true
        if hum.MaxHealth == math.huge or hum.MaxHealth <= 0 then
            hum.MaxHealth = 100
            hum.Health = math.min(hum.Health, 100)
        end
    end)
end

function _antiDieSetEnabled(enabled)
    enabled = enabled == true
    if enabled == _antiDieEnabled then return end
    if enabled then
        _antiDie.StartEngine()
    else
        _antiDie.StopEngine()
    end
end

function _antiDieRefresh()
    if _antiDieStopped then return end
    local manual = _antiDieSources.toggle
    _antiDieSetEnabled(manual or _antiDieSources.autobat)
end

_G.__CrystalAntiDieSource = function(source, enabled)
    if _antiDieStopped or _antiDieSources[source] == nil then return end
    _antiDieSources[source] = enabled == true
    _antiDieRefresh()
end

_G.__CrystalAntiDieSet = function(enabled)
    _G.__CrystalAntiDieSource("autobat", enabled)
end

_G.__CrystalAntiDieIsEnabled = function() return _antiDieEnabled end
_G.__CrystalAntiDieStop = function()
    _antiDieStopped = true
    _antiDieSources.toggle = false
    _antiDieSources.autobat = false
    _antiDieSetEnabled(false)
end

function activateOnCharacter(char)
    if not _antiDieEnabled then return end
    _antiDie.AttachHealth(char)
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then _antiDie.SuperHeal(hum) end
end

AntiDieModule = {
    enabled = false,
    start = function()
        AntiDieModule.enabled = true
        antiDieEnabled = true
        _antiDieStopped = false
        _G.__CrystalAntiDieSource("toggle", true)
    end,
    stop = function()
        AntiDieModule.enabled = false
        antiDieEnabled = false
        _G.__CrystalAntiDieSource("toggle", false)
    end,
}
_G.AntiDie = AntiDieModule

local AntiBat = { Connection = nil }
local ANTIBAT_RANGE = 4000

function stopAntiBat()
    antiBatEnabled = false
    if AntiBat.Connection then
        AntiBat.Connection:Disconnect()
        AntiBat.Connection = nil
    end
    -- Desactivar source autobat del anti-die cuando se apaga el anti bate
    if _G.__CrystalAntiDieSource then
        pcall(_G.__CrystalAntiDieSource, "autobat", false)
    end
    -- Restaurar estados normales si el anti-die manual no está activo
    pcall(function()
        if _antiDieSources and not _antiDieSources.toggle then
            local char = LP and LP.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                hum.BreakJointsOnDeath = true
            end
        end
    end)
end

function startAntiBat()
    stopAntiBat()
    antiBatEnabled = true
    -- Activar anti-die automáticamente cuando anti bate está prendido
    if _G.__CrystalAntiDieSource then
        pcall(_G.__CrystalAntiDieSource, "autobat", true)
    end

    AntiBat.Connection = RunService.Heartbeat:Connect(function()
        if not antiBatEnabled then return end
        local character = LP.Character
        if not character then return end
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        -- Proteger health antes de aplicar velocidades para evitar reset
        local hum = character:FindFirstChildOfClass("Humanoid")
        if hum then
            if hum.Health <= 0 then
                pcall(function()
                    hum.Health = hum.MaxHealth or 100
                    hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                end)
                return
            end
            pcall(function()
                hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                hum.BreakJointsOnDeath = false
            end)
        end
        local v = hrp.Velocity
        hrp.Velocity = Vector3.new(
            math.random(-ANTIBAT_RANGE, ANTIBAT_RANGE),
            v.Y,
            math.random(-ANTIBAT_RANGE, ANTIBAT_RANGE)
        )
        RunService.RenderStepped:Wait()
        if hrp and hrp.Parent then
            hrp.Velocity = v
        end
        -- Verificar health después de aplicar velocidades
        if hum and hum.Parent and hum.Health <= 0 then
            pcall(function()
                hum.Health = hum.MaxHealth or 100
                hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            end)
        end
    end)
end

LP.CharacterAdded:Connect(function()
    task.wait(0.3)
    if antiBatEnabled then startAntiBat() end
end)

setAntiFlingVisual = nil

local _antiFlingState = {
    connection    = nil,
    threshold     = 80,
    spinThreshold = 40,
}
local _ANTI_FLING_DRIVE_GRACE = 0.3

local function _afHubSpeedLive()
    local live = _G.__RivalHubLiveSpeed
    if type(live) ~= "table" then return false end
    local stamp, speed = tonumber(live.t), tonumber(live.v)
    if not stamp or not speed or speed <= 0 then return false end
    return (os.clock() - stamp) <= _ANTI_FLING_DRIVE_GRACE
end

local function _afOwnMoverActive()
    if autoBatEnabled then return true end
    if autoLeftEnabled or autoRightEnabled then return true end
    return false
end

function startAntiFling()
    if _antiFlingState.connection then return end
    antiFlingEnabled = true
    _antiFlingState.connection = RunService.Heartbeat:Connect(function()
        if not antiFlingEnabled then return end
        local character = LP.Character
        if not character then return end
        local root = character:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid and (humanoid.Health <= 0 or humanoid.SeatPart) then return end

        if dropActive or _G.IsDropping then return end
        if _afOwnMoverActive() then return end
        if _afHubSpeedLive() then return end

        local velocity = root.AssemblyLinearVelocity
        local flat = Vector3.new(velocity.X, 0, velocity.Z)

        if flat.Magnitude > _antiFlingState.threshold then
            pcall(function()
                root.AssemblyLinearVelocity = Vector3.new(0, velocity.Y, 0)
                root.AssemblyAngularVelocity = Vector3.zero
            end)
            return
        end
        if root.AssemblyAngularVelocity.Magnitude > _antiFlingState.spinThreshold then
            pcall(function() root.AssemblyAngularVelocity = Vector3.zero end)
        end
    end)
end

function stopAntiFling()
    antiFlingEnabled = false
    if _antiFlingState.connection then
        _antiFlingState.connection:Disconnect()
        _antiFlingState.connection = nil
    end
end

LP.CharacterAdded:Connect(function()
    task.wait(0.3)
    if antiFlingEnabled then startAntiFling() end
end)

local espEnabled = espEnabled or false
local espActivePlayers = {}
local espLoopConn = nil
local Camera = workspace.CurrentCamera
local GUI2_ESP_COLOR = Color3.fromRGB(139, 72, 246)
local GUI2_ESP_TEXT_STROKE = Color3.fromRGB(48, 16, 92)

local function getESPThemeColors(accent)
    local c = getThemeColor() or accent or Color3.fromRGB(245, 245, 250)
    return c, Color3.fromRGB(0, 0, 0)
end

local function createESP()
    local currentFill, currentLine = getESPThemeColors()
    local esp = {
        Tracer = Drawing and Drawing.new("Line") or nil,
        Highlight = Instance.new("Highlight"),
        Lines = {},
    }
    if esp.Tracer then
        esp.Tracer.Color = currentLine
        esp.Tracer.Thickness = 3
        esp.Tracer.Transparency = 1
        esp.Tracer.ZIndex = 2
    end

    esp.Highlight.Name = "RivalHubESP_Highlight"
    esp.Highlight.FillColor = currentFill
    esp.Highlight.OutlineColor = Color3.fromRGB(0, 0, 0)
    esp.Highlight.FillTransparency = 0.12
    esp.Highlight.OutlineTransparency = 0.18
    esp.Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop

    local parentGui
    local ok, robGui = pcall(function() return game:GetService("CoreGui"):FindFirstChild("RobloxGui") end)
    if ok and robGui then parentGui = robGui else parentGui = LP:FindFirstChildOfClass("PlayerGui") end
    if not parentGui then parentGui = game:GetService("CoreGui") end
    pcall(function() esp.Highlight.Parent = parentGui end)

    local nameBillboard = Instance.new("BillboardGui")
    nameBillboard.Name = "RivalHubESP_Name"
    nameBillboard.Size = UDim2.new(0, 170, 0, 28)
    nameBillboard.StudsOffset = Vector3.new(0, 1.3, 0)
    nameBillboard.AlwaysOnTop = true
    nameBillboard.Parent = parentGui

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0, 170, 0, 28)
    nameLabel.AnchorPoint = Vector2.new(0.5, 0)
    nameLabel.Position = UDim2.new(0.5, 0, 0, 2)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = ""
    nameLabel.TextColor3 = getThemeColor()
    nameLabel.TextSize = 15
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextXAlignment = Enum.TextXAlignment.Center
    nameLabel.TextYAlignment = Enum.TextYAlignment.Center
    nameLabel.ZIndex = 5
    nameLabel.Parent = nameBillboard

    local nameOutline = Instance.new("UIStroke")
    nameOutline.Color = Color3.fromRGB(0, 0, 0)
    nameOutline.Thickness = 2.2
    nameOutline.Transparency = 0.12
    nameOutline.Parent = nameLabel

    if Drawing then
        for i = 1, 14 do
            local line = Drawing.new("Line")
            line.Color = currentLine
            line.Thickness = 2
            line.Transparency = 1
            line.ZIndex = 3
            table.insert(esp.Lines, line)
        end
    end

    esp.NameTag = nameBillboard
    esp.NameLabel = nameLabel
    esp.NameOutline = nameOutline
    return esp
end

_G.__RivalHubRefreshESPTheme = function(accent, outlineColor)
    local fillColor, lineColor = getESPThemeColors(accent)
    for _, data in pairs(espActivePlayers) do
        if type(data) == "table" and data.esp then
            local esp = data.esp
            if esp.Tracer then pcall(function() esp.Tracer.Color = lineColor end) end
            if esp.Highlight then
                esp.Highlight.FillColor = fillColor
                esp.Highlight.OutlineColor = Color3.fromRGB(0, 0, 0)
            end
            if esp.NameLabel then
                esp.NameLabel.TextColor3 = getThemeColor()
                esp.NameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            end
            if esp.NameOutline then esp.NameOutline.Color = Color3.fromRGB(0, 0, 0) end
            for _, line in ipairs(esp.Lines or {}) do pcall(function() line.Color = lineColor end) end
        end
    end
end

local function removeESP(plr)
    local data = espActivePlayers[plr]
    if data then
        if data.esp.Tracer then pcall(function() data.esp.Tracer:Remove() end) end
        if data.esp.Highlight then pcall(function() data.esp.Highlight:Destroy() end) end
        if data.esp.NameTag then pcall(function() data.esp.NameTag:Destroy() end) end
        for _, line in ipairs(data.esp.Lines) do
            pcall(function() line:Remove() end)
        end
        espActivePlayers[plr] = nil
    end
end

local function updateESP(plr, esp)
    if not espEnabled then
        if esp.Tracer then esp.Tracer.Visible = false end
        esp.Highlight.Enabled = false
        for _, line in ipairs(esp.Lines) do line.Visible = false end
        if esp.NameTag then esp.NameTag.Enabled = false end
        return
    end

    local character = plr.Character
    local hum = character and character:FindFirstChildOfClass("Humanoid")
    local trackPart = character and (
        character:FindFirstChild("HumanoidRootPart")
        or character.PrimaryPart
        or character:FindFirstChild("UpperTorso")
        or character:FindFirstChild("Torso")
        or character:FindFirstChild("Head")
    )

    if character and trackPart and (not hum or hum.Health > 0) then
        esp.Highlight.Adornee = character
        esp.Highlight.Enabled = espEnabled
        if esp.Tracer then esp.Tracer.Visible = false end
        for _, line in ipairs(esp.Lines) do line.Visible = false end

        if esp.NameTag and esp.NameLabel then
            pcall(function()
                esp.NameTag.Adornee = trackPart
                esp.NameTag.Enabled = espEnabled
                esp.NameLabel.Text = plr.Name or plr.DisplayName or ""
                esp.NameLabel.TextColor3 = getThemeColor()
            end)
        end
    else
        if esp.Tracer then esp.Tracer.Visible = false end
        esp.Highlight.Enabled = false
        esp.Highlight.Adornee = nil
        for _, line in ipairs(esp.Lines) do line.Visible = false end
        if esp.NameTag then esp.NameTag.Enabled = false end
    end
end

local function createESPForPlayer(plr)
    if plr == LP then return end
    if espActivePlayers[plr] and espActivePlayers[plr].esp then
        if espActivePlayers[plr].esp.NameTag then
            pcall(function() espActivePlayers[plr].esp.NameTag.Enabled = espEnabled end)
        end
        return
    end
    removeESP(plr)
    local esp = createESP()
    espActivePlayers[plr] = { esp = esp }
end

local espJoinConn, espLeaveConn
local function enableESP()
    if espEnabled then return end
    espEnabled = true
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LP then task.spawn(function() createESPForPlayer(plr) end) end
    end
    local espAccumulator = 0
    if espLoopConn then espLoopConn:Disconnect() end
    espLoopConn = RunService.Heartbeat:Connect(function(dt)
        espAccumulator = espAccumulator + (dt or 0)
        if espAccumulator < 0.15 then return end
        espAccumulator = 0
        for plr, data in pairs(espActivePlayers) do
            if typeof(plr) == "Instance" and plr:IsA("Player") and data and data.esp then
                pcall(function() updateESP(plr, data.esp) end)
            end
        end
    end)
    espJoinConn = Players.PlayerAdded:Connect(function(plr)
        if not espEnabled or plr == LP then return end
        task.wait(0.3)
        createESPForPlayer(plr)
    end)
    espLeaveConn = Players.PlayerRemoving:Connect(function(plr) removeESP(plr) end)
end

local function disableESP()
    espEnabled = false
    if espLoopConn then espLoopConn:Disconnect(); espLoopConn = nil end
    if espJoinConn then pcall(function() espJoinConn:Disconnect() end); espJoinConn = nil end
    if espLeaveConn then pcall(function() espLeaveConn:Disconnect() end); espLeaveConn = nil end
    for plr in pairs(espActivePlayers) do
        if typeof(plr) == "Instance" and plr:IsA("Player") and plr ~= LP then removeESP(plr) end
    end
end

function toggleESP(on)
    if on then enableESP() else disableESP() end
    if setESPVIsual then setESPVIsual(on) end
end

local _espLineGui = nil
local _espLineFrame = nil
local _espLineOutline = nil
local _espLineConn = nil
local _espLineState = { scanElapsed = math.huge, targetRoot = nil }

local function ensureESPLineGui()
    if _espLineGui and _espLineGui.Parent then return end
    _espLineGui = Instance.new("ScreenGui")
    _espLineGui.Name = "RivalHubESPLine"
    _espLineGui.IgnoreGuiInset = true
    _espLineGui.ResetOnSpawn = false
    _espLineGui.DisplayOrder = 1000
    _espLineGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local okP = pcall(function() _espLineGui.Parent = game:GetService("CoreGui") end)
    if not okP then _espLineGui.Parent = LP:WaitForChild("PlayerGui") end

    _espLineFrame = Instance.new("Frame")
    _espLineFrame.Name = "ESPLine"
    _espLineFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    _espLineFrame.BorderSizePixel = 0
    _espLineFrame.Active = false
    _espLineFrame.Visible = false
    _espLineFrame.BackgroundColor3 = getThemeColor()
    _espLineFrame.ZIndex = 1
    _espLineFrame.Parent = _espLineGui

    _espLineOutline = Instance.new("UIStroke", _espLineFrame)
    _espLineOutline.Color = Color3.fromRGB(255, 255, 255)
    _espLineOutline.Thickness = 0.7
    _espLineOutline.Transparency = 0.45
end

function stopESPLine()
    espLineEnabled = false
    if _espLineConn then _espLineConn:Disconnect(); _espLineConn = nil end
    if _espLineFrame then _espLineFrame.Visible = false end
    _espLineState.targetRoot = nil
    _espLineState.scanElapsed = math.huge
    if _espLineGui and _espLineGui.Parent then
        pcall(function() _espLineGui:Destroy() end)
    end
    _espLineGui = nil
    _espLineFrame = nil
    _espLineOutline = nil
end

function startESPLine()
    espLineEnabled = true
    ensureESPLineGui()
    if _espLineConn then _espLineConn:Disconnect() end
    _espLineState.scanElapsed = math.huge
    _espLineState.targetRoot = nil

    _espLineConn = RunService.RenderStepped:Connect(function(dt)
        if not espLineEnabled then
            if _espLineFrame then _espLineFrame.Visible = false end
            return
        end
        local ok = pcall(function()
            local activeCamera = workspace.CurrentCamera
            local myChar = LP.Character
            local myRoot = myChar and (myChar:FindFirstChild("LowerTorso") or myChar:FindFirstChild("HumanoidRootPart") or myChar:FindFirstChild("Torso"))
            if not activeCamera or not myRoot then
                if _espLineFrame then _espLineFrame.Visible = false end
                return
            end

            local myPosition = myRoot.Position
            _espLineState.scanElapsed = _espLineState.scanElapsed + (dt or 0)
            local cachedRoot = _espLineState.targetRoot
            local cachedCharacter = cachedRoot and cachedRoot.Parent
            local cachedHumanoid = cachedCharacter and cachedCharacter:FindFirstChildOfClass("Humanoid")
            local cachedValid = cachedRoot and cachedRoot.Parent and (not cachedHumanoid or cachedHumanoid.Health > 0)
            if _espLineState.scanElapsed >= 0.12 or not cachedValid then
                _espLineState.scanElapsed = 0
                local closestRoot, closestDistanceSq = nil, math.huge
                for _, otherPlayer in ipairs(Players:GetPlayers()) do
                    if otherPlayer ~= LP then
                        local character = otherPlayer.Character
                        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
                        local root = character and (character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("LowerTorso") or character:FindFirstChild("Torso"))
                        if root and root:IsA("BasePart") and (not humanoid or humanoid.Health > 0) then
                            local delta = root.Position - myPosition
                            local distanceSq = delta:Dot(delta)
                            if distanceSq < closestDistanceSq then
                                closestRoot, closestDistanceSq = root, distanceSq
                            end
                        end
                    end
                end
                _espLineState.targetRoot = closestRoot
            end

            local closestRoot = _espLineState.targetRoot
            if not closestRoot or not closestRoot.Parent then
                if _espLineFrame then _espLineFrame.Visible = false end
                return
            end
            local closestPosition = closestRoot.Position

            local from = activeCamera:WorldToViewportPoint(myPosition - Vector3.new(0, myRoot.Size.Y * 0.35, 0))
            local to, targetOnScreen = activeCamera:WorldToViewportPoint(closestPosition)
            local viewport = activeCamera.ViewportSize
            local fromPoint = Vector2.new(from.X, from.Y)
            local targetPoint = Vector2.new(to.X, to.Y)

            if not targetOnScreen or to.Z <= 0 then
                local center = viewport * 0.5
                local direction = targetPoint - center
                if to.Z <= 0 then direction = -direction end
                if direction.Magnitude < 0.001 then direction = Vector2.new(0, -1) end
                local limit = Vector2.new(math.max(8, center.X - 8), math.max(8, center.Y - 8))
                local scaleX = limit.X / math.max(math.abs(direction.X), 0.001)
                local scaleY = limit.Y / math.max(math.abs(direction.Y), 0.001)
                targetPoint = center + direction * math.min(scaleX, scaleY)
            end

            if from.Z <= 0 or fromPoint.X < 0 or fromPoint.X > viewport.X or fromPoint.Y < 0 or fromPoint.Y > viewport.Y then
                fromPoint = Vector2.new(viewport.X * 0.5, viewport.Y - 8)
            end

            local delta = targetPoint - fromPoint
            local midpoint = (fromPoint + targetPoint) * 0.5
            _espLineFrame.Position = UDim2.fromOffset(midpoint.X, midpoint.Y)
            _espLineFrame.Size = UDim2.fromOffset(math.max(delta.Magnitude, 1), 2)
            _espLineFrame.Rotation = math.deg(math.atan2(delta.Y, delta.X))
            _espLineFrame.BackgroundColor3 = getThemeColor()
            _espLineFrame.Visible = true
        end)
        if not ok and _espLineFrame then
            pcall(function() _espLineFrame.Visible = false end)
        end
    end)
end

local _enemySpeedAcc = 0
function updateEnemySpeedLabels()
    _enemySpeedAcc = _enemySpeedAcc + 1
    if _enemySpeedAcc < 6 then return end
    _enemySpeedAcc = 0
    local color = getThemeColor()
    local players = _GetPlayersCached()
    for i = 1, #players do
        local player = players[i]
        if player ~= LP then
            local char = player.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local v = hrp.AssemblyLinearVelocity
                local speed = _sqrt(v.X*v.X + v.Z*v.Z)
                local label = enemySpeedLabels[player]
                if not label then
                    local head = char:FindFirstChild("Head")
                    if head then
                        local bb = Instance.new("BillboardGui")
                        bb.Size = UDim2.new(0, 100, 0, 25)
                        bb.StudsOffset = _V3new(0, 5.5, 0)
                        bb.AlwaysOnTop = true
                        bb.Name = "EnemySpeedGui"
                        bb.Parent = head
                        local tl = Instance.new("TextLabel", bb)
                        tl.Size = UDim2.new(1, 0, 1, 0)
                        tl.BackgroundTransparency = 1
                        tl.TextColor3 = color
                        tl.Font = Enum.Font.GothamBold
                        tl.TextScaled = true
                        tl.TextStrokeTransparency = 0
                        tl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                        enemySpeedLabels[player] = tl
                        label = tl
                    end
                elseif label.Parent and label.Parent.Parent ~= char then
                    local head = char:FindFirstChild("Head")
                    if head then label.Parent.Parent = head end
                end
                if label then
                    label.Text = string.format("%.1f", speed)
                    if label.TextColor3 ~= color then label.TextColor3 = color end
                end
            else
                local label = enemySpeedLabels[player]
                if label and label.Parent and label.Parent.Parent then label.Parent.Parent = nil end
                enemySpeedLabels[player] = nil
            end
        end
    end
end

function startEnemySpeed()
    if enemySpeedConn then enemySpeedConn:Disconnect() end
    enemySpeedConn = RunService.Heartbeat:Connect(updateEnemySpeedLabels)
end

function stopEnemySpeed()
    if enemySpeedConn then enemySpeedConn:Disconnect(); enemySpeedConn = nil end
end

local function getClosestTargetBody()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local rpos = root.Position
    local closest, minDist = nil, _huge
    local plist = _GetPlayersCached()
    for i = 1, #plist do
        local plr = plist[i]
        if plr ~= LP then
            local c = plr.Character
            if c then
                local tRoot = c:FindFirstChild("HumanoidRootPart")
                if tRoot then
                    local hum = c:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local dx = tRoot.Position.X - rpos.X
                        local dy = tRoot.Position.Y - rpos.Y
                        local dz = tRoot.Position.Z - rpos.Z
                        local d = dx*dx + dy*dy + dz*dz
                        if d < minDist then minDist = d; closest = tRoot end
                    end
                end
            end
        end
    end
    return closest
end

local function _bodyLockTick()
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local target = getClosestTargetBody()
    if not target then
        if not hum.AutoRotate then hum.AutoRotate = true end
        return
    end
    local dist = (target.Position - root.Position).Magnitude
    if dist > bodyLockRange then
        if not hum.AutoRotate then hum.AutoRotate = true end
        return
    end
    if hum.AutoRotate then hum.AutoRotate = false end
    local targetVel = target.AssemblyLinearVelocity
    local speed3 = targetVel.Magnitude
    local predictTime = _clamp(speed3 / 80, 0.08, 0.35)
    local predictedPos = target.Position + targetVel * predictTime
    local targetHead = target.Parent and target.Parent:FindFirstChild("Head")
    local targetHeight = targetHead and targetHead.Position.Y or target.Position.Y
    local myHeight = root.Position.Y + (hum.HipHeight or 0)
    local heightDiff = targetHeight - myHeight
    local verticalCorrection = _clamp(heightDiff * 0.15, -1.5, 1.5)
    local flatTarget = _V3new(predictedPos.X, root.Position.Y + verticalCorrection, predictedPos.Z)
    local toPredict = flatTarget - root.Position
    if toPredict.Magnitude > 0.1 then
        local goalCF = _CFlookAt(root.Position, flatTarget)
        local diffCF = root.CFrame:Inverse() * goalCF
        local _, ry, _ = diffCF:ToEulerAnglesXYZ()
        ry = _clamp(ry, -2.5, 2.5)
        root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(_V3new(0, ry * 42, 0))
    end
end

function startBodyLock()
    if _bodyLockConn then _bodyLockConn:Disconnect() end
    local acc = 0
    _bodyLockConn = RunService.Heartbeat:Connect(function(dt)
        if not bodyLockEnabled then return end
        if _blSuppressCount > 0 then return end
        acc = acc + dt
        if acc < 0.033 then return end
        acc = 0
        _bodyLockTick()
    end)
end

function stopBodyLock()
    if _bodyLockConn then
        _bodyLockConn:Disconnect()
        _bodyLockConn = nil
    end
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then
        root.AssemblyAngularVelocity = _V3zero
        root.AssemblyLinearVelocity = _V3new(root.AssemblyLinearVelocity.X, -0.1, root.AssemblyLinearVelocity.Z)
    end
    local hum2 = c and c:FindFirstChildOfClass("Humanoid")
    if hum2 then hum2.AutoRotate = true end
end

function _suppressBodyLock()
    _blSuppressCount = _blSuppressCount + 1
    if _blSuppressCount == 1 and bodyLockEnabled then
        _blWasEnabled = true
        stopBodyLock()
        if bodyLockSetVisual then bodyLockSetVisual(false) end
        if _blRestoreTimer then
            task.cancel(_blRestoreTimer)
            _blRestoreTimer = nil
        end
        _blSmoothRestore = false
    end
end

function _unsuppressBodyLock(delayed)
    if _blSuppressCount > 0 then
        _blSuppressCount = _blSuppressCount - 1
    end
    if _blSuppressCount == 0 and _blWasEnabled then
        _blWasEnabled = false
        if _blRestoreTimer then
            pcall(task.cancel, _blRestoreTimer)
            _blRestoreTimer = nil
        end
        local function restore()
            _blRestoreTimer = nil
            if bodyLockEnabled then
                _blSmoothRestore = true
                startBodyLock()
                if bodyLockSetVisual then bodyLockSetVisual(true) end
                task.delay(0.5, function() _blSmoothRestore = false end)
            end
        end
        if delayed then
            _blRestoreTimer = task.delay(1, restore)
        else
            restore()
        end
    end
end

function setupSpeedIndicator(char)
    local head = char:WaitForChild("Head", 5)
    if not head then return end
    for _, name in ipairs({"RivalHubSpeedIndicator", "DiscordText", "RivalHubDiscordTag"}) do
        local old = head:FindFirstChild(name)
        if old then old:Destroy() end
    end
    speedLabel = nil

    -- Indicador simples: somente o modo de velocidade, sem fundo nem marca.
    local bb = Instance.new("BillboardGui", head)
    bb.Name = "RivalHubSpeedIndicator"
    bb.Size = UDim2.fromOffset(176, 42)
    bb.StudsOffset = Vector3.new(0, 3.9, 0)
    bb.AlwaysOnTop = true
    bb.LightInfluence = 0
    bb.MaxDistance = 0
    bb.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local card = Instance.new("Frame", bb)
    card.Name = "SpeedCard"
    card.Size = UDim2.fromScale(1, 1)
    card.BackgroundTransparency = 1
    card.BorderSizePixel = 0

    speedLabel = Instance.new("TextLabel", card)
    speedLabel.Name = "SpeedLabel"
    local brandLabel = Instance.new("TextLabel", card)
    brandLabel.Name = "KiraVSLabel"
    brandLabel.Position = UDim2.fromOffset(0, 0)
    brandLabel.Size = UDim2.new(1, 0, 0, 18)
    brandLabel.BackgroundTransparency = 1
    brandLabel.Text = "KIRA.VS"
    brandLabel.TextColor3 = Color3.fromRGB(255, 35, 45)
    brandLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    brandLabel.TextStrokeTransparency = 0
    brandLabel.Font = Enum.Font.GothamBlack
    brandLabel.TextSize = 14
    brandLabel.TextXAlignment = Enum.TextXAlignment.Center
    brandLabel.ZIndex = 3

    speedLabel.Position = UDim2.fromOffset(0, 18)
    speedLabel.Size = UDim2.new(1, 0, 0, 24)
    speedLabel.BackgroundTransparency = 1
    local initialMode = getSpeedModeName()
    local initialSpeed = (initialMode == "CARRY") and CS or NS
    speedLabel.Text = tostring(math.floor((tonumber(initialSpeed) or 0) + 0.5)) .. " - " .. ((initialMode == "CARRY") and "CARRY SPEED" or "NORMAL SPEED")
    speedLabel.TextColor3 = Color3.fromRGB(255, 35, 45)
    speedLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    speedLabel.TextStrokeTransparency = 0
    speedLabel.Font = Enum.Font.GothamBlack
    speedLabel.TextSize = 14
    speedLabel.TextXAlignment = Enum.TextXAlignment.Center
    speedLabel.ZIndex = 3


end

local unwalkSavedAnimate = nil

function startUnwalk()
    local c = LP.Character
    if not c then return end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if hum then
        for _, t in ipairs(hum:GetPlayingAnimationTracks()) do pcall(function() t:Stop() end) end
    end
    local anim = c:FindFirstChild("Animate")
    if anim then
        unwalkSavedAnimate = anim:Clone()
        anim:Destroy()
    end
end

function stopUnwalk()
    local c = LP.Character
    if c then
        local existing = c:FindFirstChild("Animate")
        if not existing then
            local src = game:GetService("StarterPlayer"):FindFirstChildOfClass("StarterCharacterScripts")
            local starterAnim = src and src:FindFirstChild("Animate")
            if starterAnim then
                starterAnim:Clone().Parent = c
            elseif unwalkSavedAnimate then
                unwalkSavedAnimate:Clone().Parent = c
            end
        end
    end
    unwalkSavedAnimate = nil
end

function refreshSpeedModeLabel()
    if modeValLbl then
        if laggerCarryToggled then modeValLbl.Text = "Lagger Carry"
        elseif laggerToggled then modeValLbl.Text = "Lagger Normal"
        elseif speedMode then modeValLbl.Text = "Carry"
        else modeValLbl.Text = "Normal" end
    end
    if setCarryModeVisual then setCarryModeVisual(speedMode) end
    if setLaggerModeVisual then setLaggerModeVisual(laggerToggled) end
    if setLaggerCarryVisual then setLaggerCarryVisual(laggerCarryToggled) end
end

function resetMovementState()
    refreshSpeedModeLabel()
    if mobSetCarry then mobSetCarry(speedMode) end
    if mobSetLagger1 then mobSetLagger1(laggerToggled) end
    if mobSetLagger2 then mobSetLagger2(laggerCarryToggled) end
    if setLaggerModeVisual then setLaggerModeVisual(laggerToggled) end
    if setLaggerCarryVisual then setLaggerCarryVisual(laggerCarryToggled) end
end

function toggleCarryMode()
    if laggerToggled or laggerCarryToggled then
        laggerToggled = false; laggerCarryToggled = false; speedMode = true
    else speedMode = not speedMode end
    resetMovementState()
    if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
end

function toggleLaggerMode()
    if laggerCarryToggled then laggerCarryToggled = false end
    speedMode = false; laggerToggled = not laggerToggled
    resetMovementState()
    if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
end
function toggleLaggerCarryMode()
    if laggerToggled then laggerToggled = false end
    speedMode = false; laggerCarryToggled = not laggerCarryToggled
    resetMovementState()
    if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
end

function toggleLaggerCycle()
    if speedMode then
        speedMode = false
        laggerToggled = true
        laggerCarryToggled = false
    elseif laggerToggled then
        speedMode = false
        laggerToggled = false
        laggerCarryToggled = true
    else
        speedMode = true
        laggerToggled = false
        laggerCarryToggled = false
    end
    resetMovementState()
    if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
end

function stopAutoLeft()
    if alConn then alConn:Disconnect(); alConn = nil end
    alPhase = 1
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:Move(_V3zero, false) end
    end
    if autoLeftSetVisual then autoLeftSetVisual(false) end
    if mobSetAutoLeft then mobSetAutoLeft(false) end
    _unsuppressBodyLock(true)
end

function startAutoLeft()
    if autoRightEnabled then
        autoRightEnabled = false
        stopAutoRight()
        if autoRightSetVisual then autoRightSetVisual(false) end
        if mobSetAutoRight then mobSetAutoRight(false) end
    end
    disableAllAimbots()
    _suppressBodyLock()
    if alConn then alConn:Disconnect() end
    alPhase = 1
    alConn = RunService.Heartbeat:Connect(function()
        if not autoLeftEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        local spd = NS
        if alPhase == 1 then
            local tgt = _V3new(AP.L1.X, root.Position.Y, AP.L1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                alPhase = 2
                local d = AP.L2 - root.Position
                local mv = _V3new(d.X, 0, d.Z).Unit
                hum:Move(mv, false)
                root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = AP.L1 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif alPhase == 2 then
            local tgt = _V3new(AP.L2.X, root.Position.Y, AP.L2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum:Move(_V3zero, false)
                root.AssemblyLinearVelocity = _V3zero
                autoLeftEnabled = false
                if alConn then alConn:Disconnect(); alConn = nil end
                alPhase = 1
                if autoLeftSetVisual then autoLeftSetVisual(false) end
                if mobSetAutoLeft then mobSetAutoLeft(false) end
                _unsuppressBodyLock(true)
                local facePos = _V3new(AP.L_FACE.X, root.Position.Y, AP.L_FACE.Z)
                if (facePos - root.Position).Magnitude > 0.01 then
                    root.CFrame = _CFnew(root.Position, facePos)
                end
                return
            end
            local d = AP.L2 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

function stopAutoRight()
    if arConn then arConn:Disconnect(); arConn = nil end
    arPhase = 1
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:Move(_V3zero, false) end
    end
    if autoRightSetVisual then autoRightSetVisual(false) end
    if mobSetAutoRight then mobSetAutoRight(false) end
    _unsuppressBodyLock(true)
end

function startAutoRight()
    if autoLeftEnabled then
        autoLeftEnabled = false
        stopAutoLeft()
        if autoLeftSetVisual then autoLeftSetVisual(false) end
        if mobSetAutoLeft then mobSetAutoLeft(false) end
    end
    disableAllAimbots()
    _suppressBodyLock()
    if arConn then arConn:Disconnect() end
    arPhase = 1
    arConn = RunService.Heartbeat:Connect(function()
        if not autoRightEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        local spd = NS
        if arPhase == 1 then
            local tgt = _V3new(AP.R1.X, root.Position.Y, AP.R1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                arPhase = 2
                local d = AP.R2 - root.Position
                local mv = _V3new(d.X, 0, d.Z).Unit
                hum:Move(mv, false)
                root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = AP.R1 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif arPhase == 2 then
            local tgt = _V3new(AP.R2.X, root.Position.Y, AP.R2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum:Move(_V3zero, false)
                root.AssemblyLinearVelocity = _V3zero
                autoRightEnabled = false
                if arConn then arConn:Disconnect(); arConn = nil end
                arPhase = 1
                if autoRightSetVisual then autoRightSetVisual(false) end
                if mobSetAutoRight then mobSetAutoRight(false) end
                _unsuppressBodyLock(true)
                local facePos = _V3new(AP.R_FACE.X, root.Position.Y, AP.R_FACE.Z)
                if (facePos - root.Position).Magnitude > 0.01 then
                    root.CFrame = _CFnew(root.Position, facePos)
                end
                return
            end
            local d = AP.R2 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

function getClosestTarget()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local rpos = root.Position
    local closest, minDist = nil, _huge
    local plist = _GetPlayersCached()
    for i = 1, #plist do
        local plr = plist[i]
        if plr ~= LP then
            local c = plr.Character
            if c then
                local tRoot = c:FindFirstChild("HumanoidRootPart")
                if tRoot then
                    local hum = c:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local dx = tRoot.Position.X - rpos.X
                        local dy = tRoot.Position.Y - rpos.Y
                        local dz = tRoot.Position.Z - rpos.Z
                        local d = dx*dx + dy*dy + dz*dz
                        if d < minDist then minDist = d; closest = tRoot end
                    end
                end
            end
        end
    end
    return closest
end

function trySwing()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local currentTool = char:FindFirstChildOfClass("Tool")
        if currentTool and not isBatTool(currentTool) then return end
        local bat = findBat()
        if bat then
            if bat.Parent ~= char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(bat) end) end
            end
            pcall(function() bat:Activate() end)
        end
    end)
end

do
local CONFIG = {
    FOLLOW_SPEED = 55,              
    ACTIVATE_DISTANCE = 13,         
    MIN_FOLLOW_DISTANCE = 1,        
    PREDICTION_TIME = 0.22,         
    PREDICT_AHEAD = 3,              
    MAX_VELOCITY_CHANGE = 150,      
    VELOCITY_SMOOTHING = 0.2,       
    MAX_HORIZONTAL_VELOCITY = 80,   
    SERVER_TICKRATE = 1/60,         
    MIN_PING_COMPENSATION = 0.03,   
    MAX_PING_COMPENSATION = 0.25,   
    ACCELERATION_PREDICTION_WEIGHT = 0.3,
    DIRECTION_CHANGE_DETECTION_TIME = 0.12,
    QUICK_DIRECTION_CHANGE_MULTIPLIER = 1.5,
    GRAVITY = 196.2,
    AIR_CONTROL_FACTOR = 0.8,
    AERIAL_VELOCITY_DECAY = 0.95,
    MIN_AIRBORNE_TIME = 0.08,
    FALLING_SPEED_THRESHOLD = -15,
    SWING_COOLDOWN = 0.08,
    KEYBIND = Enum.KeyCode.X,       
}

local STATE = {
    enabled = false,
    conn = nil,
    targetPlayer = nil,
    lastTargetPos = nil,
    targetVelocity = Vector3.zero,
    smoothedVelocity = Vector3.zero,
    velocityHistory = {},
    accelerationHistory = {},
    aerialVelocityHistory = {},
    verticalVelocityHistory = {},
    previousDirection = nil,
    lastDirectionChangeTime = 0,
    airborneTime = 0,
    lastYVelocity = 0,
    lastJumpTime = 0,
    lastActivationTime = 0,
    currentPing = 0.1,
    realPingMs = 0,
    previousAutoRotate = nil,
    swingLocked = false,
    nextSwingAt = 0,
}

local function getAverage(list)
    if #list == 0 then return Vector3.zero end
    local sum = Vector3.zero
    for _, v in ipairs(list) do sum = sum + v end
    return sum / #list
end

local function addToHistory(list, value, maxSize)
    table.insert(list, value)
    if #list > maxSize then table.remove(list, 1) end
end

local function resetState()
    STATE.targetPlayer = nil
    STATE.lastTargetPos = nil
    STATE.targetVelocity = Vector3.zero
    STATE.smoothedVelocity = Vector3.zero
    STATE.velocityHistory = {}
    STATE.accelerationHistory = {}
    STATE.aerialVelocityHistory = {}
    STATE.verticalVelocityHistory = {}
    STATE.previousDirection = nil
    STATE.airborneTime = 0
    STATE.lastYVelocity = 0
end

local function findBat()
    local char = LP.Character
    if not char then return nil end

    local equipped = char:FindFirstChildOfClass("Tool")
    if equipped then
        local name = equipped.Name:lower()
        if name:find("bat") or name:find("slap") then
            return equipped
        end
    end

    local backpack = LP:FindFirstChildOfClass("Backpack")
    if backpack then
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") then
                local name = tool.Name:lower()
                if name:find("bat") or name:find("slap") then
                    return tool
                end
            end
        end
    end
    
    return nil
end

local function findNearestEnemy(myRoot)
    local nearest, nearestDist = nil, math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP and player.Character then
            local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local targetHum = player.Character:FindFirstChildOfClass("Humanoid")
            if targetRoot and targetHum and targetHum.Health > 0 then
                local dist = (targetRoot.Position - myRoot.Position).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    nearest = player
                end
            end
        end
    end
    return nearest, nearestDist
end

local function faceTarget(myRoot, direction)
    if direction.Magnitude < 0.01 then return end
    local cross = myRoot.CFrame.LookVector:Cross(direction.Unit)
    local angle = math.asin(math.clamp(cross.Magnitude, -1, 1))
    myRoot.AssemblyAngularVelocity = cross.Magnitude > 0.01 
        and cross.Unit * angle * 80 
        or Vector3.zero
end

local function updatePing()
    local ok, ping = pcall(function()
        return game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    if ok and type(ping) == "number" then
        STATE.realPingMs = math.floor(ping)
    end
    STATE.currentPing = math.clamp(
        STATE.realPingMs / 1000,
        CONFIG.MIN_PING_COMPENSATION,
        CONFIG.MAX_PING_COMPENSATION
    )
end

local function swingBat()
    local now = tick()
    if STATE.swingLocked or now < STATE.nextSwingAt then return end
    STATE.swingLocked = true
    STATE.nextSwingAt = now + CONFIG.SWING_COOLDOWN
    
    pcall(function()
        local char = LP.Character
        if not char then return end
        
        local bat = findBat()
        if not bat then return end

        if bat.Parent ~= char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum:EquipTool(bat) end
        end

        bat:Activate()
        local remote = bat:FindFirstChildWhichIsA("RemoteEvent")
        if remote then remote:FireServer() end
    end)
    
    task.delay(CONFIG.SWING_COOLDOWN, function()
        STATE.swingLocked = false
    end)
end

local autoBatVersion = "v2" -- V2/V3 selector for the existing target hub

local function aimbotLoop(dt)
    CONFIG.FOLLOW_SPEED = tonumber(State.bypassBatSpeed) or BYPASS_AIMBOT_SPEED
    if not STATE.enabled then return end
    
    local char = LP.Character
    if not char then return end
    
    local myRoot = char:FindFirstChild("HumanoidRootPart")
    local myHum = char:FindFirstChildOfClass("Humanoid")
    if not myRoot or not myHum or myHum.Health <= 0 then return end

    myHum.AutoRotate = false

    local equippedTool = char:FindFirstChildOfClass("Tool")
    if not equippedTool then
        local bat = findBat()
        if bat and bat.Parent ~= char then
            pcall(function() myHum:EquipTool(bat) end)
        end
    end

    local target, dist = findNearestEnemy(myRoot)
    if not target or not target.Character then
        resetState()
        return
    end
    
    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
    local targetHum = target.Character:FindFirstChildOfClass("Humanoid")
    if not targetRoot or not targetHum or targetHum.Health <= 0 then
        resetState()
        return
    end
    
    if STATE.targetPlayer ~= target then resetState() end
    STATE.targetPlayer = target

    updatePing()

    local targetPos = targetRoot.Position
    local dt_clamped = math.max(dt, 1/240)
    
    if STATE.lastTargetPos then
        
        local instantVel = (targetPos - STATE.lastTargetPos) / dt_clamped
        local accel = instantVel - STATE.targetVelocity

        if accel.Magnitude > CONFIG.MAX_VELOCITY_CHANGE then
            instantVel = STATE.targetVelocity + accel.Unit * CONFIG.MAX_VELOCITY_CHANGE
        end

        local horizontalVel = Vector3.new(instantVel.X, 0, instantVel.Z)
        if horizontalVel.Magnitude > CONFIG.MAX_HORIZONTAL_VELOCITY then
            horizontalVel = horizontalVel.Unit * CONFIG.MAX_HORIZONTAL_VELOCITY
            instantVel = Vector3.new(horizontalVel.X, instantVel.Y, horizontalVel.Z)
        end

        addToHistory(STATE.accelerationHistory, (instantVel - STATE.targetVelocity) / dt_clamped, 4)
        addToHistory(STATE.velocityHistory, instantVel, 8)
        addToHistory(STATE.verticalVelocityHistory, instantVel.Y, 5)
        
        STATE.targetVelocity = instantVel
        STATE.smoothedVelocity = STATE.smoothedVelocity:Lerp(instantVel, CONFIG.VELOCITY_SMOOTHING)
    end
    STATE.lastTargetPos = targetPos

    local isAirborne = targetHum.FloorMaterial == Enum.Material.Air
    STATE.airborneTime = isAirborne and (STATE.airborneTime + dt_clamped) or 0
    
    if isAirborne and STATE.airborneTime >= CONFIG.MIN_AIRBORNE_TIME then
        addToHistory(STATE.aerialVelocityHistory, STATE.targetVelocity, 6)
    elseif not isAirborne then
        STATE.aerialVelocityHistory = {}
    end

    local smoothedVel = STATE.smoothedVelocity
    if isAirborne and #STATE.aerialVelocityHistory > 0 then
        local avgAerial = getAverage(STATE.aerialVelocityHistory)
        smoothedVel = Vector3.new(avgAerial.X, STATE.targetVelocity.Y, avgAerial.Z) * CONFIG.AIR_CONTROL_FACTOR
    end

    local isQuickTurn = false
    local horizontalTargetVel = Vector3.new(STATE.targetVelocity.X, 0, STATE.targetVelocity.Z)
    if horizontalTargetVel.Magnitude > 5 then
        local newDir = horizontalTargetVel.Unit
        if STATE.previousDirection and STATE.previousDirection:Dot(newDir) < 0.5 then
            isQuickTurn = (tick() - STATE.lastDirectionChangeTime) < CONFIG.DIRECTION_CHANGE_DETECTION_TIME
            STATE.lastDirectionChangeTime = tick()
        end
        STATE.previousDirection = newDir
    end

    local serverDelay = STATE.currentPing + CONFIG.SERVER_TICKRATE
    local adjustedDelay = isQuickTurn 
        and (serverDelay * CONFIG.QUICK_DIRECTION_CHANGE_MULTIPLIER) 
        or serverDelay
    
    local avgAccel = getAverage(STATE.accelerationHistory)
    local accelContribution = avgAccel * CONFIG.ACCELERATION_PREDICTION_WEIGHT 
        * (adjustedDelay * adjustedDelay * 0.5)
    
    local predictedPos = targetPos + smoothedVel * adjustedDelay + accelContribution

    local predictionTime = CONFIG.PREDICTION_TIME * (autoBatVersion == "v3" and 1.35 or 1.1)
    if isAirborne then
        predictedPos = predictedPos + smoothedVel * predictionTime 
            + Vector3.new(0, -0.5 * CONFIG.GRAVITY * predictionTime * predictionTime, 0)
    else
        predictedPos = predictedPos + smoothedVel * predictionTime
    end

    local horizSmoothed = Vector3.new(smoothedVel.X, 0, smoothedVel.Z)
    if horizSmoothed.Magnitude > 1 then
        local leadDistance = CONFIG.PREDICT_AHEAD * (autoBatVersion == "v3" and 1.35 or 1)
        predictedPos = predictedPos + horizSmoothed.Unit * leadDistance
    end

    local direction = predictedPos - myRoot.Position
    faceTarget(myRoot, direction)

    if (targetPos - myRoot.Position).Magnitude <= CONFIG.ACTIVATE_DISTANCE then
        if tick() - STATE.lastActivationTime >= 0.3 then
            swingBat()
            STATE.lastActivationTime = tick()
        end
    end

    if direction.Magnitude > CONFIG.MIN_FOLLOW_DISTANCE then
        local speed = CONFIG.FOLLOW_SPEED
        myRoot.AssemblyLinearVelocity = direction.Unit * speed
    else
        local currentVel = myRoot.AssemblyLinearVelocity
        myRoot.AssemblyLinearVelocity = Vector3.new(0, currentVel.Y * 0.5, 0)
    end
end

local function startAimbot()
    if STATE.enabled then return end
    STATE.enabled = true
    
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            STATE.previousHumanoid = hum
            STATE.previousAutoRotate = hum.AutoRotate
            hum.AutoRotate = false
        end
    end
    
    resetState()
    STATE.lastActivationTime = 0
    
    STATE.conn = RunService.RenderStepped:Connect(function(dt)
        if _mySession ~= _G.RivalHubSession then stopAimbotAdapt(); return end
        pcall(aimbotLoop, dt)
    end)
    
    print("[Bat Aimbot] ✅ ACTIVADO")
end

local function stopAimbot()
    if not STATE.enabled then return end
    STATE.enabled = false
    
    if STATE.conn then
        STATE.conn:Disconnect()
        STATE.conn = nil
    end
    
    local char = LP.Character
    if char then
        local myRoot = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        
        if myRoot then
            myRoot.AssemblyLinearVelocity = Vector3.zero
            myRoot.AssemblyAngularVelocity = Vector3.zero
        end
        
        local previous = STATE.previousHumanoid
        if previous and previous.Parent then
            previous.AutoRotate = STATE.previousAutoRotate == nil and true or STATE.previousAutoRotate
        elseif hum then hum.AutoRotate = true end
    end
    
    STATE.previousHumanoid = nil
    STATE.previousAutoRotate = nil
    resetState()
    
    print("[Bat Aimbot] ❌ DESACTIVADO")
end

local function toggleAimbot()
    if STATE.enabled then
        stopAimbot()
    else
        startAimbot()
    end
    return STATE.enabled
end

    startAimbotAdapt = function()
        if STATE.enabled then return end
        State.bypassBatEnabled = true
        _suppressBodyLock()
        startAimbot()
    end
    stopAimbotAdapt = function()
        local wasEnabled = STATE.enabled
        State.bypassBatEnabled = false
        stopAimbot()
        if wasEnabled then _unsuppressBodyLock(true) end
    end
    disableAutoBat = function()
        autoBatEnabled = false
        stopAimbotAdapt()
        if autoBatSetVisual then autoBatSetVisual(false) end
        if mobSetAutoBat then mobSetAutoBat(false) end
        if saveAllSettings then pcall(saveAllSettings) end
    end
    enableAutoBat = function()
        if _G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2() then
            _G.__RivalHubStopBatV2()
            if mobSetBatV2 then mobSetBatV2(false) end
        end
        if autoLeftEnabled then autoLeftEnabled = false; stopAutoLeft() end
        if autoRightEnabled then autoRightEnabled = false; stopAutoRight() end
        autoBatEnabled = true
        startAimbotAdapt()
        if autoBatSetVisual then autoBatSetVisual(true) end
        if mobSetAutoBat then mobSetAutoBat(true) end
        if saveAllSettings then pcall(saveAllSettings) end
    end
    _G.__kiraBatBypassState = STATE
end

function findBat()
    local char = LP.Character
    if not char then return nil end
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local t = char:FindFirstChild(name)
        if t and t:IsA("Tool") then return t end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(t) end) end
                return t
            end
        end
    end
    for _, ch in ipairs(char:GetChildren()) do
        if ch:IsA("Tool") and (ch.Name:lower():find("bat") or ch.Name:lower():find("slap")) then
            return ch
        end
    end
    return nil
end

function isBatTool(tool)
    if not tool then return false end
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        if tool.Name == name then return true end
    end
    return tool.Name:lower():find("bat") or tool.Name:lower():find("slap")
end

function findBatForCounter()
    local char = LP.Character
    if not char then return nil end
    local backpack = LP:FindFirstChildOfClass("Backpack")
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local tool = char:FindFirstChild(name) or (backpack and backpack:FindFirstChild(name))
        if tool then return tool end
    end
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Tool") and (child.Name:lower():find("bat") or child.Name:lower():find("slap")) then
            return child
        end
    end
    if backpack then
        for _, child in ipairs(backpack:GetChildren()) do
            if child:IsA("Tool") and (child.Name:lower():find("bat") or child.Name:lower():find("slap")) then
                return child
            end
        end
    end
    return nil
end

function swingBatForCounter(bat, character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if bat.Parent ~= character and humanoid then
        pcall(function() humanoid:EquipTool(bat) end)
        task.wait(0.05)
    end
    local remote = bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then
        pcall(function() remote:FireServer() end)
        task.wait(0.1)
        pcall(function() remote:FireServer() end)
    else
        pcall(function() bat:Activate() end)
        task.wait(0.1)
        pcall(function() bat:Activate() end)
    end
end

batCounterDebounce = false

function stopBatCounter()
    if Conns.batCounter then
        Conns.batCounter:Disconnect()
        Conns.batCounter = nil
    end
    batCounterDebounce = false
end

function startBatCounter()
    if Conns.batCounter then return end
    Conns.batCounter = RunService.Heartbeat:Connect(function()
        if not batCounterEnabled then return end
        if batCounterDebounce then return end
        local character = LP.Character
        if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        local state = humanoid:GetState()
        if state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown then
            batCounterDebounce = true
            _suppressBodyLock()
            task.spawn(function()
                task.wait(0.15)
                local bat = findBatForCounter()
                if bat then swingBatForCounter(bat, character) end
                task.wait(0.3)
                batCounterDebounce = false
                _unsuppressBodyLock(true)
            end)
        end
    end)
end

local DROP_ASCEND_DURATION = 0.2
local DROP_ASCEND_SPEED = 150
local _dropConn = nil

function stopDropBrainrot()
    dropActive = false
    if _dropConn then
        _dropConn:Disconnect()
        _dropConn = nil
    end
    for _, t in ipairs(dropConnections) do
        if type(t) == "thread" then pcall(task.cancel, t)
        elseif type(t) == "RBXScriptConnection" then pcall(t.Disconnect, t) end
    end
    dropConnections = {}
    local c = LP.Character
    if c then
        local root = c:FindFirstChild("HumanoidRootPart")
        if root then root.AssemblyLinearVelocity = _V3zero end
    end
    if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
    if mobSetDropBR then mobSetDropBR(false) end
end

function runDropBrainrot()
    if dropActive then return end
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end
    if dropMode == 1 then
        local speedH = 0
        if root then
            local vel = root.AssemblyLinearVelocity
            speedH = _V3new(vel.X, 0, vel.Z).Magnitude
        end
        local cooldown = (speedH > 5) and 0.6 or 0.25
        if _tick() - lastDropTime < cooldown then return end
        lastDropTime = _tick()
        dropActive = true
        if dropBrainrotSetVisual then dropBrainrotSetVisual(true) end
        if mobSetDropBR then mobSetDropBR(true) end
        local wasAutoBat = false
        if autoBatEnabled then
            wasAutoBat = true
            disableAutoBat()
            if autoBatSetVisual then autoBatSetVisual(false) end
            if mobSetAutoBat then mobSetAutoBat(false) end
        end
        local function finishDrop(threadRef)
            if threadRef and dropConnections then
                for i = #dropConnections, 1, -1 do
                    if dropConnections[i] == threadRef then
                        table.remove(dropConnections, i)
                        break
                    end
                end
            end
            dropActive = false
            local c = LP.Character
            if c then
                local r = c:FindFirstChild("HumanoidRootPart")
                local h = c:FindFirstChildOfClass("Humanoid")
                if r then
                    r.AssemblyLinearVelocity = _V3zero
                    r.AssemblyAngularVelocity = _V3zero
                    if r.Position.Y < -100 then
                        r.CFrame = _CFnew(r.Position.X, 5, r.Position.Z)
                    end
                    local rp = RaycastParams.new()
                    rp.FilterDescendantsInstances = {c}
                    rp.FilterType = Enum.RaycastFilterType.Exclude
                    local rr = workspace:Raycast(r.Position, _V3new(0, -2000, 0), rp)
                    if rr then
                        local off = (h and h.HipHeight or 2) + (r.Size.Y / 2)
                        r.CFrame = _CFnew(r.Position.X, rr.Position.Y + off, r.Position.Z)
                    end
                    if h and h.Health > 0 then h:ChangeState(Enum.HumanoidStateType.Running) end
                end
            end
            if wasAutoBat then
                enableAutoBat()
                if autoBatSetVisual then autoBatSetVisual(true) end
                if mobSetAutoBat then mobSetAutoBat(true) end
            end
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
        end
        local flingThread = nil
        flingThread = task.spawn(function()
            local startTime = _tick()
            while dropActive and (_tick() - startTime) < 0.25 do
                RunService.Heartbeat:Wait()
                local c = LP.Character
                local r = c and c:FindFirstChild("HumanoidRootPart")
                if not r then break end
                local vel = r.AssemblyLinearVelocity
                vel = _V3new(0, vel.Y, 0)
                r.AssemblyLinearVelocity = vel * 10000 + _V3new(0, 10000, 0)
                RunService.RenderStepped:Wait()
                if r and r.Parent then r.AssemblyLinearVelocity = vel end
                RunService.Stepped:Wait()
                if r and r.Parent then r.AssemblyLinearVelocity = vel + _V3new(0, 0.1, 0) end
            end
            finishDrop(flingThread)
        end)
        table.insert(dropConnections, flingThread)
        task.delay(0.35, function()
            if dropActive then finishDrop(flingThread) end
        end)
        return
    end

    if autoBatEnabled then
        autoBatEnabled = false
        if autoBatSetVisual then autoBatSetVisual(false) end
        if mobSetAutoBat then mobSetAutoBat(false) end
        if stopAimbotAdapt then stopAimbotAdapt() end
    end
    dropActive = true
    if dropBrainrotSetVisual then dropBrainrotSetVisual(true) end
    if mobSetDropBR then mobSetDropBR(false) end
    local t0 = _tick()
    if _dropConn then _dropConn:Disconnect() end
    _dropConn = RunService.Heartbeat:Connect(function()
        local r = char and char:FindFirstChild("HumanoidRootPart")
        if not r then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            dropActive = false
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
            return
        end
        if _tick() - t0 >= DROP_ASCEND_DURATION then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            local rp = _RayParams_new()
            rp.FilterDescendantsInstances = {char}
            rp.FilterType = Enum.RaycastFilterType.Exclude
            local rr = workspace:Raycast(r.Position, _V3new(0, -2000, 0), rp)
            if rr then
                local hum2 = char:FindFirstChildOfClass("Humanoid")
                local off = ((hum2 and hum2.HipHeight) or 2) + (r.Size.Y / 2)
                r.CFrame = _CFnew(r.Position.X, rr.Position.Y + off, r.Position.Z)
                r.AssemblyLinearVelocity = _V3zero
            end
            dropActive = false
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
            return
        end
        r.Velocity = _V3new(r.Velocity.X, DROP_ASCEND_SPEED, r.Velocity.Z)
    end)
end

function executeDropWithToggle(setVisual)
    if dropActive then return end
    task.spawn(function()
        if setVisual then setVisual(true) end
        runDropBrainrot()
        while dropActive do task.wait() end
        task.wait(0.1)
        if setVisual then setVisual(false) end
    end)
end

local _alSaved = setmetatable({}, { __mode = "k" })
local _alDetached = {}
local _alAnimators = setmetatable({}, { __mode = "k" })
local _alStoppedTracks = setmetatable({}, { __mode = "k" })
local _alGeneration = 0
local _alScanCancel = nil
local _alConns = {}
local _alLightOrig = nil
local _alRunning = false

local AL_LOW_FLAGS = {
    {"DFIntClusterSenderMaxJoinBandwidthBps", "2100000000"},
    {"DFIntClusterSenderMaxUpdateBandwidthBps", "2100000000"},
    {"DFIntServerFramesBetweenJoins", "1"},
    {"DFIntRaknetBandwidthInfluxHundredthsPercentageV2", "10000"},
    {"DFIntConnectionMTUSize", "1400"},
    {"FIntRakNetResendBufferArrayLength", "1024"},
    {"DFIntRakNetNakResendDelayMsMax", "1"},
    {"DFIntWaitOnUpdateNetworkLoopEndedMS", "100"},
    {"DFIntWaitOnRecvFromLoopEndedMS", "100"},
    {"DFIntLargePacketQueueSizeCutoffMB", "1000"},
    {"DFIntSendRakNetStatsInterval", "2147483647"},
    {"DFIntRakNetLoopMs", "1"},
    {"DFIntRakNetSelectTimeoutMs", "1"},
    {"DFIntNetworkClusterPacketCacheNumParallelTasks", "8"},
    {"DFIntReplicationDataCacheNumParallelTasks", "8"},
    {"DFIntMegaReplicatorNumParallelTasks", "16"},
    {"DFIntMaxProcessPacketsStepsPerCyclic", "512"},
    {"DFIntMaxProcessPacketsStepsAccumulated", "0"},
    {"DFIntMaxProcessPacketsJobScaling", "1000"},
    {"DFIntClientPacketMaxFrameMicroseconds", "200000"},
    {"DFIntClientPacketExcessMicroseconds", "10000"},
    {"DFIntClientPacketMinMicroseconds", "1"},
    {"DFIntClientPacketMaxDelayMs", "1"},
    {"DFIntMaxWaitTimeBeforeForcePacketProcessMS", "1"},
    {"DFIntMaxFrameBufferSize", "4"},
    {"DFIntBufferCompressionThreshold", "100"},
    {"DFIntOverrideISRReplicatorStepBandwidthBytes", "131072"},
    {"DFIntTaskSchedulerJobInitThreads", "8"},
    {"DFIntTaskSchedulerJobInGameThreads", "8"},
    {"FIntTaskSchedulerAutoThreadLimit", "16"},
    {"FIntTaskSchedulerAsyncTasksMinimumThreadCount", "4"},
    {"DFIntRuntimeConcurrency", "16"},
    {"FIntSimWorldTaskQueueParallelTasks", "20"},
    {"DFIntHttpBatchLimit", "256"},
    {"FIntHttpBatchLimit", "256"},
    {"DFIntHttpCurlConnectionCacheSize", "512"},
    {"FIntDefaultMeshCacheSizeMB", "512"},
    {"DFIntMemCacheMaxCapacityMB", "256"},
    {"DFIntNumAssetsMaxToPreload", "1"},
    {"DFFlagEnableSoundPreloading", "false"},
    {"FFlagSlimContentProvider", "true"},
    {"DFFlagDebugSkipMeshVoxelizer", "true"},
    {"DFFlagTextureQualityOverrideEnabled", "true"},
    {"DFIntTextureQualityOverride", "0"},
    {"FIntDebugTextureManagerSkipMips", "7"},
    {"DFIntDebugLimitMinTextureResolutionWhenSkipMips", "8"},
    {"FFlagTM2SkipMipsForUnstreamable2", "true"},
    {"DFFlagDoNotSkipMipsBasedOnSystemMemoryPS", "true"},
    {"FFlagRenderUseTextureManager224", "false"},
    {"DFIntDebugFRMQualityLevelOverride", "1"},
    {"DFFlagDebugPauseVoxelizer", "true"},
    {"FFlagFastGPULightCulling3", "true"},
    {"FIntRenderLocalLightFadeInMs", "0"},
    {"FIntRenderLocalLightUpdatesMax", "1"},
    {"FIntRenderShadowmapBias", "0"},
    {"FIntSSAOMipLevels", "0"},
    {"FIntDebugForceMSAASamples", "1"},
    {"FIntDebugFRMOptionalMSAALevelOverride", "0"},
    {"FIntRobloxGuiBlurIntensity", "0"},
    {"FIntFRMMinGrassDistance", "0"},
    {"FIntFRMMaxGrassDistance", "0"},
    {"DFFlagCoreScriptTelemetry2", "false"},
    {"DFFlagBrowserTrackerIdTelemetryEnabled", "false"},
    {"FFlagPerfDataOnTelemetryV2", "false"},
    {"FFlagSendRenderFidelityTelemetry2", "false"},
    {"FFlagEnableTelemetryServiceMemoryCPUInfo", "false"},
    {"DFIntTelemetryProfilerHundredthsPercentage", "0"},
    {"FIntTelemetryProfilerFrequency", "0"},
    {"FIntPerformanceTelemetryQueueProcessLimit", "0"},
    {"DFIntContentProviderPreloadHangTelemetryHundredthsPercentage", "0"},
}

local function _alApplyLowFlags()
    local env = (getgenv and getgenv()) or _G
    local setter = rawget(env, "setfflag") or rawget(_G, "set_fflag")
    local getter = rawget(env, "getfflag") or rawget(_G, "get_fflag")
    if type(setter) ~= "function" then return end
    for _, flag in ipairs(AL_LOW_FLAGS) do
        local canSet = true
        if type(getter) == "function" then
            local ok, current = pcall(getter, flag[1])
            canSet = ok and current ~= nil
        end
        if canSet then pcall(setter, flag[1], tostring(flag[2])) end
    end
end

local function _alSaveLighting()
    if _alLightOrig then return end
    _alLightOrig = {
        Brightness = Lighting.Brightness,
        ClockTime = Lighting.ClockTime,
        OutdoorAmbient = Lighting.OutdoorAmbient,
        Ambient = Lighting.Ambient,
        GlobalShadows = Lighting.GlobalShadows,
        EnvironmentDiffuseScale = Lighting.EnvironmentDiffuseScale,
        EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
        FogStart = Lighting.FogStart,
        FogEnd = Lighting.FogEnd,
    }
end

local function _alApplyLowPermanentSettings()
    pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
    pcall(function()
        UserSettings():GetService("UserGameSettings").SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel1
    end)
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.EnvironmentDiffuseScale = 0.35
        Lighting.EnvironmentSpecularScale = 0.35
        Lighting.FogStart = 0
        Lighting.FogEnd = 1e10
        Lighting.Brightness = math.max(tonumber(Lighting.Brightness) or 2, 2.5)
        local amb = Lighting.Ambient
        local out = Lighting.OutdoorAmbient
        Lighting.Ambient = Color3.fromRGB(math.max(amb.R * 255, 145), math.max(amb.G * 255, 145), math.max(amb.B * 255, 145))
        Lighting.OutdoorAmbient = Color3.fromRGB(math.max(out.R * 255, 155), math.max(out.G * 255, 155), math.max(out.B * 255, 155))
    end)
    pcall(function()
        local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
        if atmosphere then
            atmosphere.Density = 0
            atmosphere.Glare = 0
            atmosphere.Haze = 0
        end
    end)
    pcall(function()
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            terrain.Decoration = false
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
            terrain.WaterReflectance = 0
            local clouds = terrain:FindFirstChildOfClass("Clouds")
            if clouds then clouds.Enabled = false end
        end
    end)
end

local function _alChange(object, property, value)
    pcall(function()
        local original = object[property]
        if original == value then return end
        object[property] = value
        local bucket = _alSaved[object]
        if not bucket then bucket = {}; _alSaved[object] = bucket end
        if not bucket[property] then bucket[property] = { value = original } end
    end)
end

local function _alIsProtectedVisual(object)
    local char = LP.Character
    if char and (object == char or object:IsDescendantOf(char)) then return true end
    if object:GetAttribute("_AdaptDuelsSky") == true then return true end
    local parent = object
    while parent and parent ~= workspace do
        if parent:IsA("LayerCollector") or parent:IsA("GuiObject") or parent:IsA("GuiBase2d") then
            return true
        end
        local name = parent.Name or ""
        if name:match("^Crystal") or name:match("^Eclipse") or name:match("^ESP_")
            or name:match("^BloodHounds") or name:match("^RivalHub") then
            return true
        end
        parent = parent.Parent
    end
    return false
end

function applyAntiLagDerender(obj)
    pcall(function()
        if not _alRunning then return end
        if _alIsProtectedVisual(obj) then return end

        if obj:IsA("Terrain") then
            _alChange(obj, "Decoration", false)
            _alChange(obj, "WaterWaveSize", 0)
            _alChange(obj, "WaterWaveSpeed", 0)
            _alChange(obj, "WaterReflectance", 0)
            _alChange(obj, "WaterTransparency", 1)
        end
        if obj:IsA("BasePart") then
            _alChange(obj, "Material", Enum.Material.Plastic)
            _alChange(obj, "MaterialVariant", "")
            _alChange(obj, "Reflectance", 0)
            _alChange(obj, "CastShadow", false)
            if obj:IsA("MeshPart") then
                _alChange(obj, "TextureID", "")
                _alChange(obj, "RenderFidelity", Enum.RenderFidelity.Performance)
                _alChange(obj, "DoubleSided", false)
            elseif obj:IsA("PartOperation") then
                _alChange(obj, "RenderFidelity", Enum.RenderFidelity.Performance)
            end
        elseif obj:IsA("SpecialMesh") then
            _alChange(obj, "TextureId", "")
        elseif obj:IsA("SurfaceAppearance") then
            _alDetached[obj] = true
            _alChange(obj, "Parent", nil)
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            _alChange(obj, "Transparency", 1)
            _alChange(obj, "Texture", "")
        elseif obj:IsA("ParticleEmitter") then
            _alChange(obj, "Enabled", false)
            _alChange(obj, "Rate", 0)
            pcall(function() obj:Clear() end)
        elseif obj:IsA("Trail") then
            _alChange(obj, "Enabled", false)
            pcall(function() obj:Clear() end)
        elseif obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke")
            or obj:IsA("Sparkles") or obj:IsA("Light") or obj:IsA("PostEffect")
            or obj:IsA("Clouds") then
            _alChange(obj, "Enabled", false)
        elseif obj:IsA("Atmosphere") then
            _alChange(obj, "Density", 0)
            _alChange(obj, "Haze", 0)
            _alChange(obj, "Glare", 0)
        elseif obj:IsA("Sky") then
            for _, face in ipairs({"SkyboxBk","SkyboxDn","SkyboxFt","SkyboxLf","SkyboxRt","SkyboxUp","SunTextureId","MoonTextureId"}) do
                _alChange(obj, face, "")
            end
            _alChange(obj, "StarCount", 0)
            _alChange(obj, "CelestialBodiesShown", false)
        elseif obj:IsA("Shirt") then
            _alChange(obj, "ShirtTemplate", "")
        elseif obj:IsA("Pants") then
            _alChange(obj, "PantsTemplate", "")
        elseif obj:IsA("ShirtGraphic") then
            _alChange(obj, "Graphic", "")
        elseif obj:IsA("Animator") then
            if not _alAnimators[obj] then
                local char = LP.Character
                if not (char and obj:IsDescendantOf(char)) then
                    _alAnimators[obj] = true
                    local token = _alGeneration
                    local ok, conn = pcall(function()
                        return obj.AnimationPlayed:Connect(function(track)
                            if not _alRunning or _alGeneration ~= token then return end
                            task.defer(function()
                                if _alRunning and _alGeneration == token then
                                    pcall(function() track:Stop(0) end)
                                end
                            end)
                        end)
                    end)
                    if ok then table.insert(_alConns, conn) end
                    pcall(function()
                        for _, track in ipairs(obj:GetPlayingAnimationTracks()) do
                            if track.IsPlaying then
                                _alStoppedTracks[track] = {
                                    animator = obj,
                                    time = track.TimePosition,
                                    speed = track.Speed,
                                }
                                track:Stop(0)
                            end
                        end
                    end)
                end
            end
        end
    end)
end

function enableAntiLag()
    if _alRunning then return end
    _alRunning = true
    antiLagEnabled = true
    _alGeneration = _alGeneration + 1
    local token = _alGeneration

    _alSaveLighting()
    _alApplyLowFlags()
    _alApplyLowPermanentSettings()

    for _, delaySeconds in ipairs({2, 6, 12, 20}) do
        task.delay(delaySeconds, function()
            if _alRunning and _alGeneration == token then
                _alApplyLowFlags()
            end
        end)
    end

    pcall(function() game:GetService("ReplicatedFirst"):RemoveDefaultLoadingScreen() end)

    table.insert(_alConns, workspace.DescendantAdded:Connect(function(obj)
        task.defer(function()
            if _alRunning and _alGeneration == token then
                pcall(applyAntiLagDerender, obj)
            end
        end)
    end))
    table.insert(_alConns, Lighting.DescendantAdded:Connect(function(obj)
        task.defer(function()
            if _alRunning and _alGeneration == token then
                pcall(applyAntiLagDerender, obj)
            end
        end)
    end))

    local function walkBatched(roots, apply, shouldContinue)
        task.spawn(function()
            local pending = {}
            for _, root in ipairs(roots) do
                if root then
                    local ok, children = pcall(function() return root:GetChildren() end)
                    if ok then for _, c in ipairs(children) do table.insert(pending, c) end end
                end
            end
            while #pending > 0 and _alRunning and _alGeneration == token and shouldContinue() do
                local deadline = os.clock() + 0.0008
                while #pending > 0 and os.clock() < deadline do
                    local obj = table.remove(pending)
                    pcall(apply, obj)
                    local ok, children = pcall(function() return obj:GetChildren() end)
                    if ok then for _, c in ipairs(children) do table.insert(pending, c) end end
                end
                RunService.Heartbeat:Wait()
            end
        end)
    end

    walkBatched({workspace, Lighting}, applyAntiLagDerender, function()
        return _alRunning and _alGeneration == token
    end)
end

function disableAntiLag()
    if not _alRunning then
        antiLagEnabled = false
        return
    end
    _alRunning = false
    antiLagEnabled = false
    _alGeneration = _alGeneration + 1

    if _alScanCancel then pcall(_alScanCancel); _alScanCancel = nil end

    for _, conn in ipairs(_alConns) do pcall(function() conn:Disconnect() end) end
    _alConns = {}

    for object, properties in pairs(_alSaved) do
        for property, original in pairs(properties) do
            pcall(function() object[property] = original.value end)
        end
    end
    _alSaved = setmetatable({}, { __mode = "k" })
    _alDetached = {}
    _alAnimators = setmetatable({}, { __mode = "k" })

    for track, state in pairs(_alStoppedTracks) do
        pcall(function()
            if state.animator.Parent and not track.IsPlaying then
                track:Play(0, 1, state.speed)
                track.TimePosition = state.time
            end
        end)
    end
    _alStoppedTracks = setmetatable({}, { __mode = "k" })

    if _alLightOrig then
        pcall(function()
            Lighting.Brightness = _alLightOrig.Brightness
            Lighting.ClockTime = _alLightOrig.ClockTime
            Lighting.OutdoorAmbient = _alLightOrig.OutdoorAmbient
            Lighting.Ambient = _alLightOrig.Ambient
            Lighting.GlobalShadows = _alLightOrig.GlobalShadows
            Lighting.EnvironmentDiffuseScale = _alLightOrig.EnvironmentDiffuseScale
            Lighting.EnvironmentSpecularScale = _alLightOrig.EnvironmentSpecularScale
            Lighting.FogStart = _alLightOrig.FogStart
            Lighting.FogEnd = _alLightOrig.FogEnd
        end)
        _alLightOrig = nil
    end
end

function applyStretchFOV(val)
    local cam = workspace.CurrentCamera
    if cam then pcall(function() cam.FieldOfView = val end) end
end

function enableStretch()
    if stretchConn then return end
    stretchEnabled = true
    local cam = workspace.CurrentCamera
    if not cam then return end
    origFOV = cam.FieldOfView or 70
    applyStretchFOV(stretchFOV)
    stretchConn = RunService.RenderStepped:Connect(function()
        if not stretchEnabled then
            stretchConn:Disconnect()
            stretchConn = nil
            return
        end
        local c = workspace.CurrentCamera
        if c then c.CFrame = c.CFrame * _CFnew(0,0,0,1,0,0,0,0.7,0,0,0,1) end
    end)
    if stretchFovConn then stretchFovConn:Disconnect() end
    stretchFovConn = RunService.RenderStepped:Connect(function()
        if stretchEnabled then applyStretchFOV(stretchFOV)
        else stretchFovConn:Disconnect(); stretchFovConn = nil end
    end)
end

function disableStretch()
    stretchEnabled = false
    if stretchConn then stretchConn:Disconnect(); stretchConn = nil end
    if stretchFovConn then stretchFovConn:Disconnect(); stretchFovConn = nil end
    local cam = workspace.CurrentCamera
    if cam then pcall(function() cam.FieldOfView = origFOV or 70 end) end
end

local function saveLightingState()
    if _originalLighting then return end
    _originalLighting = {
        Brightness = Lighting.Brightness, ClockTime = Lighting.ClockTime,
        OutdoorAmbient = Lighting.OutdoorAmbient, GlobalShadows = Lighting.GlobalShadows,
        FogEnd = Lighting.FogEnd, FogStart = Lighting.FogStart,
        FogColor = Lighting.FogColor, Ambient = Lighting.Ambient,
        ColorCorrection = nil, Bloom = nil,
    }
    for _, e in ipairs(Lighting:GetChildren()) do
        if e:IsA("ColorCorrectionEffect") then
            _originalLighting.ColorCorrection = { Enabled = e.Enabled, Brightness = e.Brightness, Contrast = e.Contrast, Saturation = e.Saturation, TintColor = e.TintColor }
        elseif e:IsA("BloomEffect") then
            _originalLighting.Bloom = { Enabled = e.Enabled, Intensity = e.Intensity, Size = e.Size, Threshold = e.Threshold }
        end
    end
end

local function restoreLightingState()
    if not _originalLighting then return end
    local old = _originalLighting
    Lighting.Brightness = old.Brightness
    Lighting.ClockTime = old.ClockTime
    Lighting.OutdoorAmbient = old.OutdoorAmbient
    Lighting.GlobalShadows = old.GlobalShadows
    Lighting.FogEnd = old.FogEnd
    Lighting.FogStart = old.FogStart
    Lighting.FogColor = old.FogColor
    Lighting.Ambient = old.Ambient
    if old.ColorCorrection then
        local cc = Lighting:FindFirstChildOfClass("ColorCorrectionEffect")
        if cc then
            cc.Enabled = old.ColorCorrection.Enabled; cc.Brightness = old.ColorCorrection.Brightness
            cc.Contrast = old.ColorCorrection.Contrast; cc.Saturation = old.ColorCorrection.Saturation
            cc.TintColor = old.ColorCorrection.TintColor
        end
    end
    if old.Bloom then
        local bloom = Lighting:FindFirstChildOfClass("BloomEffect")
        if bloom then
            bloom.Enabled = old.Bloom.Enabled; bloom.Intensity = old.Bloom.Intensity
            bloom.Size = old.Bloom.Size; bloom.Threshold = old.Bloom.Threshold
        end
    end
end

SKY_PRESETS_LIST = {"Off","Night","Aurora","Sunset","Galaxy","Cyber","Sakura","Pink Night","Blood Moon","Emerald Dawn","Volcanic","Arctic","Midnight Ocean","Vaporwave","Toxic","Solar Eclipse","Hellscape","Heaven","Storm","Sunrise","Deep Space","Lavender Dream","Inferno","Mint Sky"}

SKY_PRESETS = {
    ["Off"]={kind="off"},
    ["Night"]={clock=22,brightness=2,ambient={110,100,130},outAmb={120,110,140},sky={stars=4000,moon=18,sun=0,moonTex=true},atm={dens=0.45,color={120,60,180},decay={60,20,100},glare=0.5,haze=1.2}},
    ["Aurora"]={clock=14,brightness=3,ambient={150,120,150},outAmb={160,130,150},atm={dens=0.55,color={255,80,200},decay={255,20,150},glare=2.5,haze=3},clouds={cover=0.7,dens=0.7,color={255,240,250}}},
    ["Sunset"]={clock=17.2,brightness=2.5,ambient={170,120,100},outAmb={180,130,110},sky={stars=0,sun=25,moon=0},atm={dens=0.5,color={255,130,60},decay={255,80,30},glare=2,haze=2.5},clouds={cover=0.55,dens=0.55,color={255,200,140}}},
    ["Galaxy"]={clock=0,brightness=1.5,ambient={70,60,100},outAmb={80,70,110},sky={stars=10000,moon=30,sun=0},atm={dens=0.15,color={40,20,80},decay={20,10,50},glare=0.3,haze=0.5}},
    ["Cyber"]={clock=21,brightness=2.2,ambient={90,130,170},outAmb={100,140,180},sky={stars=2000,moon=12},atm={dens=0.4,color={0,200,255},decay={150,0,255},glare=2,haze=2},clouds={cover=0.4,dens=0.6,color={100,200,255}}},
    ["Sakura"]={clock=11,brightness=3.5,ambient={170,150,160},outAmb={180,160,170},sky={sun=8},atm={dens=0.3,color={255,200,220},decay={255,170,200},glare=1,haze=1.5},clouds={cover=0.6,dens=0.4,color={255,250,252}}},
    ["Pink Night"]={clock=23,brightness=2.2,ambient={120,60,110},outAmb={140,70,120},sky={stars=5000,moon=22,sun=0,moonTex=true},atm={dens=0.5,color={255,80,180},decay={140,30,100},glare=0.7,haze=1.4},clouds={cover=0.3,dens=0.5,color={180,90,150}}},
    ["Blood Moon"]={clock=22.5,brightness=1.6,ambient={130,40,40},outAmb={150,50,50},sky={stars=1500,moon=28,sun=0,moonTex=true},atm={dens=0.6,color={220,30,30},decay={120,10,10},glare=1.4,haze=2},clouds={cover=0.5,dens=0.7,color={120,30,30}}},
    ["Emerald Dawn"]={clock=6.5,brightness=2.8,ambient={130,170,140},outAmb={140,180,150},sky={sun=18,moon=0,stars=0},atm={dens=0.4,color={80,200,140},decay={40,150,90},glare=1.8,haze=2.2},clouds={cover=0.5,dens=0.5,color={200,255,220}}},
    ["Volcanic"]={clock=19,brightness=2,ambient={180,80,40},outAmb={200,90,50},sky={stars=200,sun=12,moon=0},atm={dens=0.75,color={255,60,0},decay={180,20,0},glare=3,haze=3.5},clouds={cover=0.8,dens=0.9,color={120,40,20}}},
    ["Arctic"]={clock=9,brightness=3.2,ambient={200,220,235},outAmb={210,230,245},sky={sun=10,stars=0,moon=0},atm={dens=0.3,color={180,220,255},decay={140,200,240},glare=1.5,haze=1.8},clouds={cover=0.7,dens=0.6,color={250,253,255}}},
    ["Midnight Ocean"]={clock=1.5,brightness=1.7,ambient={60,90,130},outAmb={70,100,140},sky={stars=6000,moon=24,sun=0,moonTex=true},atm={dens=0.5,color={20,60,140},decay={10,30,90},glare=0.6,haze=1.5}},
    ["Vaporwave"]={clock=19.5,brightness=2.4,ambient={180,120,200},outAmb={190,130,210},sky={stars=1000,moon=14},atm={dens=0.45,color={255,100,220},decay={120,60,255},glare=2.2,haze=2.4},clouds={cover=0.55,dens=0.55,color={200,150,255}}},
    ["Toxic"]={clock=13,brightness=2.5,ambient={140,180,80},outAmb={150,190,90},atm={dens=0.55,color={100,220,40},decay={60,150,20},glare=1.8,haze=2.6},clouds={cover=0.65,dens=0.7,color={180,255,120}}},
    ["Solar Eclipse"]={clock=12,brightness=0.9,ambient={50,40,60},outAmb={60,50,70},sky={stars=3500,sun=22,moon=0},atm={dens=0.5,color={255,140,40},decay={30,20,40},glare=2.8,haze=1.8}},
    ["Hellscape"]={clock=18,brightness=1.8,ambient={200,60,30},outAmb={220,70,40},sky={stars=100,sun=30,moon=0},atm={dens=0.85,color={255,30,0},decay={120,0,0},glare=3.5,haze=4},clouds={cover=0.95,dens=0.95,color={80,20,10}}},
    ["Heaven"]={clock=12,brightness=4,ambient={240,235,210},outAmb={250,245,220},sky={sun=16,moon=0,stars=0},atm={dens=0.25,color={255,250,220},decay={255,240,200},glare=3,haze=1.5},clouds={cover=0.85,dens=0.5,color={255,255,255}}},
    ["Storm"]={clock=15,brightness=1.4,ambient={90,90,110},outAmb={100,100,120},sky={stars=0,sun=6,moon=0},atm={dens=0.65,color={80,90,120},decay={40,50,80},glare=0.5,haze=3},clouds={cover=0.95,dens=0.95,color={60,65,80}}},
    ["Sunrise"]={clock=6.2,brightness=2.8,ambient={220,180,130},outAmb={230,190,140},sky={sun=22,stars=0,moon=0},atm={dens=0.45,color={255,180,100},decay={255,140,80},glare=2.4,haze=2.2},clouds={cover=0.4,dens=0.4,color={255,220,180}}},
    ["Deep Space"]={clock=0,brightness=1,ambient={30,25,50},outAmb={40,35,60},sky={stars=15000,moon=0,sun=0},atm={dens=0.08,color={15,5,40},decay={5,0,20},glare=0.2,haze=0.3}},
    ["Lavender Dream"]={clock=18.5,brightness=2.6,ambient={180,160,220},outAmb={190,170,230},sky={stars=800,moon=16,sun=0},atm={dens=0.4,color={200,160,255},decay={160,120,220},glare=1.4,haze=1.8},clouds={cover=0.55,dens=0.5,color={220,200,255}}},
    ["Inferno"]={clock=17.5,brightness=2.2,ambient={220,100,40},outAmb={235,110,50},sky={sun=26,moon=0,stars=0},atm={dens=0.6,color={255,90,20},decay={200,40,0},glare=3,haze=3.2},clouds={cover=0.7,dens=0.7,color={200,80,40}}},
    ["Mint Sky"]={clock=10,brightness=3.2,ambient={180,230,210},outAmb={190,240,220},sky={sun=10},atm={dens=0.32,color={150,255,210},decay={100,220,180},glare=1.6,haze=1.6},clouds={cover=0.55,dens=0.45,color={240,255,250}}},
}

local function _vC3(t) return Color3.fromRGB(t[1], t[2], t[3]) end

function _v4mpClearSky()
    for _, child in ipairs(Lighting:GetChildren()) do
        if child:GetAttribute("_AdaptDuelsSky") then
            pcall(function() child:Destroy() end)
        end
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        for _, child in ipairs(terrain:GetChildren()) do
            if child:GetAttribute("_AdaptDuelsSky") then
                pcall(function() child:Destroy() end)
            end
        end
    end
end

function applyCustomSky(mode)
    _v4mpClearSky()
    local preset = SKY_PRESETS[mode]
    if not preset or preset.kind == "off" then
        Lighting.ClockTime = 14
        Lighting.Brightness = 2
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = true
        skyTheme = "Off"
        return
    end
    Lighting.FogStart = 0
    Lighting.FogEnd = 100000
    Lighting.FogColor = Color3.fromRGB(200,200,200)
    Lighting.ColorShift_Top = Color3.fromRGB(0,0,0)
    Lighting.ColorShift_Bottom = Color3.fromRGB(0,0,0)
    Lighting.GlobalShadows = true
    Lighting.ClockTime = preset.clock or 14
    Lighting.Brightness = math.max(tonumber(preset.brightness) or 2, 2.5)
    if preset.outAmb then Lighting.OutdoorAmbient = _vC3(preset.outAmb) end
    if preset.ambient then Lighting.Ambient = _vC3(preset.ambient) end
    local amb = Lighting.Ambient
    local out = Lighting.OutdoorAmbient
    Lighting.Ambient = Color3.fromRGB(math.max(amb.R * 255, 135), math.max(amb.G * 255, 135), math.max(amb.B * 255, 135))
    Lighting.OutdoorAmbient = Color3.fromRGB(math.max(out.R * 255, 145), math.max(out.G * 255, 145), math.max(out.B * 255, 145))
    if preset.sky then
        local skyInst = Instance.new("Sky")
        skyInst:SetAttribute("_AdaptDuelsSky", true)
        if preset.sky.stars then skyInst.StarCount = preset.sky.stars end
        if preset.sky.moon then skyInst.MoonAngularSize = preset.sky.moon end
        if preset.sky.sun then skyInst.SunAngularSize = preset.sky.sun end
        if preset.sky.moonTex then skyInst.MoonTextureId = "rbxasset://sky/moon.jpg" end
        skyInst.Parent = Lighting
    end
    if preset.atm then
        local atm = Instance.new("Atmosphere")
        atm:SetAttribute("_AdaptDuelsSky", true)
        atm.Density = preset.atm.dens or 0.3
        atm.Color = _vC3(preset.atm.color)
        atm.Decay = _vC3(preset.atm.decay)
        atm.Glare = preset.atm.glare or 1
        atm.Haze = preset.atm.haze or 1
        atm.Parent = Lighting
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if preset.clouds and terrain then
        local clouds = Instance.new("Clouds")
        clouds:SetAttribute("_AdaptDuelsSky", true)
        clouds.Cover = preset.clouds.cover or 0.5
        clouds.Density = preset.clouds.dens or 0.5
        clouds.Color = _vC3(preset.clouds.color)
        clouds.Parent = terrain
    end
    skyTheme = mode
end

function enableVividGraphics()
    if vividGraphicsEnabled and #_vividEffects > 0 then return end
    vividGraphicsEnabled = true
    for _, eff in ipairs(_vividEffects) do
        pcall(function() eff:Destroy() end)
    end
    _vividEffects = {}
    local color = Instance.new("ColorCorrectionEffect")
    color.Parent = Lighting
    color.Saturation = 0.6; color.Contrast = 0.4; color.Brightness = 0.05
    color.TintColor = Color3.fromRGB(255, 240, 220)
    table.insert(_vividEffects, color)
    local bloom = Instance.new("BloomEffect")
    bloom.Parent = Lighting
    bloom.Intensity = 0.8; bloom.Size = 24; bloom.Threshold = 1
    table.insert(_vividEffects, bloom)
    local atmosphere = Instance.new("Atmosphere")
    atmosphere.Parent = Lighting
    atmosphere.Density = 0.3; atmosphere.Offset = 0.25
    atmosphere.Color = Color3.fromRGB(199, 199, 255)
    atmosphere.Decay = Color3.fromRGB(106, 112, 125)
    atmosphere.Glare = 0.2; atmosphere.Haze = 1
    table.insert(_vividEffects, atmosphere)
    local sun = Instance.new("SunRaysEffect")
    sun.Parent = Lighting
    sun.Intensity = 0.2; sun.Spread = 0.8
    table.insert(_vividEffects, sun)
    local dof = Instance.new("DepthOfFieldEffect")
    dof.Parent = Lighting
    dof.FocusDistance = 25; dof.InFocusRadius = 10
    dof.NearIntensity = 0.2; dof.FarIntensity = 0.4
    table.insert(_vividEffects, dof)
    task.spawn(function()
        while vividGraphicsEnabled and color and color.Parent do
            color.Contrast = 0.35 + math.sin(tick() * 2) * 0.05
            task.wait(0.03)
        end
    end)
end

function disableVividGraphics()
    vividGraphicsEnabled = false
    for _, eff in ipairs(_vividEffects) do
        pcall(function() eff:Destroy() end)
    end
    _vividEffects = {}
end

function toggleVividGraphics(on)
    if on then enableVividGraphics() else disableVividGraphics() end
    saveAllSettings(true)
end

function applyHideButtons(state)
    hideButtonsEnabled = state
    if MobilePanel then
        for _, child in ipairs(MobilePanel:GetChildren()) do
            if child:IsA("TextButton") then
                child.Visible = not state
            end
        end
    end
end

function toggleHideButtons(on)
    applyHideButtons(on)
    if setHideButtonsVisual then setHideButtonsVisual(on) end
    saveAllSettings(true)
end

function kiraUI.new(class, parent, props)
    local object = Instance.new(class)
    for key, value in pairs(props or {}) do object[key] = value end
    object.Parent = parent
    return object
end
function kiraUI.round(object, radius)
    local corner = object:FindFirstChildOfClass("UICorner") or Instance.new("UICorner", object)
    corner.CornerRadius = UDim.new(0, radius or 12)
    return corner
end
function kiraUI.stroke(object, color, transparency, thickness)
    return kiraUI.new("UIStroke", object, {Color = color, Transparency = transparency or 0,
        Thickness = thickness or 1, ApplyStrokeMode = Enum.ApplyStrokeMode.Border})
end
function kiraUI.text(parent, text, position, size, fontSize, color, z)
    return kiraUI.new("TextLabel", parent, {Text = text, Position = position, Size = size,
        BackgroundTransparency = 1, TextColor3 = color or Color3.fromRGB(245, 241, 235),
        Font = Enum.Font.GothamMedium, TextSize = fontSize or 12, ZIndex = z or 8,
        TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd})
end
function kiraUI.pumpkin(parent, position, size, z)
    -- Substitui o ícone de abóbora pelo avatar do jogador que executou o script.
    local avatar = kiraUI.new("ImageLabel", parent, {Name = "PlayerAvatar", Size = UDim2.fromOffset(size, size),
        Position = position, BackgroundColor3 = Color3.fromRGB(24, 27, 34), BackgroundTransparency = 0,
        BorderSizePixel = 0, Image = "", ScaleType = Enum.ScaleType.Crop, ZIndex = z, Active = false})
    kiraUI.round(avatar, size)
    kiraUI.stroke(avatar, getThemeColor(), 0.2, 1.5)
    task.spawn(function()
        local ok, image = pcall(function()
            return Players:GetUserThumbnailAsync(LP.UserId, Enum.ThumbnailType.AvatarBust, Enum.ThumbnailSize.Size150x150)
        end)
        if ok and avatar and avatar.Parent then avatar.Image = image end
    end)
    return avatar
end
function kiraUI.web(parent, position, size, z)
    local root = kiraUI.new("Frame", parent, {Name = "Web", Position = position,
        Size = UDim2.fromOffset(size, size), BackgroundTransparency = 1, ZIndex = z})
    local function line(x1, y1, x2, y2)
        local dx, dy = x2 - x1, y2 - y1
        kiraUI.new("Frame", root, {Size = UDim2.fromOffset(math.sqrt(dx * dx + dy * dy), 1),
            Position = UDim2.fromOffset((x1 + x2) / 2, (y1 + y2) / 2), AnchorPoint = Vector2.new(0.5, 0.5),
            Rotation = math.deg(math.atan2(dy, dx)), BackgroundColor3 = Color3.fromRGB(220, 174, 128),
            BackgroundTransparency = 0.84, BorderSizePixel = 0, ZIndex = z})
    end
    for i = 0, 4 do
        local a = math.pi * i / 8
        line(0, 0, math.cos(a) * size, math.sin(a) * size)
    end
    for ring = 1, 3 do
        local r = size * ring / 3
        for i = 0, 3 do
            local a, b = math.pi * i / 8, math.pi * (i + 1) / 8
            line(math.cos(a) * r, math.sin(a) * r, math.cos(b) * r, math.sin(b) * r)
        end
    end
    return root
end
function kiraUI.sectionIcon(parent, name, position, size, color, z)
    local root = kiraUI.new("Frame", parent, {Name = "SectionIcon", Position = position,
        Size = UDim2.fromOffset(size, size), BackgroundTransparency = 1, ZIndex = z})
    local function line(x1, y1, x2, y2, thickness)
        local dx, dy = x2 - x1, y2 - y1
        local ink = kiraUI.new("Frame", root, {Name = "IconInk", AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.fromScale((x1 + x2) / 2, (y1 + y2) / 2),
            Size = UDim2.fromOffset(math.sqrt(dx * dx + dy * dy) * size, thickness or 1.6),
            BackgroundColor3 = color, BorderSizePixel = 0, Rotation = math.deg(math.atan2(dy, dx)), ZIndex = z})
        kiraUI.round(ink, 2)
    end
    local function box(x, y, w, h, radius, solid)
        local object = kiraUI.new("Frame", root, {Name = "IconInk", Position = UDim2.fromScale(x, y),
            Size = UDim2.fromScale(w, h), BackgroundColor3 = color, BackgroundTransparency = solid and 0 or 1,
            BorderSizePixel = 0, ZIndex = z})
        kiraUI.round(object, radius)
        if not solid then kiraUI.stroke(object, color, 0, 1.4) end
        return object
    end
    if name == "Speed" then
        line(0.18, 0.22, 0.44, 0.5); line(0.44, 0.5, 0.18, 0.78)
        line(0.52, 0.22, 0.79, 0.5); line(0.79, 0.5, 0.52, 0.78)
    elseif name == "Custom" then
        line(0.5, 0.1, 0.83, 0.24); line(0.83, 0.24, 0.78, 0.62)
        line(0.78, 0.62, 0.5, 0.88); line(0.5, 0.88, 0.22, 0.62)
        line(0.22, 0.62, 0.17, 0.24); line(0.17, 0.24, 0.5, 0.1)
        line(0.36, 0.47, 0.47, 0.58); line(0.47, 0.58, 0.66, 0.37)
    elseif name == "Visual" then
        box(0.15, 0.15, 0.27, 0.27, 2, false)
        box(0.58, 0.15, 0.27, 0.27, 2, true)
        box(0.15, 0.58, 0.27, 0.27, 2, true)
        box(0.58, 0.58, 0.27, 0.27, 2, false)
    elseif name == "Settings" then
        line(0.5, 0.13, 0.82, 0.3); line(0.82, 0.3, 0.82, 0.77)
        line(0.82, 0.77, 0.18, 0.77); line(0.18, 0.77, 0.18, 0.3)
        line(0.18, 0.3, 0.5, 0.13); line(0.18, 0.3, 0.82, 0.3)
        line(0.5, 0.39, 0.5, 0.67); line(0.37, 0.54, 0.5, 0.67); line(0.63, 0.54, 0.5, 0.67)
    elseif name == "Keybinds" then
        box(0.08, 0.22, 0.84, 0.56, 3, false)
        box(0.2, 0.37, 0.12, 0.12, 1, true)
        box(0.44, 0.37, 0.12, 0.12, 1, true)
        box(0.68, 0.37, 0.12, 0.12, 1, true)
        line(0.29, 0.64, 0.71, 0.64)
    end
    return root
end
function kiraUI.colorSectionIcon(root, color)
    if not root then return end
    for _, object in ipairs(root:GetDescendants()) do
        if object:IsA("Frame") and object.Name == "IconInk" then object.BackgroundColor3 = color end
        if object:IsA("UIStroke") then object.Color = color end
    end
end

function kiraUI.fit()
    if not mainUIScale or not main or not main.Parent then return end
    local cam = Workspace.CurrentCamera
    if not cam then return end
    local viewport = cam.ViewportSize
    local width = 350
    local scale = math.max(0.25, math.min(uiScaleValue / 100, (viewport.X - 24) / width, (viewport.Y - 24) / 460))
    local height = math.min(570, math.floor((viewport.Y - 24) / scale))
    mainUIScale.Scale = scale
    kiraUI.width, kiraUI.height = width, height
    main.Size = UDim2.fromOffset(width, height)
    local size = Vector2.new(width * mainUIScale.Scale, height * mainUIScale.Scale)
    local point = main.AbsolutePosition
    main.Position = UDim2.fromOffset(_clamp(point.X, 8, math.max(8, viewport.X - size.X - 8)),
        _clamp(point.Y, 8, math.max(8, viewport.Y - size.Y - 8)))
    if pbFrame and pbFrame.Parent and kiraUI.hudScale then
        local compact = viewport.X < 600 or viewport.X < viewport.Y * 1.18
        local right = main.Position.X.Offset + size.X + 14
        local room = viewport.X - right - 12
        local hudScale = compact and math.min(1, (viewport.X - 24) / 264) or math.clamp(room / 264, 0.65, 1)
        kiraUI.hudScale.Scale = hudScale
        local hudWidth, hudHeight = 264 * hudScale, 86 * hudScale
        if not savedProgressBarPos then
            if compact then
                pbFrame.Position = UDim2.fromOffset((viewport.X - hudWidth) / 2,
                    math.min(viewport.Y - hudHeight - 12, main.Position.Y.Offset + size.Y + 14))
            else
                pbFrame.Position = UDim2.fromOffset(viewport.X - hudWidth - 12, 16)
            end
        else
            local point = pbFrame.AbsolutePosition
            pbFrame.Position = UDim2.fromOffset(math.clamp(point.X, 6, math.max(6, viewport.X - hudWidth - 6)),
                math.clamp(point.Y, 6, math.max(6, viewport.Y - hudHeight - 6)))
        end
    end

end
function kiraUI.effectsLoop(root)
    local state = kiraUI
    if state.effectConn then state.effectConn:Disconnect() end
    local accumulator, elapsed = 0, 0
    local connection
    connection = RunService.Heartbeat:Connect(function(dt)
        if not root.Parent or _mySession ~= _G.RivalHubSession then
            connection:Disconnect()
            if state.effectConn == connection then state.effectConn = nil end
            return
        end
        accumulator = accumulator + dt
        elapsed = elapsed + dt
        if accumulator < 1 / 24 then return end
        accumulator = 0
        local panel = root:FindFirstChild("Main")
        if not state.effects or not panel or not panel.Visible then return end
        for i, item in ipairs(state.animated) do
            if item.object.Parent then
                item.object.Position = item.position + UDim2.fromOffset(math.sin(elapsed * 0.6 + i) * (item.x or 0),
                    math.sin(elapsed * 1.1 + i) * (item.y or 4))
                if item.rotate then item.object.Rotation = math.sin(elapsed * 0.7 + i) * item.rotate end
            end
        end
        if state.titleGradient and state.titleGradient.Parent then
            state.titleGradient.Offset = Vector2.new(math.sin(elapsed * 0.45) * 0.28, 0)
        end
    end)
    state.effectConn = connection
    root.Destroying:Connect(function()
        connection:Disconnect()
        if state.effectConn == connection then state.effectConn = nil end
        for _, tween in pairs(state.tweens) do pcall(function() tween:Cancel() end) end
        if state.backgroundTween then pcall(function() state.backgroundTween:Cancel() end) end
        state.animated, state.cards, state.tweens = {}, {}, {}
    end)
end

function paintFloatingBtn(button, active)
    if not button or not button.Parent then return end
    local color = getThemeColor()
    local gradient = button:FindFirstChild("BtnGrad")
    local stroke = button:FindFirstChildOfClass("UIStroke")
    local label = button:FindFirstChild("TextLabel")
    local status = button:FindFirstChild("Status")
    local dot = button:FindFirstChild("StatusDot")
    local edge = button:FindFirstChild("AccentEdge")

    button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

    if customColorEnabled then
        if gradient then
            if active then
                gradient.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, color:Lerp(Color3.fromRGB(255, 255, 255), 0.55)),
                    ColorSequenceKeypoint.new(1, color),
                })
            else
                gradient.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, color:Lerp(Color3.fromRGB(24, 25, 32), 0.70)),
                    ColorSequenceKeypoint.new(1, color:Lerp(Color3.fromRGB(8, 8, 12), 0.88)),
                })
            end
        end
        if stroke then
            stroke.Color = active and Color3.fromRGB(255, 255, 255) or color
            stroke.Thickness = active and 1.8 or 1.2
            stroke.Transparency = active and 0.05 or 0.25
        end
        if dot then
            dot.BackgroundColor3 = active and Color3.fromRGB(255, 255, 255) or color
        end
        if label then
            label.TextColor3 = Color3.fromRGB(255, 255, 255)
            label.TextStrokeTransparency = 0.20
            label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        end
        if status then
            status.Text = active and "ON" or "OFF"
            status.TextColor3 = active and Color3.fromRGB(255, 255, 255) or color:Lerp(Color3.fromRGB(255, 255, 255), 0.65)
        end
        if edge then
            edge.BackgroundColor3 = color
            edge.BackgroundTransparency = active and 0.05 or 0.30
        end
    else
        if gradient then
            gradient.Color = active and ColorSequence.new(color:Lerp(Color3.fromRGB(255, 222, 157), 0.5), color:Lerp(Color3.fromRGB(105, 45, 18), 0.48))
                or ColorSequence.new(Color3.fromRGB(30, 34, 40), Color3.fromRGB(10, 14, 19))
        end
        local dark = Color3.fromRGB(51, 28, 17)
        if stroke then
            stroke.Color = active and Color3.fromRGB(255, 220, 162) or Color3.fromRGB(110, 94, 74)
            stroke.Thickness = active and 1.4 or 1
            stroke.Transparency = active and 0.15 or 0.54
        end
        if label then
            label.TextColor3 = active and dark or Color3.fromRGB(231, 228, 220)
            label.TextStrokeTransparency = 1
        end
        if status then
            status.Text = active and "ON" or "OFF"
            status.TextColor3 = active and dark or Color3.fromRGB(126, 132, 139)
        end
        if dot then
            dot.BackgroundColor3 = active and dark or Color3.fromRGB(65, 72, 81)
        end
        if edge then
            edge.BackgroundColor3 = color
            edge.BackgroundTransparency = active and 1 or 0.30
        end
    end
end

function applyFloatingButtonScale()
    for _, uiScale in ipairs(_floatingUIScales) do
        if uiScale and uiScale.Parent then
            uiScale.Scale = floatingButtonScale
            local button = uiScale.Parent
            if button:IsA("GuiObject") and Workspace.CurrentCamera then
                local viewport = Workspace.CurrentCamera.ViewportSize
                local size = 64 * floatingButtonScale
                button.Position = UDim2.fromOffset(_clamp(button.Position.X.Offset, 6, math.max(6, viewport.X - size - 6)),
                    _clamp(button.Position.Y.Offset, 6, math.max(6, viewport.Y - size - 6)))
            end
        end
    end
end

local function drag(f, dragZoneHeight)
    local dn, ds, sp, di = false, nil, nil, nil
    local endConn = nil
    local zone = dragZoneHeight or 99999
    local function stopDrag()
        local wasDragging = dn
        dn = false
        di = nil
        if endConn then
            endConn:Disconnect()
            endConn = nil
        end
        if wasDragging then task.defer(function() pcall(saveAllSettings) end) end
    end
    f.InputBegan:Connect(function(i)
        if uiLocked then return end
        if _isDraggingButton then return end
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            local framePos = f.AbsolutePosition
            if i.Position.Y > framePos.Y + zone then return end
            dn = true; ds = i.Position; sp = f.Position
            if endConn then endConn:Disconnect() end
            endConn = i.Changed:Connect(function()
                if i.UserInputState == Enum.UserInputState.End then stopDrag() end
            end)
        end
    end)
    f.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            stopDrag()
        end
    end)
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            stopDrag()
        end
    end)
    f.InputChanged:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then di = i end
    end)
    UIS.InputChanged:Connect(function(i)
        if i == di and dn then
            if uiLocked then stopDrag(); return end
            if _isDraggingButton then return end
            if not ds or not sp then return end
            local nX = sp.X.Offset + (i.Position.X - ds.X)
            local nY = sp.Y.Offset + (i.Position.Y - ds.Y)
            f.Position = UDim2.new(sp.X.Scale, nX, sp.Y.Scale, nY)
        end
    end)
end

_G.__RivalHubSpeedEngine = _G.__RivalHubSpeedEngine or {}
do
    local function blocked(hum)
        return not hum or hum.Health <= 0 or _isRagdollState(hum) or autoBatEnabled
            or autoLeftEnabled or autoRightEnabled or dropActive or _G.IsDropping
            or TPBatState.enabled or (_G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2())
    end
    local function applyNow()
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root or blocked(hum) then lastMoveDir = _V3zero; return end
        if useVenixCarryEngine then return end
        if _s2VelState.root ~= root then _hookVelHRP(_setupVelChecked(char)) end
        local direction = hum.MoveDirection
        if direction.Magnitude > 0 then lastMoveDir = direction
        else
            direction = nil
            if lastMoveDir.Magnitude > 0 then
                for key in pairs(MOVE_KEYS) do
                    if UIS:IsKeyDown(key) then direction = lastMoveDir; break end
                end
            end
        end
        _applyVelocitySpeed(direction, getActiveMoveSpeed(), root)
    end
    local function stop()
        if _G.__RivalHubSpeedEngine.conn then _G.__RivalHubSpeedEngine.conn:Disconnect(); _G.__RivalHubSpeedEngine.conn = nil end
        kiraVenixCarry:stop()
        lastMoveDir = _V3zero
    end
    local function start()
        stop()
        if useVenixCarryEngine then kiraVenixCarry:start() end
        _G.__RivalHubSpeedEngine.conn = RunService.RenderStepped:Connect(function()
            if _mySession ~= _G.RivalHubSession then stop(); return end
            applyNow()
        end)
    end
    _G.__RivalHubRefreshSpeedEngine = start
    _G.__RivalHubStopSpeedEngine = stop
    _G.__RivalHubApplySpeedNow = applyNow
    pcall(start)
    LP.CharacterRemoving:Connect(function() stop(); _s2VelState.root = nil end)
    LP.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        if _mySession ~= _G.RivalHubSession then return end
        _hookVelHRP(_setupVelChecked(char))
        start()
    end)
end

function setupMovementAndIndicators(char)
    if steppedConn then steppedConn:Disconnect(); steppedConn = nil end
    if movementLoop then movementLoop:Disconnect(); movementLoop = nil end

    local ccAcc = 0
    steppedConn = RunService.Heartbeat:Connect(function(dt)
        ccAcc = ccAcc + dt
        if ccAcc < 0.05 then return end
        ccAcc = 0
        local plist = _GetPlayersCached()
        for i = 1, #plist do
            local p = plist[i]
            if p ~= LP then
                local ch = p.Character
                if ch then
                    local parts = ch:GetChildren()
                    for j = 1, #parts do
                        local part = parts[j]
                        if part:IsA("BasePart") and part.CanCollide then
                            part.CanCollide = false
                        end
                    end
                end
            end
        end
    end)

    local frameCount, lastStats, statAccumulator = 0, _tick(), 0
    movementLoop = RunService.RenderStepped:Connect(function(dt)
        frameCount = frameCount + 1
        statAccumulator = statAccumulator + dt
        if statAccumulator >= 0.5 then
            local now = _tick()
            local fps = _floor(frameCount / math.max(now - lastStats, 0.01) + 0.5)
            frameCount, statAccumulator, lastStats = 0, 0, now
            local ping
            pcall(function() ping = game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue() end)
            if type(ping) ~= "number" then pcall(function() ping = LP:GetNetworkPing() * 1000 end) end
            local pingText = type(ping) == "number" and tostring(_floor(ping + 0.5)) or "--"
            stealHud.statsText = tostring(fps) .. " FPS  ·  PING: " .. pingText .. " MS"
            if stealHud.stats and stealHud.stats.Parent then stealHud.stats.Text = stealHud.statsText end
            if stealHud.mode and stealHud.mode.Parent then stealHud.mode.Text = autoStealMode end
            if kiraUI.footerStats and kiraUI.footerStats.Parent then kiraUI.footerStats.Text = stealHud.statsText end
        end
        local char2 = LP.Character
        if not char2 then return end
        local hum = char2:FindFirstChildOfClass("Humanoid")
        local hrp = char2:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end
        if not autoBatEnabled and not autoLeftEnabled and not autoRightEnabled then
            if _isRagdollState(hum) then
                lastMoveDir = _V3zero
            else
                local md = hum.MoveDirection
                if md.Magnitude > 0 then
                    lastMoveDir = md
                end
            end
        end
        if speedLabel and speedLabel.Parent then
            local currentSpeed, displayMode
            if laggerCarryToggled then
                currentSpeed, displayMode = LAGGER_CARRY_SPEED, "LAGGER CARRY"
            elseif laggerToggled then
                currentSpeed, displayMode = LAGGER_SPEED, "LAGGER NORMAL"
            elseif speedMode then
                currentSpeed, displayMode = CS, "CARRY SPEED"
            else
                currentSpeed, displayMode = NS, "NORMAL SPEED"
            end
            speedLabel.Text = tostring(math.floor((tonumber(currentSpeed) or 0) + 0.5)) .. " - " .. displayMode
        end
    end)
    setupSpeedIndicator(char)
    startEnemySpeed()
end

function toggleLockUI(state)
    if state == nil then uiLocked = not uiLocked else uiLocked = state end
    if setLockUIVisual then setLockUIVisual(uiLocked) end
end

function disableAllAimbots()
    if autoBatEnabled then
        disableAutoBat()
        if autoBatSetVisual then autoBatSetVisual(false) end
        if mobSetAutoBat then mobSetAutoBat(false) end
    end
end

function stopAllBackgroundTasks()
    if movementLoop then movementLoop:Disconnect(); movementLoop = nil end
    if steppedConn then steppedConn:Disconnect(); steppedConn = nil end
    stopEnemySpeed()
    if stretchEnabled then disableStretch() end
    if stretchConn then stretchConn:Disconnect(); stretchConn = nil end
    if stretchFovConn then stretchFovConn:Disconnect(); stretchFovConn = nil end
    if AntiRagdollV1.isRunning() then AntiRagdollV1.stop() end
    if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
    if antiDieEnabled then AntiDieModule.stop() end
    if antiBatEnabled then stopAntiBat() end
    if antiFlingEnabled then stopAntiFling() end
    stopBatCounter()
    if stopBatCounterV2 then stopBatCounterV2() end
    stopAutoSteal()
    disableAutoBat()
    stopAutoLeft()
    stopAutoRight()
    if _G.__RivalHubStopBatV2 then pcall(_G.__RivalHubStopBatV2) end
    if TPBatState and TPBatState.enabled and tpBatSetEnabled then pcall(tpBatSetEnabled, false) end
    if unwalkEnabled then stopUnwalk() end
    if antiLagEnabled then disableAntiLag() end
    if espEnabled then toggleESP(false) end
    if espLineEnabled then stopESPLine() end
    if dropActive then stopDropBrainrot() end
    if bodyLockEnabled then stopBodyLock() end
    _blSuppressCount = 0
    _blWasEnabled = false
    if _blRestoreTimer then pcall(task.cancel, _blRestoreTimer); _blRestoreTimer = nil end
    if _bodyLockConn then _bodyLockConn:Disconnect(); _bodyLockConn = nil end
    for _, t in ipairs(dropConnections) do
        if type(t) == "thread" then pcall(task.cancel, t)
        elseif type(t) == "RBXScriptConnection" then pcall(t.Disconnect, t) end
    end
    dropConnections = {}
    dropActive = false
    alPhase = 1
    arPhase = 1
    lastDropTime = 0
            if _G.__RivalHubStopSpeedEngine then pcall(_G.__RivalHubStopSpeedEngine) end
end

local _rivalHubSaveState = {
    lastAt = 0,
    queued = false,
    writing = false,
    dirty = false,
    revision = 0,
    retryCount = 0,
}
local function resolveRivalHubFileApi()
    local env = (type(getgenv) == "function" and getgenv()) or _G
    local synApi = rawget(env, "syn") or rawget(_G, "syn")
    local readCandidates = {rawget(env, "readfile"), synApi and synApi.readfile, rawget(_G, "readfile")}
    local writeCandidates = {rawget(env, "writefile"), synApi and synApi.writefile, rawget(_G, "writefile")}
    local readFn, writeFn
    for _, fn in ipairs(readCandidates) do if type(fn) == "function" then readFn = fn; break end end
    for _, fn in ipairs(writeCandidates) do if type(fn) == "function" then writeFn = fn; break end end
    return readFn, writeFn
end

local function rivalHubFileExists(path)
    local readFn = resolveRivalHubFileApi()
    if not readFn then return false end
    local ok, data = pcall(readFn, path)
    return ok and type(data) == "string"
end

function buildConfigTable()
    local config = {
        normalSpeed = NS, carrySpeed = CS,
        laggerSpeed1 = LAGGER_SPEED, laggerSpeed2 = LAGGER_CARRY_SPEED,
        stealRadius = CONFIG.STEAL_RANGE,
        holdMin = CONFIG.HOLD_MIN, holdMax = CONFIG.HOLD_MAX, entryDelay = CONFIG.ENTRY_DELAY, cooldown = CONFIG.COOLDOWN, primeRange = CONFIG.PRIME_RANGE,
        antiRagdollMode = antiRagdollMode, antiDieEnabled = antiDieEnabled,
        antiBat = antiBatEnabled,
        antiFling = antiFlingEnabled,
        autoSteal = CONFIG.AUTO_STEAL_ENABLED,
        autoBat = autoBatEnabled, autoLeft = autoLeftEnabled, autoRight = autoRightEnabled,
        batCounter = batCounterEnabled,
        batCounterV2 = batCounterV2Enabled, unwalkEnabled = unwalkEnabled,
        laggerToggled = laggerToggled, laggerCarryToggled = laggerCarryToggled,
        carryMode = speedMode, batAimbotSpeed = BAT_AIMBOT_SPEED,
        dropMode = dropMode, stretchEnabled = stretchEnabled, stretchFOV = stretchFOV,
        uiScale = uiScaleValue, animPack = currentAnimPack,
        espEnabled = espEnabled,
        espLineEnabled = espLineEnabled,
        vividGraphics = vividGraphicsEnabled,
        hideButtons = hideButtonsEnabled,
        antiLag = antiLagEnabled,
        skyTheme = skyTheme, mobileButtonPositions = savedButtonPositions,
        dropBrainrotKey = {kb = KB.DropBrainrot.kb and KB.DropBrainrot.kb.Name, gp = KB.DropBrainrot.gp and KB.DropBrainrot.gp.Name},
        autoLeftKey = {kb = KB.AutoLeft.kb and KB.AutoLeft.kb.Name, gp = KB.AutoLeft.gp and KB.AutoLeft.gp.Name},
        autoRightKey = {kb = KB.AutoRight.kb and KB.AutoRight.kb.Name, gp = KB.AutoRight.gp and KB.AutoRight.gp.Name},
        autoBatKey = {kb = KB.AutoBat.kb and KB.AutoBat.kb.Name, gp = KB.AutoBat.gp and KB.AutoBat.gp.Name},
        tpFloorKey = {kb = KB.TPFloor.kb and KB.TPFloor.kb.Name, gp = KB.TPFloor.gp and KB.TPFloor.gp.Name},
        guiHideKey = {kb = KB.GuiHide.kb and KB.GuiHide.kb.Name, gp = KB.GuiHide.gp and KB.GuiHide.gp.Name},
        carryToggleKey = {kb = KB.CarryToggle.kb and KB.CarryToggle.kb.Name, gp = KB.CarryToggle.gp and KB.CarryToggle.gp.Name},
        laggerModeKey = {kb = KB.LaggerMode.kb and KB.LaggerMode.kb.Name, gp = KB.LaggerMode.gp and KB.LaggerMode.gp.Name},
        batV2Key = {kb = KB.BatV2.kb and KB.BatV2.kb.Name, gp = KB.BatV2.gp and KB.BatV2.gp.Name},
        instaResetKey = {kb = KB.InstaReset.kb and KB.InstaReset.kb.Name, gp = KB.InstaReset.gp and KB.InstaReset.gp.Name},
        bodyLockEnabled = bodyLockEnabled, bodyLockRange = bodyLockRange,
        progressBarPos = savedProgressBarPos, lockUI = uiLocked,
        backgroundIndex = backgroundIndex,
        backgroundImageTransparency = backgroundImageTransparency,
        backgroundMode = backgroundMode, floatingButtonScale = floatingButtonScale,
        outfitIndex = currentOutfitIndex, vx7AvatarIndex = VX7A.index, themeColor = currentColorTheme, customColorEnabled = customColorEnabled,
        rivalTitleTheme = currentRivalTitleTheme,
        infiniteJumpEnabled = InfiniteJump.enabled == true,
        infiniteJumpMode = InfiniteJump.mode == "manual" and "manual" or "hold",
        autoStealMode = autoStealMode,
        useVenixCarryEngine = useVenixCarryEngine,
        softCarry = kiraVenixCarry.softStealEnabled,
        softCarrySpeed = kiraVenixCarry.softStealSpeed,
        softCarryRadius = kiraVenixCarry.softStealRadius,
        uiEffects = kiraUI.effects,
        mobileButtonsLocked = mobileButtonsLocked,
        batV2 = _G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2() or false,
    }
    if main then
        local basePos = _G.__RivalHubMainOriginalPos or main.Position
        config.mainPosition = {
            XScale = basePos.X.Scale,
            XOffset = basePos.X.Offset,
            YScale = basePos.Y.Scale,
            YOffset = basePos.Y.Offset,
        }
    end
    if pbFrame then
        config.progressBarPos = {XScale = pbFrame.Position.X.Scale, XOffset = pbFrame.Position.X.Offset, YScale = pbFrame.Position.Y.Scale, YOffset = pbFrame.Position.Y.Offset}
    end
    if MobilePanel then
        config.mobilePanelPos = {XScale = 0, XOffset = 10, YScale = 0, YOffset = 0}
        for _, btn in ipairs(MobilePanel:GetChildren()) do
            if btn:IsA("TextButton") then
                savedButtonPositions[btn.Name] = {X = btn.Position.X.Offset, Y = btn.Position.Y.Offset}
            end
        end
        config.mobileButtonPositions = savedButtonPositions
    end
    return config
end

local function scheduleRivalHubSave(delaySeconds)
    if _rivalHubSaveState.queued then return end
    _rivalHubSaveState.queued = true
    task.delay(delaySeconds or 0.25, function()
        _rivalHubSaveState.queued = false
        if _G.RivalHubRunning and not _isResetting and not _isLoading then
            pcall(saveAllSettings, true)
        end
    end)
end

function saveAllSettings(force)
    if _isResetting or _isLoading then return true end

    local config = buildConfigTable()
    config.configVersion = 2
    config.userId = LP.UserId
    local encodedOk, canonicalJSON = pcall(function() return HS:JSONEncode(config) end)
    if not encodedOk then
        warn("[Rival Hub Save] Could not encode settings.")
        return false
    end
    if not force and canonicalJSON == _lastSavedJSON and not _rivalHubSaveState.dirty then
        return true
    end
    _rivalHubSaveState.dirty = true

    if _rivalHubSaveState.writing then
        scheduleRivalHubSave(0.25)
        return true
    end
    if not force and _tick() - _rivalHubSaveState.lastAt < 0.25 then
        scheduleRivalHubSave(0.25)
        return true
    end

    local readFn, writeFn = resolveRivalHubFileApi()
    if not readFn or not writeFn then
        warn("[Rival Hub Save] Executor file API is unavailable.")
        scheduleRivalHubSave(1)
        return false
    end

    _rivalHubSaveState.writing = true
    _rivalHubSaveState.revision = _rivalHubSaveState.revision + 1
    local writeConfig = config
    writeConfig.revision = _rivalHubSaveState.revision
    writeConfig.savedAt = os.time()
    local json = HS:JSONEncode(writeConfig)
    local ok = false

    for attempt = 1, 3 do
        local attemptOk = pcall(function()
            if rivalHubFileExists(CONFIG_FILE) then
                local previous = readFn(CONFIG_FILE)
                if type(previous) == "string" then writeFn(CONFIG_FILE .. ".backup", previous) end
            end
            writeFn(CONFIG_FILE, json)
            local readBack = readFn(CONFIG_FILE)
            if readBack ~= json then error("save verification failed") end
            HS:JSONDecode(readBack)
        end)
        if attemptOk then
            ok = true
            break
        end
        if attempt < 3 then task.wait(0.1 * attempt) end
    end

    _rivalHubSaveState.writing = false
    if ok then
        _rivalHubSaveState.lastAt = _tick()
        _rivalHubSaveState.retryCount = 0
        _lastSavedJSON = canonicalJSON
        _rivalHubSaveState.dirty = false
        local latestOk, latest = pcall(function()
            local latestConfig = buildConfigTable()
            latestConfig.configVersion = 2
            latestConfig.userId = LP.UserId
            return HS:JSONEncode(latestConfig)
        end)
        if not latestOk or latest ~= _lastSavedJSON then
            _rivalHubSaveState.dirty = true
            scheduleRivalHubSave(0.25)
        end
    else
        _rivalHubSaveState.retryCount = _rivalHubSaveState.retryCount + 1
        warn("[Rival Hub Save] Write or verification failed; will retry.")
        scheduleRivalHubSave(math.min(2, 0.5 * _rivalHubSaveState.retryCount))
    end
    return ok
end

task.spawn(function()
    task.wait(5)
    while _G.RivalHubRunning do
        task.wait(2)
        if gui and main and not _isLoading then pcall(saveAllSettings) end
    end
end)
pcall(function()
    game:BindToClose(function()
        pcall(saveAllSettings, true)
    end)
end)

do
    local readFn, writeFn = resolveRivalHubFileApi()
    if readFn and writeFn then
        local legacyPaths = {"kira.json"}

        local okNew, _ = pcall(readFn, CONFIG_FILE)
        if not okNew then
            for _, oldPath in ipairs(legacyPaths) do
                local okOld, old = pcall(readFn, oldPath)
                if okOld and type(old) == "string" and #old > 0 then
                    pcall(writeFn, CONFIG_FILE, old)
                    warn("[Rival Hub] Config migrada de " .. oldPath .. " a " .. CONFIG_FILE)
                    break
                end
            end
        end
    end
end

function loadAllSettings()
    local readFn = resolveRivalHubFileApi()
    if not readFn or not rivalHubFileExists(CONFIG_FILE) then return false end
    local success, data = pcall(function() return HS:JSONDecode(readFn(CONFIG_FILE)) end)
    if not success or type(data) ~= "table" then return false end
    _isLoading = true

    local function num(v, fallback)
        local n = tonumber(v)
        if n == nil or n ~= n or n == math.huge or n == -math.huge then
            return fallback
        end
        return n
    end
    local function boolOr(v, fallback)
        if v == nil then return fallback end
        return v == true
    end

    NS                  = num(data.normalSpeed,       NS)
    CS                  = num(data.carrySpeed,        CS)
    LAGGER_SPEED        = num(data.laggerSpeed1,      LAGGER_SPEED)
    LAGGER_CARRY_SPEED  = num(data.laggerSpeed2,      LAGGER_CARRY_SPEED)
    CONFIG.STEAL_RANGE  = num(data.stealRadius,       CONFIG.STEAL_RANGE)
    CONFIG.HOLD_MIN     = num(data.holdMin,           CONFIG.HOLD_MIN)
    CONFIG.HOLD_MAX     = num(data.holdMax,           CONFIG.HOLD_MAX)
    CONFIG.ENTRY_DELAY  = num(data.entryDelay,        CONFIG.ENTRY_DELAY)
    CONFIG.COOLDOWN     = num(data.cooldown,          CONFIG.COOLDOWN)
    CONFIG.PRIME_RANGE  = num(data.primeRange,        CONFIG.PRIME_RANGE)

    if radInput then radInput.Text = tostring(CONFIG.STEAL_RANGE) end

    if num(data.configVersion, 1) >= 2 and data.lockUI ~= nil then
        uiLocked = boolOr(data.lockUI, false)
    else
        uiLocked = false
    end
    mobileButtonsLocked = false

    if data.antiRagdollMode then
        antiRagdollMode = tostring(data.antiRagdollMode)
    else
        antiRagdollMode = data.antiRagdoll and "v2" or "off"
    end
    antiDieEnabled = boolOr(data.antiDieEnabled, false)
    antiBatEnabled = boolOr(data.antiBat, false)
    antiFlingEnabled = boolOr(data.antiFling, false)
    CONFIG.AUTO_STEAL_ENABLED = boolOr(data.autoSteal, false)
    autoBatEnabled = boolOr(data.autoBat, false)
    autoLeftEnabled = boolOr(data.autoLeft, false)
    autoRightEnabled = boolOr(data.autoRight, false)
    setAutoStealMode(data.autoStealMode or "75")
    useVenixCarryEngine = boolOr(data.useVenixCarryEngine, false)
    kiraVenixCarry.softStealSpeed = _clamp(num(data.softCarrySpeed, 30), 1, 500)
    kiraVenixCarry.softStealRadius = _clamp(num(data.softCarryRadius, 10), 1, 200)
    kiraVenixCarry:setSoftStealEnabled(boolOr(data.softCarry, false))
    kiraUI.effects = boolOr(data.uiEffects, true)
    mobileButtonsLocked = boolOr(data.mobileButtonsLocked, false)
    if _G.__RivalHubRefreshSpeedEngine then pcall(_G.__RivalHubRefreshSpeedEngine) end
    InfiniteJump.mode = data.infiniteJumpMode == "manual" and "manual" or "hold"
    InfiniteJump.enabled = boolOr(data.infiniteJumpEnabled, false)
    batCounterEnabled = boolOr(data.batCounter, false)
    batCounterV2Enabled = boolOr(data.batCounterV2, false)
    unwalkEnabled = boolOr(data.unwalkEnabled, boolOr(data.unwalk, false))
    antiLagEnabled = boolOr(data.antiLag, false)
    laggerToggled = boolOr(data.laggerToggled, false)
    speedMode = boolOr(data.carryMode, false)
    laggerCarryToggled = boolOr(data.laggerCarryToggled, false)
    uiScaleValue = _clamp(num(data.uiScale, 90), 50, 150)
    if mainUIScale then mainUIScale.Scale = uiScaleValue / 100 end
    if pbScale then pbScale.Scale = uiScaleValue / 100 end
    espEnabled = boolOr(data.espEnabled, false)
    espLineEnabled = boolOr(data.espLineEnabled, false)
    if espEnabled then pcall(toggleESP, true) else pcall(toggleESP, false) end

    vividGraphicsEnabled = boolOr(data.vividGraphics, false)
    hideButtonsEnabled = boolOr(data.hideButtons, false)

    TPBatState.enabled = false

    customColorEnabled = boolOr(data.customColorEnabled, true)
    for _, setter in ipairs(_customColorToggles) do pcall(setter, customColorEnabled) end

    currentColorTheme = (data.themeColor and (COLOR_THEMES[data.themeColor] or data.themeColor == "Rainbow")) and data.themeColor or "Orange"
    selectedColor = COLOR_THEMES[currentColorTheme] or Color3.fromRGB(248, 165, 75)
    task.defer(function() updateAllUIThemeColors(selectedColor) end)

    if data.rivalTitleTheme and RIVAL_TITLE_THEMES[data.rivalTitleTheme] then
        task.defer(function() applyRivalTitleTheme(data.rivalTitleTheme) end)
    end

    skyTheme = data.skyTheme or "Off"
    if skyTheme ~= "Off" then pcall(applyCustomSky, skyTheme) end
    if skySelectorLabel then skySelectorLabel.Text = skyTheme end
    if data.animPack and ANIM_PACKS[data.animPack] then
        pcall(startAnimPack, data.animPack)
    else
        currentAnimPack = "Off"
        stopAnimPack()
    end
    local function lk(e, d)
        if not d then return end
        e.kb = d.kb and Enum.KeyCode[d.kb] or nil
        e.gp = d.gp and Enum.KeyCode[d.gp] or nil
    end
    lk(KB.DropBrainrot, data.dropBrainrotKey)
    lk(KB.AutoLeft, data.autoLeftKey)
    lk(KB.AutoRight, data.autoRightKey)
    lk(KB.AutoBat, data.autoBatKey)
    lk(KB.TPFloor, data.tpFloorKey)
    lk(KB.GuiHide, data.guiHideKey)
    lk(KB.CarryToggle, data.carryToggleKey)
    lk(KB.LaggerMode, data.laggerModeKey)
    lk(KB.BatV2, data.batV2Key)
    lk(KB.InstaReset, data.instaResetKey)
    if data.mobileButtonPositions then savedButtonPositions = data.mobileButtonPositions end
    if data.mobilePanelPos then savedMobilePanelPos = data.mobilePanelPos end
    if data.mainPosition and main and type(data.mainPosition) == "table" then
        main.Position = UDim2.new(
            num(data.mainPosition.XScale, 0),
            num(data.mainPosition.XOffset, 20),
            num(data.mainPosition.YScale, 0),
            num(data.mainPosition.YOffset, 2)
        )
    end
    if data.progressBarPos and type(data.progressBarPos) == "table" then
        savedProgressBarPos = data.progressBarPos
    end
    if data.bodyLockEnabled ~= nil then
        bodyLockEnabled = boolOr(data.bodyLockEnabled, false)
        if bodyLockEnabled then
            task.defer(function()
                if bodyLockSetVisual then bodyLockSetVisual(true) end
                startBodyLock()
            end)
        end
    end
    if data.bodyLockRange ~= nil then
        bodyLockRange = _clamp(num(data.bodyLockRange, 20), 5, 200)
        if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
    end
    dropMode = num(data.dropMode, 1)
    stretchEnabled = boolOr(data.stretchEnabled, false)
    stretchFOV = num(data.stretchFOV, 120)
    BAT_AIMBOT_SPEED = num(data.batAimbotSpeed, BAT_AIMBOT_SPEED)
    BYPASS_AIMBOT_SPEED = BAT_AIMBOT_SPEED
    if State then State.bypassBatSpeed = BYPASS_AIMBOT_SPEED end

        if type(data.backgroundIndex) == "number" and data.backgroundIndex >= 1 and data.backgroundIndex <= #backgroundImages then
        backgroundIndex = data.backgroundIndex
    end
    backgroundImageTransparency = _clamp(num(data.backgroundImageTransparency, 0), 0, 1)
    if data.backgroundMode == "None" then
        backgroundMode = "None"
    elseif type(data.backgroundMode) == "string" and data.backgroundMode ~= "" then
        backgroundMode = data.backgroundMode
    else
        backgroundMode = (BACKGROUND_OPTIONS and BACKGROUND_OPTIONS[backgroundIndex + 1]) or ("Background " .. tostring(backgroundIndex))
    end

    floatingButtonScale = _clamp(num(data.floatingButtonScale, 1), 0.5, 2)

    if type(data.outfitIndex) == "number"
       and type(OUTFITS) == "table"
       and #OUTFITS > 0
       and data.outfitIndex >= 1
       and data.outfitIndex <= #OUTFITS then
        currentOutfitIndex = data.outfitIndex
        task.defer(function()
            pcall(function() applyOutfitByIndex(currentOutfitIndex) end)
            if outfitSelectorLabel and OUTFITS[currentOutfitIndex] then
                outfitSelectorLabel.Text = OUTFITS[currentOutfitIndex].label
            end
        end)
    end

    if type(data.vx7AvatarIndex) == "number"
       and data.vx7AvatarIndex >= 1
       and data.vx7AvatarIndex <= #VX7A.labels - 1 then
        local _vx7Saved = data.vx7AvatarIndex
        task.defer(function()
            task.wait(1.5)
            pcall(VX7A.apply, _vx7Saved)
        end)
    end

    if vividGraphicsEnabled then
        task.defer(function() pcall(enableVividGraphics) end)
    else
        task.defer(function() pcall(disableVividGraphics) end)
    end
    if dropModeBtnRef then dropModeBtnRef.Text = dropMode == 1 and "Fling" or "Jump Drop" end
    refreshSpeedModeLabel()
    _lastSavedJSON = HS:JSONEncode(buildConfigTable())
    _isLoading = false

    if data.batV2 == true and _G.__RivalHubStartBatV2 then
        task.defer(function()
            pcall(_G.__RivalHubStartBatV2)
            if mobSetBatV2 then mobSetBatV2(true) end
        end)
    end

    return true
end

function forceResetUI()
    if normalBox then normalBox.Text = tostring(NS) end
    if carryBox then carryBox.Text = tostring(CS) end
    if radInput then radInput.Text = tostring(CONFIG.STEAL_RANGE) end
    if laggerBox then laggerBox.Text = tostring(LAGGER_SPEED) end
    if lagger2Box then lagger2Box.Text = tostring(LAGGER_CARRY_SPEED) end
    if batSpeedBox then batSpeedBox.Text = tostring(BYPASS_AIMBOT_SPEED) end
    if uiScaleBox then uiScaleBox.Text = tostring(uiScaleValue) end
    if dropModeBtnRef then dropModeBtnRef.Text = dropMode == 1 and "Fling" or "Jump Drop" end
    if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
    local function safeSet(fn, val) if fn then fn(val) end end
    safeSet(autoBatSetVisual, false)
    safeSet(autoLeftSetVisual, false)
    safeSet(autoRightSetVisual, false)
    safeSet(setBatCounterVisual, false)
    safeSet(setBatCounterV2Visual, false)
    safeSet(setUnwalkVisual, false)
    safeSet(setAntiLagVisual, false)
    safeSet(setLockUIVisual, false)
    safeSet(setInstaGrab, false)
    safeSet(setESPVIsual, false)
    safeSet(setESPLineVisual, false)
    safeSet(setVividVisual, false)
    safeSet(setHideButtonsVisual, false)
    safeSet(bodyLockSetVisual, false)
    safeSet(setAntiDieVisual, false)
    safeSet(setAntiBatVisual, false)
    safeSet(setAntiFlingVisual, false)
    safeSet(setAntiRagVisual, false)
    safeSet(infJumpSetVisual, false)
    if autoStealModeBtn then autoStealModeBtn.Text = autoStealMode end
    if kiraUI.motorLabel then kiraUI.motorLabel.Text = "Direct" end
    if kiraUI.carrySetter then kiraUI.carrySetter(false) end
    if kiraUI.effectsSetter then kiraUI.effectsSetter(kiraUI.effects) end
    if kiraUI.lockButtonsSetter then kiraUI.lockButtonsSetter(false) end
    applyBackgroundMode(backgroundMode)
    if _G.__RivalHubRefreshSpeedEngine then pcall(_G.__RivalHubRefreshSpeedEngine) end
    safeSet(mobSetBatV2, false)
    if _G.__RivalHubStopBatV2 then pcall(_G.__RivalHubStopBatV2) end
    disableVividGraphics()
    applyHideButtons(false)
    if _G.stretchToggleSetter then _G.stretchToggleSetter(false) end
    safeSet(mobSetAutoBat, false)
    safeSet(mobSetAutoLeft, false)
    safeSet(mobSetAutoRight, false)
    safeSet(mobSetDropBR, false)
    safeSet(mobSetTpDown, false)
    safeSet(mobSetCarry, false)
    safeSet(mobSetLagger1, false)
    safeSet(mobSetLagger2, false)
    refreshSpeedModeLabel()
    updateProgressBarVisibility()
    disableAntiLag()
    if stopAntiBat then stopAntiBat() end
    if stopAntiFling then stopAntiFling() end
    if stopESPLine then stopESPLine() end
    skyTheme = "Off"
    pcall(applyCustomSky, "Off")
    if skySelectorLabel then skySelectorLabel.Text = "Off" end
    if antiDieEnabled then
        AntiDieModule.stop()
        antiDieEnabled = false
    end
    for _, ref in ipairs(keyButtonRefs) do
        local entry = ref.entry
        local label = (entry.gp and entry.gp.Name) or (entry.kb and entry.kb.Name) or "None"
        ref.btn.Text = label
    end
    currentColorTheme = "Orange"
    selectedColor = COLOR_THEMES["Orange"]
    updateAllUIThemeColors(selectedColor)
    if miniBtn then
        local stroke = miniBtn:FindFirstChildOfClass("UIStroke")
        if stroke then stroke.Color = Color3.fromRGB(42, 43, 47) end
    end
    if MobilePanel then
        for _, btn in ipairs(MobilePanel:GetChildren()) do
            if btn:IsA("TextButton") then
                local label = btn:FindFirstChildOfClass("TextLabel")
                if label then
                    local isActive = btn.BackgroundColor3 == selectedColor
                    if not isActive then label.TextColor3 = selectedColor end
                end
            end
        end
    end
    saveAllSettings()
end

function resetFloatingPositions()
    if MobilePanel then
        savedButtonPositions = {}
        for _, btn in ipairs(MobilePanel:GetChildren()) do
            if btn:IsA("TextButton") and btn.Name then
                local defX, defY = getDefaultButtonPosition(btn.Name)
                btn.Position = UDim2.new(0, defX, 0, defY)
            end
        end
    end
    if pbFrame then
        pbFrame.Position = UDim2.new(1, -282, 0, 18)
        savedProgressBarPos = nil
    end
    savedMobilePanelPos = nil
    if kiraUI.fit then kiraUI.fit() end
end

function resetToFactoryDefaults()
    _isResetting = true
    local ok, err = pcall(function()
        stopAutoSteal()
        stopBatCounter()
        if stopBatCounterV2 then stopBatCounterV2() end
            if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
        if antiDieEnabled then AntiDieModule.stop() end
        if antiBatEnabled and stopAntiBat then stopAntiBat() end
        if antiFlingEnabled and stopAntiFling then stopAntiFling() end
        if _G.__RivalHubStopBatV2 then pcall(_G.__RivalHubStopBatV2) end
        stopUnwalk()
        InfiniteJump.stop()
        InfiniteJump.setMode("hold")
        disableAutoBat()
        stopBodyLock()
        if espEnabled then toggleESP(false) end
        if espLineEnabled and stopESPLine then stopESPLine() end
        if stretchEnabled then disableStretch() end
        if antiLagEnabled then disableAntiLag() end
        if vividGraphicsEnabled then disableVividGraphics() end
        if hideButtonsEnabled then applyHideButtons(false) end
        if dropActive then stopDropBrainrot() end
        skyTheme = "Off"
        pcall(applyCustomSky, "Off")
        if skySelectorLabel then skySelectorLabel.Text = "Off" end
        if antiDieEnabled then AntiDieModule.stop(); antiDieEnabled = false end
        NS = 60; CS = 29; LAGGER_SPEED = 15; LAGGER_CARRY_SPEED = 24.5
        CONFIG.STEAL_RANGE = 61
        CONFIG.HOLD_MIN = 0.05
        CONFIG.HOLD_MAX = 0.15
        CONFIG.ENTRY_DELAY = 0.1
        CONFIG.COOLDOWN = 0.2
        CONFIG.PRIME_RANGE = 60
        speedMode = false; laggerToggled = false; laggerCarryToggled = false
        antiRagdollMode = "off"; antiDieEnabled = false
        antiBatEnabled = false
        antiFlingEnabled = false
        batCounterEnabled = false; batCounterV2Enabled = false
        autoBatEnabled = false; autoLeftEnabled = false; autoRightEnabled = false
        unwalkEnabled = false; antiLagEnabled = false
        uiLocked = false; mobileButtonsLocked = false
        CONFIG.AUTO_STEAL_ENABLED = false
        setAutoStealMode("75")
        useVenixCarryEngine = false
        kiraVenixCarry:stop()
        kiraVenixCarry.softStealEnabled = false
        kiraVenixCarry.softStealSpeed = 30
        kiraVenixCarry.softStealRadius = 10
        kiraUI.effects = true
        if _G._rivalHubStealModeSetter then pcall(_G._rivalHubStealModeSetter, "V1") end
        BAT_AIMBOT_SPEED = 55
        BYPASS_AIMBOT_SPEED = BAT_AIMBOT_SPEED
        if State then State.bypassBatSpeed = BYPASS_AIMBOT_SPEED end
        dropMode = 1; stretchEnabled = false; stretchFOV = 120
        uiScaleValue = 90
        if kiraUI.fit then kiraUI.fit() end
        if pbScale then pbScale.Scale = 1 end
        espEnabled = false; espLineEnabled = false; vividGraphicsEnabled = false; hideButtonsEnabled = false
        bodyLockEnabled = false; bodyLockRange = 20
        backgroundIndex = 1; backgroundImageTransparency = 0; backgroundMode = "Background 1"
        floatingButtonScale = 1
        currentAnimPack = "Off"; stopAnimPack()
        currentOutfitIndex = 1
        pcall(VX7A.stop)
        currentColorTheme = "Orange"; selectedColor = COLOR_THEMES["Orange"]
        currentRivalTitleTheme = "Orange"
        applyRivalTitleTheme("Orange")
        if main then main.Position = UDim2.new(0, 20, 0, 2); _G.__RivalHubMainOriginalPos = main.Position end
        for key, val in pairs(DEFAULT_KB) do
            if KB[key] then KB[key].kb = val.kb; KB[key].gp = val.gp end
        end
        resetFloatingPositions()
        forceResetUI()
        updateProgressBarVisibility()
        refreshSpeedModeLabel()
        _lastSavedJSON = nil
    end)
    _isResetting = false
    if not ok then warn("[resetToFactoryDefaults]", err) end
    if ok then saveAllSettings(true) end
    return ok
end

function updateProgressBarVisibility()
    if pbFrame then pbFrame.Visible = true end
end

function applyShimmerToText(obj, speed, baseColor)
    speed = speed or 0.8
    local color = baseColor or Color3.fromRGB(215, 215, 225)
    local grad = Instance.new("UIGradient", obj)
    grad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, color),
        ColorSequenceKeypoint.new(0.3, color:Lerp(Color3.new(0.15,0.15,0.15), 0.25)),
        ColorSequenceKeypoint.new(0.5, color:Lerp(Color3.new(1,1,1), 0.55)),
        ColorSequenceKeypoint.new(0.7, color:Lerp(Color3.new(0.15,0.15,0.15), 0.25)),
        ColorSequenceKeypoint.new(1, color),
    })
    grad.Rotation = 45
    grad.Offset = Vector2.new(0,0)
    task.spawn(function()
        local t = 0
        while grad and grad.Parent do
            t = t + 0.02
            grad.Offset = Vector2.new(math.sin(t * speed) * 0.4, 0)
            grad.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.00, color),
                ColorSequenceKeypoint.new(0.30, color:Lerp(Color3.new(0.15,0.15,0.15), 0.25)),
                ColorSequenceKeypoint.new(0.50, color:Lerp(Color3.new(1,1,1), 0.55)),
                ColorSequenceKeypoint.new(0.70, color:Lerp(Color3.new(0.15,0.15,0.15), 0.25)),
                ColorSequenceKeypoint.new(1.00, color),
            })
            task.wait(0.04)
        end
    end)
    return grad
end

function getDefaultButtonPosition(name)
    local map = {DropBR = 0, AutoLeft = 1, AutoBat = 2, AutoRight = 3, TpDown = 4, Carry = 5, Lagger1 = 6, Lagger2 = 7, BatV2 = 8}
    local index = map[name] or 0
    local cam = Workspace.CurrentCamera
    local size = cam and cam.ViewportSize or Vector2.new(844, 390)
    local spacing = 64 * floatingButtonScale + 8
    local startX = math.max(8, size.X - spacing * 3 - 18)
    local startY = math.max(110, size.Y - spacing * 3 - 96)
    return startX + index % 3 * spacing, startY + _floor(index / 3) * spacing
end

local function tpBatDisconnectAntiDie()
    for _, conn in ipairs(TPBatState.antiDieConnections) do
        pcall(function() conn:Disconnect() end)
    end
    TPBatState.antiDieConnections = {}
end

local function tpBatHookAntiDie(character)
    tpBatDisconnectAntiDie()
    if not TPBatState.enabled or not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    pcall(function()
        humanoid.BreakJointsOnDeath = false
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dying, false)
    end)
    local _antiDieCooldown = false
    table.insert(TPBatState.antiDieConnections,
        humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if _antiDieCooldown then return end
            if TPBatState.enabled and humanoid.Parent and humanoid.Health <= 0 then
                _antiDieCooldown = true
                task.defer(function()
                    pcall(function()
                        if humanoid and humanoid.Parent then
                            humanoid.Health = humanoid.MaxHealth
                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                        end
                    end)
                    task.wait(0.5)
                    _antiDieCooldown = false
                end)
            end
        end)
    )
end

local function tpBatFindBat()
    local character = LP.Character
    if not character then return nil end
    local function isBat(tool)
        if not tool:IsA("Tool") then return false end
        local name = tool.Name:lower()
        return name:find("bat") ~= nil or name:find("slap") ~= nil
    end
    for _, child in ipairs(character:GetChildren()) do
        if isBat(child) then return child end
    end
    local backpack = LP:FindFirstChildOfClass("Backpack")
    if not backpack then return nil end
    for _, child in ipairs(backpack:GetChildren()) do
        if isBat(child) then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                pcall(function() humanoid:EquipTool(child) end)
            end
            return child
        end
    end
    return nil
end

local function tpBatSwing()
    if TPBatState.hitCooldown then return end
    TPBatState.hitCooldown = true
    pcall(function()
        local bat = tpBatFindBat()
        if not bat then return end
        bat:Activate()
        local ev = bat:FindFirstChildWhichIsA("RemoteEvent", true)
        if ev then pcall(function() ev:FireServer() end) end
    end)
    task.delay(0.05, function()
        TPBatState.hitCooldown = false
    end)
end

local function tpBatTick()
    if not TPBatState.enabled then return end
    local character = LP.Character
    if not character then return end
    local root = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not root or not humanoid or humanoid.Health <= 0 then return end

    local closestPlayer, closestDistance = nil, math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP and player.Character then
            local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local targetHumanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if targetRoot and targetHumanoid and targetHumanoid.Health > 0 then
                local distance = (root.Position - targetRoot.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end

    if not closestPlayer or closestDistance > 100 then return end
    local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end

    -- TP suave con Lerp + zerear velocidad en cada frame para no salir volando
    local targetPos = targetRoot.Position + Vector3.new(0, 0.9, 0)
    if closestDistance > 5 then
        root.CFrame = CFrame.new(targetPos, targetRoot.Position)
    elseif closestDistance > 2 then
        root.CFrame = CFrame.new(root.Position:Lerp(targetPos, 0.35), targetRoot.Position)
    end

    pcall(function()
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
    end)

    local cam = workspace.CurrentCamera
    if cam then
        cam.CFrame = CFrame.new(cam.CFrame.Position, targetRoot.Position + Vector3.new(0, 0.5, 0))
    end

    tpBatSwing()
end

function tpBatSetEnabled(value)
    TPBatState.enabled = value == true
    if TPBatState.enabled then
        tpBatHookAntiDie(LP.Character)
        if TPBatState.connection then
            TPBatState.connection:Disconnect()
            TPBatState.connection = nil
        end
        TPBatState.connection = RunService.Heartbeat:Connect(tpBatTick)
    else
        if TPBatState.connection then
            TPBatState.connection:Disconnect()
            TPBatState.connection = nil
        end
        tpBatDisconnectAntiDie()
        local character = LP.Character
        local root = character and character:FindFirstChild("HumanoidRootPart")
        if root then
            pcall(function()
                if root.SetNetworkOwner then root:SetNetworkOwner(LP) end
            end)
        end
    end
end

LP.CharacterAdded:Connect(function(character)
    if TPBatState.enabled then
        task.wait(0.1)
        tpBatHookAntiDie(character)
    end
end)


function buildGui()
    local ROW_BG = Color3.fromRGB(20, 23, 28)
    local ROW_BORDER = Color3.fromRGB(87, 73, 59)
    local WHITE = Color3.fromRGB(249, 242, 229)
    local INP = Color3.fromRGB(12, 15, 19)
    local GUI_W, GUI_H = 570, 570
    for _, parent in ipairs({CoreGui, pgui}) do
        for _, name in ipairs({"RivalHub", "RivalHubMobilePanel", "RivalHubSpeedIndicator", "RivalHubBootErrors"}) do
            pcall(function() local old = parent:FindFirstChild(name); if old then old:Destroy() end end)
        end
    end
    gui = kiraUI.new("ScreenGui", nil, {Name = "RivalHub", ResetOnSpawn = false,
        IgnoreGuiInset = true, DisplayOrder = 30, ZIndexBehavior = Enum.ZIndexBehavior.Sibling})
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
    local ok = pcall(function() gui.Parent = CoreGui end)
    if not ok then gui.Parent = pgui end
    main = kiraUI.new("Frame", gui, {Name = "Main", Size = UDim2.fromOffset(GUI_W, GUI_H),
        Position = UDim2.fromOffset(18, 16), BackgroundColor3 = Color3.fromRGB(9, 12, 16),
        BorderSizePixel = 0, ClipsDescendants = true, Active = true})
    kiraUI.round(main, 18)
    local border = kiraUI.stroke(main, Color3.fromRGB(220, 35, 42), 0.25, 1.2)
    border.Name = "MainBorder"
    mainUIScale = kiraUI.new("UIScale", main, {Scale = uiScaleValue / 100})
    local bg = kiraUI.new("ImageLabel", main, {Name = "BackgroundImage", Size = UDim2.fromScale(1, 1),
        Image = getBackgroundAsset(backgroundIndex), ImageTransparency = (backgroundMode == "None" and 1 or backgroundImageTransparency), Visible = (backgroundMode ~= "None"),
        BackgroundTransparency = 1, ScaleType = Enum.ScaleType.Crop, ZIndex = 1})
    kiraUI.round(bg, 18)
    local veil = kiraUI.new("Frame", main, {Name = "BackgroundVeil", Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = Color3.fromRGB(24, 2, 5), BackgroundTransparency = 0.34, BorderSizePixel = 0, ZIndex = 2})
    kiraUI.new("UIGradient", veil, {Rotation = 35, Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.04), NumberSequenceKeypoint.new(0.6, 0.22), NumberSequenceKeypoint.new(1, 0.10)})})
    kiraUI.round(veil, 18)
    local innerBorder = kiraUI.new("Frame", main, {Name = "PanelInset", Position = UDim2.fromOffset(3, 3),
        Size = UDim2.new(1, -6, 1, -6), BackgroundTransparency = 1, BorderSizePixel = 0, ZIndex = 3})
    kiraUI.round(innerBorder, 15)
    kiraUI.stroke(innerBorder, Color3.fromRGB(239, 206, 157), 0.93, 1)
    local topLight = kiraUI.new("Frame", main, {Name = "TopLight", Position = UDim2.fromOffset(28, 1),
        Size = UDim2.new(1, -56, 0, 1), BackgroundColor3 = Color3.fromRGB(255, 202, 126), BorderSizePixel = 0, ZIndex = 4})
    kiraUI.new("UIGradient", topLight, {Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.5, 0.25), NumberSequenceKeypoint.new(1, 1)})})
    local header = kiraUI.new("Frame", main, {Name = "ScriptHeader", Position = UDim2.fromOffset(14, 6),
        Size = UDim2.new(1, -28, 0, 72), BackgroundColor3 = Color3.fromRGB(18, 20, 25),
        BorderSizePixel = 0, ClipsDescendants = true, ZIndex = 5})
    kiraUI.round(header, 15)
    kiraUI.stroke(header, Color3.fromRGB(180, 28, 35), 0.45, 1)
    kiraUI.art = kiraUI.new("ImageLabel", header, {Name = "Scene", Size = UDim2.fromScale(1, 1),
        Image = bg.Image, ImageTransparency = 0.22, BackgroundTransparency = 1, ScaleType = Enum.ScaleType.Crop, ZIndex = 5})
    local shade = kiraUI.new("Frame", header, {Name = "ArtworkShade", Size = UDim2.fromScale(1, 1), BorderSizePixel = 0,
        BackgroundColor3 = Color3.fromRGB(6, 10, 15), BackgroundTransparency = 0.1, ZIndex = 6})
    kiraUI.new("UIGradient", shade, {Rotation = 0, Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.02), NumberSequenceKeypoint.new(0.52, 0.2), NumberSequenceKeypoint.new(1, 0.7)})})
    local badge = kiraUI.new("Frame", header, {Name = "Edition", Position = UDim2.fromOffset(13, 5),
        Size = UDim2.fromOffset(97, 17), BackgroundColor3 = Color3.fromRGB(60, 39, 24),
        BackgroundTransparency = 0.12, BorderSizePixel = 0, ZIndex = 7})
    kiraUI.round(badge, 5)
    local edition = kiraUI.text(badge, "HALLOWEEN '26", UDim2.fromOffset(7, 0), UDim2.new(1, -14, 1, 0), 8,
        Color3.fromRGB(253, 194, 117), 8)
    edition.Font = Enum.Font.GothamBold
    scriptLogoRef = kiraUI.text(header, "kira", UDim2.fromOffset(11, 17), UDim2.fromOffset(155, 39), 36, WHITE, 8)
    scriptLogoRef.Font = Enum.Font.GothamBlack
    kiraUI.titleGradient = kiraUI.new("UIGradient", scriptLogoRef, {Rotation = 90,
        Color = ColorSequence.new(WHITE, Color3.fromRGB(247, 192, 120))})
    kiraUI.text(header, "N I G H T F A L L", UDim2.fromOffset(14, 56), UDim2.fromOffset(162, 14), 8,
        Color3.fromRGB(205, 189, 164), 8)
    local glow = kiraUI.new("Frame", header, {Name = "PlayerAvatarGlow", AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(1, -56, 0.54, 0), Size = UDim2.fromOffset(64, 64),
        BackgroundColor3 = getThemeColor(), BackgroundTransparency = 0.91, BorderSizePixel = 0, ZIndex = 7})
    kiraUI.round(glow, 50)
    local ring = kiraUI.new("Frame", header, {Name = "PlayerAvatarRing", AnchorPoint = Vector2.new(0.5, 0.5),
        Position = glow.Position, Size = UDim2.fromOffset(58, 58), BackgroundTransparency = 1, ZIndex = 7})
    kiraUI.round(ring, 50)
    kiraUI.stroke(ring, Color3.fromRGB(255, 202, 127), 0.63, 1)
    local playerAvatar = kiraUI.pumpkin(header, UDim2.new(1, -82, 0.54, -26), 52, 8)
    for i = 1, 4 do
        local ember = kiraUI.new("Frame", header, {Name = "Ember", Size = UDim2.fromOffset(2, 2),
            Position = UDim2.new(0.58 + i * 0.07, 0, 0.14 + (i % 3) * 0.25, 0),
            BackgroundColor3 = Color3.fromRGB(255, 199, 120), BackgroundTransparency = 0.5,
            BorderSizePixel = 0, ZIndex = 8})
        table.insert(kiraUI.animated, {object = ember, position = ember.Position, x = 2, y = 4})
    end

    local close = kiraUI.new("TextButton", main, {Name = "Close", Size = UDim2.fromOffset(26, 26),
        Position = UDim2.new(1, -39, 0, 15), Text = "−", TextSize = 20, Font = Enum.Font.GothamMedium,
        TextColor3 = WHITE, BackgroundColor3 = INP, BackgroundTransparency = 0.06,
        BorderSizePixel = 0, AutoButtonColor = false, ZIndex = 18})
    kiraUI.round(close, 8)
    kiraUI.stroke(close, ROW_BORDER, 0.6, 1)
    miniBtn = kiraUI.new("TextButton", gui, {Name = "MinimizedFrame", Size = UDim2.fromOffset(146, 46),
        Position = UDim2.fromOffset(16, 58), Text = "", BackgroundColor3 = Color3.fromRGB(10, 14, 19),
        BorderSizePixel = 0, AutoButtonColor = false, Visible = false, ZIndex = 25})
    kiraUI.round(miniBtn, 11)
    kiraUI.stroke(miniBtn, getThemeColor(), 0.35, 1.2)
    kiraUI.new("UIGradient", miniBtn, {Rotation = 0, Color = ColorSequence.new(Color3.fromRGB(34, 28, 24), Color3.fromRGB(11, 15, 20))})
    kiraUI.pumpkin(miniBtn, UDim2.fromOffset(9, 7), 32, 26)
    local miniTitle = kiraUI.text(miniBtn, "kira", UDim2.fromOffset(50, 7), UDim2.fromOffset(62, 18), 15, WHITE, 28)
    miniTitle.Font = Enum.Font.GothamBlack
    kiraUI.text(miniBtn, "NIGHTFALL", UDim2.fromOffset(50, 26), UDim2.fromOffset(68, 12), 7, getThemeColor(), 28)
    kiraUI.text(miniBtn, "›", UDim2.new(1, -22, 0, 8), UDim2.fromOffset(17, 30), 22, WHITE, 28)
    miniBackgroundImage = nil
    local animating, animationId, tween = false, 0, nil
    local function animate(open)
        animationId = animationId + 1
        local thisId = animationId
        if tween then tween:Cancel() end
        animating = true
        main.Visible = true
        miniBtn.Visible = false
        local width, height = kiraUI.width or GUI_W, kiraUI.height or GUI_H
        if open then main.Size = UDim2.fromOffset(width - 10, height - 8) end
        tween = TS:Create(main, TweenInfo.new(kiraUI.effects and 0.23 or 0.01, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
            {Size = UDim2.fromOffset(open and width or width - 10, open and height or height - 8)})
        tween.Completed:Connect(function()
            if thisId ~= animationId or not main.Parent then return end
            animating = false
            main.Visible = open
            miniBtn.Visible = not open
            main.Size = UDim2.fromOffset(width, height)
        end)
        tween:Play()
    end
    showGui = function() animate(true) end
    hideGui = function() animate(false) end
    close.Activated:Connect(hideGui)
    miniBtn.Activated:Connect(showGui)
    _G.__RivalHubMainOriginalPos = main.Position
    main:GetPropertyChangedSignal("Position"):Connect(function()
        if not animating then _G.__RivalHubMainOriginalPos = main.Position end
    end)
    drag(main, 80)
    local tabBar = kiraUI.new("Frame", main, {Name = "Navigation", Size = UDim2.new(0, 64, 0, 282),
        Position = UDim2.new(0, 14, 0, 166), BackgroundColor3 = Color3.fromRGB(10, 14, 19),
        BackgroundTransparency = 0.06, BorderSizePixel = 0, ZIndex = 14})
    kiraUI.round(tabBar, 16)
    kiraUI.stroke(tabBar, Color3.fromRGB(127, 97, 62), 0.62, 1)
    kiraUI.new("UIListLayout", tabBar, {FillDirection = Enum.FillDirection.Vertical,
        HorizontalAlignment = Enum.HorizontalAlignment.Center, VerticalAlignment = Enum.VerticalAlignment.Center,
        Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder})
    local headingPanel = kiraUI.new("Frame", main, {Name = "SectionPanel", Position = UDim2.fromOffset(88, 85),
        Size = UDim2.new(1, -166, 0, 27), BackgroundColor3 = Color3.fromRGB(21, 24, 29),
        BackgroundTransparency = 0.12, BorderSizePixel = 0, ZIndex = 6})
    kiraUI.round(headingPanel, 8)
    kiraUI.stroke(headingPanel, Color3.fromRGB(128, 103, 72), 0.82, 1)
    local pageHeading = kiraUI.text(main, "01   /   MOVEMENT", UDim2.fromOffset(99, 87), UDim2.new(1, -186, 0, 23), 11, WHITE, 8)
    pageHeading.Name = "SectionHeading"
    pageHeading.Font = Enum.Font.GothamBold
    local headingRule = kiraUI.new("Frame", main, {Position = UDim2.fromOffset(20, 114),
        Position = UDim2.new(0, 88, 0, 112), Size = UDim2.new(1, -166, 0, 1), BackgroundColor3 = ROW_BORDER, BackgroundTransparency = 0.48, BorderSizePixel = 0, ZIndex = 7})
    local tabContent = kiraUI.new("Frame", main, {Name = "Content", Size = UDim2.new(1, -98, 1, -230),
        Position = UDim2.fromOffset(82, 120), BackgroundTransparency = 1, ClipsDescendants = true, ZIndex = 6})
    local footer = kiraUI.new("Frame", main, {Name = "Footer", Position = UDim2.new(0, 88, 1, -34),
        Size = UDim2.new(1, -166, 0, 25), BackgroundTransparency = 1, ZIndex = 8})
    kiraUI.new("Frame", footer, {Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = ROW_BORDER,
        BackgroundTransparency = 0.62, BorderSizePixel = 0, ZIndex = 8})
    local avatar = kiraUI.new("ImageLabel", footer, {Position = UDim2.fromOffset(2, 6), Size = UDim2.fromOffset(17, 17),
        Image = "rbxthumb://type=AvatarHeadShot&id=" .. LP.UserId .. "&w=150&h=150", BackgroundColor3 = INP, BorderSizePixel = 0, ZIndex = 8})
    kiraUI.round(avatar, 5)
    kiraUI.text(footer, "@" .. LP.Name, UDim2.fromOffset(27, 5), UDim2.new(0.5, -32, 0, 18), 9, Color3.fromRGB(179, 177, 169), 9)
    kiraUI.footerStats = kiraUI.text(footer, "kira  /  NIGHTFALL", UDim2.new(0.5, 0, 0, 5), UDim2.new(0.5, -3, 0, 18), 8,
        Color3.fromRGB(211, 167, 108), 9)
    kiraUI.footerStats.TextXAlignment = Enum.TextXAlignment.Right
    local tabs = {"Speed", "Custom", "Visual", "Settings", "Keybinds"}
    local titles = {"MOVE", "COMBAT", "LOOK", "STEAL", "BINDS"}
    local headings = {"MOVEMENT", "COMBAT", "APPEARANCE", "AUTO STEAL", "KEYBINDS"}
    tabButtons = {}
    local contentPages = {}
    for i, name in ipairs(tabs) do
        local button = kiraUI.new("TextButton", tabBar, {Name = name, Text = "", Size = UDim2.fromOffset(56, 48),
            BackgroundColor3 = Color3.fromRGB(25, 25, 24), BackgroundTransparency = 1, BorderSizePixel = 0,
            Font = Enum.Font.GothamBold, TextSize = 9, TextColor3 = WHITE,
            AutoButtonColor = false, ZIndex = 15, LayoutOrder = i})
        kiraUI.round(button, 8)
        local stroke = kiraUI.stroke(button, getThemeColor(), 1, 1)
        stroke.Name = "TabStroke"
        kiraUI.new("UIGradient", button, {Name = "TabBgGrad", Enabled = false,
            Color = ColorSequence.new(Color3.fromRGB(55, 40, 25), Color3.fromRGB(24, 24, 25)), Rotation = 90})
        kiraUI.sectionIcon(button, name, UDim2.fromOffset(18, 6), 20, getThemeColor(), 16)
        local caption = kiraUI.text(button, titles[i], UDim2.fromOffset(0, 30), UDim2.fromOffset(56, 11), 7, WHITE, 16)
        caption.Name = "Caption"
        caption.TextXAlignment = Enum.TextXAlignment.Center
        local underline = kiraUI.new("Frame", button, {Name = "Underline", Size = UDim2.fromOffset(18, 2),
            Position = UDim2.fromOffset(19, 44), BackgroundColor3 = getThemeColor(), BorderSizePixel = 0, Visible = i == 1, ZIndex = 16})
        kiraUI.round(underline, 2)
        local page = kiraUI.new("ScrollingFrame", tabContent, {Name = name, Size = UDim2.fromScale(1, 1),
            BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 2, ScrollBarImageColor3 = getThemeColor(),
            ScrollBarImageTransparency = 0.36, AutomaticCanvasSize = Enum.AutomaticSize.Y, CanvasSize = UDim2.new(),
            ScrollingDirection = Enum.ScrollingDirection.Y, ClipsDescendants = true, Visible = i == 1, ZIndex = 6})
        kiraUI.new("UIListLayout", page, {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder,
            HorizontalAlignment = Enum.HorizontalAlignment.Center})
        kiraUI.new("UIPadding", page, {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8),
            PaddingTop = UDim.new(0, 3), PaddingBottom = UDim.new(0, 18)})
        contentPages[name] = page
        table.insert(tabButtons, button)
        local function select()
            pageHeading.Text = string.format("%02d   /   %s", i, headings[i])
            for _, p in pairs(contentPages) do p.Visible = false end
            page.Visible = true
            page.Position = UDim2.fromOffset(kiraUI.effects and 8 or 0, 0)
            TS:Create(page, TweenInfo.new(0.18), {Position = UDim2.fromOffset(0, 0)}):Play()
            for _, b in ipairs(tabButtons) do
                local on = b == button
                b.Underline.Visible = on
                b.TabBgGrad.Enabled = false
                b.TabStroke.Transparency = on and 0.42 or 1
                b.Caption.TextColor3 = on and WHITE or Color3.fromRGB(155, 157, 156)
                kiraUI.colorSectionIcon(b:FindFirstChild("SectionIcon"), on and getThemeColor() or Color3.fromRGB(144, 147, 150))
                TS:Create(b, TweenInfo.new(0.16), {BackgroundTransparency = on and 0.05 or 1,
                    BackgroundColor3 = on and Color3.fromRGB(53, 37, 24) or Color3.fromRGB(15, 18, 23)}):Play()
            end
        end
        button.Activated:Connect(select)
        if i == 1 then select() end
    end
    local pageCounters = {}
    local function getNextOrder(page)
        pageCounters[page] = (pageCounters[page] or 0) + 1
        return pageCounters[page]
    end
    local function mkSect(page, text)
        local row = kiraUI.new("Frame", page, {Name = "Section", Size = UDim2.new(1, -4, 0, 24), BackgroundTransparency = 1,
            LayoutOrder = getNextOrder(page), ZIndex = 7})
        local line = kiraUI.new("Frame", row, {Position = UDim2.fromOffset(1, 12), Size = UDim2.fromOffset(10, 2),
            BackgroundColor3 = getThemeColor(), BorderSizePixel = 0, ZIndex = 8})
        local label = kiraUI.text(row, text:upper(), UDim2.fromOffset(21, 0), UDim2.new(1, -24, 1, 0),
            9, Color3.fromRGB(203, 173, 127), 8)
        label.Font = Enum.Font.GothamBold
        return row
    end
    local function mkRow(page, height)
        local row = kiraUI.new("Frame", page, {Name = "Control", Size = UDim2.new(1, -4, 0, math.max(height or 40, 40)),
            BackgroundColor3 = ROW_BG, BackgroundTransparency = 0.06, BorderSizePixel = 0,
            LayoutOrder = getNextOrder(page), ZIndex = 7})
        kiraUI.round(row, 10)
        kiraUI.new("UIGradient", row, {Rotation = 0, Color = ColorSequence.new(Color3.fromRGB(30, 28, 27), Color3.fromRGB(16, 21, 28))})
        local stroke = kiraUI.stroke(row, ROW_BORDER, 0.73, 1)
        row.MouseEnter:Connect(function() TS:Create(stroke, TweenInfo.new(0.12), {Transparency = 0.34}):Play() end)
        row.MouseLeave:Connect(function() TS:Create(stroke, TweenInfo.new(0.12), {Transparency = 0.73}):Play() end)
        return row
    end
    local function mkLabel(row, text)
        return kiraUI.text(row, text, UDim2.fromOffset(14, 0), UDim2.new(0.48, -8, 1, 0), 12, WHITE, 8)
    end
    local function mkPill(row)
        local pill = kiraUI.new("Frame", row, {Name = "Track", Size = UDim2.fromOffset(56, 27),
            AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -13, 0.5, 0),
            BackgroundColor3 = Color3.fromRGB(11, 15, 20), BorderSizePixel = 0, ZIndex = 8})
        kiraUI.round(pill, 12)
        kiraUI.stroke(pill, ROW_BORDER, 0.57, 1)
        local state = kiraUI.text(pill, "OFF", UDim2.fromOffset(19, 0), UDim2.fromOffset(34, 27), 8,
            Color3.fromRGB(140, 147, 156), 9)
        state.Name = "State"
        state.TextXAlignment = Enum.TextXAlignment.Center
        local dot = kiraUI.new("Frame", pill, {Name = "Knob", AnchorPoint = Vector2.new(0.5, 0.5),
            Size = UDim2.fromOffset(8, 8), Position = UDim2.new(0, 12, 0.5, 0),
            BackgroundColor3 = Color3.fromRGB(74, 83, 94), BorderSizePixel = 0, ZIndex = 9})
        kiraUI.round(dot, 3)
        return pill, dot
    end
    local function animPill(pill, dot, on)
        TS:Create(pill, TweenInfo.new(0.16), {BackgroundColor3 = on and Color3.fromRGB(57, 38, 24) or Color3.fromRGB(11, 15, 20)}):Play()
        TS:Create(dot, TweenInfo.new(0.16), {BackgroundColor3 = on and getThemeColor() or Color3.fromRGB(74, 83, 94)}):Play()
        local state = pill:FindFirstChild("State")
        if state then state.Text = on and "ON" or "OFF"; state.TextColor3 = on and WHITE or Color3.fromRGB(140, 147, 156) end
        local stroke = pill:FindFirstChildOfClass("UIStroke")
        if stroke then stroke.Color = on and getThemeColor() or ROW_BORDER; stroke.Transparency = on and 0.28 or 0.57 end
    end
    local function mkToggle(page, text, callback)
        local row = mkRow(page, 46)
        local label = mkLabel(row, text)
        label.Size = UDim2.new(1, -86, 1, 0)
        local pill, dot = mkPill(row)
        local on = false
        local function set(value)
            on = value == true
            row:SetAttribute("kiraActive", on)
            animPill(pill, dot, on)
            row:FindFirstChildOfClass("UIStroke").Transparency = on and 0.4 or 0.73
        end
        local hit = kiraUI.new("TextButton", row, {Name = "ToggleHit", Size = UDim2.fromScale(1, 1), Text = "",
            BackgroundTransparency = 1, AutoButtonColor = false, ZIndex = 10})
        hit.Activated:Connect(function()
            local previous = on
            set(not on)
            local ok, err = pcall(callback, on)
            if not ok then set(previous); warn("[kira] " .. text .. ": " .. tostring(err)) end
            task.defer(function() pcall(saveAllSettings) end)
        end)
        return set
    end
    local function mkSelector(row, default, options, callback)
        local frame = kiraUI.new("Frame", row, {Name = "Selector", Size = UDim2.fromOffset(146, 32),
            Position = UDim2.new(1, -156, 0.5, -16), BackgroundColor3 = INP, BorderSizePixel = 0, ZIndex = 8})
        kiraUI.round(frame, 6)
        kiraUI.stroke(frame, ROW_BORDER, 0.68, 1)
        local label = kiraUI.text(frame, default, UDim2.fromOffset(30, 0), UDim2.fromOffset(86, 32), 10, WHITE, 9)
        label.Font = Enum.Font.GothamBold
        label.TextXAlignment = Enum.TextXAlignment.Center
        for i, direction in ipairs({-1, 1}) do
            local button = kiraUI.new("TextButton", frame, {Size = UDim2.fromOffset(29, 32),
                Position = UDim2.fromOffset(i == 1 and 0 or 117, 0), BackgroundColor3 = INP,
                BorderSizePixel = 0, Text = direction == -1 and "‹" or "›", TextSize = 21,
                Font = Enum.Font.GothamMedium, TextColor3 = getThemeColor(), AutoButtonColor = false, ZIndex = 9})
            kiraUI.round(button, 6)
            button.Activated:Connect(function()
                callback(direction, function(text) label.Text = text end)
                task.defer(function() pcall(saveAllSettings) end)
            end)
        end
        return label
    end
    local function mkBox(row, default, width, xoff, callback)
        local w = math.max(width or 56, 64)
        local box = kiraUI.new("TextBox", row, {Size = UDim2.fromOffset(w, 32),
            Position = UDim2.new(1, -math.max(xoff or 76, w + 12), 0.5, -16), Text = tostring(default),
            BackgroundColor3 = INP, BorderSizePixel = 0, Font = Enum.Font.GothamBold, TextSize = 13,
            TextColor3 = Color3.fromRGB(250, 202, 129), ClearTextOnFocus = false, ZIndex = 9})
        kiraUI.round(box, 9)
        local border = kiraUI.stroke(box, ROW_BORDER, 0.58, 1)
        local lastValid = tostring(default)
        box.Focused:Connect(function() border.Color = getThemeColor(); border.Transparency = 0.10 end)
        box.FocusLost:Connect(function()
            border.Color = ROW_BORDER; border.Transparency = 0.58
            local value = tonumber(box.Text)
            if value and value == value and math.abs(value) < math.huge then
                local ok = pcall(callback, value)
                if ok then lastValid = box.Text else box.Text = lastValid end
            else box.Text = lastValid end
            task.defer(function() pcall(saveAllSettings) end)
        end)
        return box
    end
    local function mkKeyButton(row, entry)
        local function name() return (entry.gp and entry.gp.Name) or (entry.kb and entry.kb.Name) or "None" end
        local button = kiraUI.new("TextButton", row, {Size = UDim2.fromOffset(116, 32),
            Position = UDim2.new(1, -128, 0.5, -16), Text = name(), BackgroundColor3 = INP,
            TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = WHITE, BorderSizePixel = 0,
            AutoButtonColor = false, ZIndex = 9})
        kiraUI.round(button, 9)
        kiraUI.stroke(button, ROW_BORDER, 0.42, 1)
        local connection, axisConnection, timeout, listening = nil, nil, nil, false
        local function stop()
            listening = false; _anyKeyListening = false
            if connection then connection:Disconnect(); connection = nil end
            if axisConnection then axisConnection:Disconnect(); axisConnection = nil end
            if timeout then pcall(task.cancel, timeout); timeout = nil end
            button.Text = name(); button.TextColor3 = WHITE
        end
        button.Activated:Connect(function()
            if listening then stop(); return end
            if _anyKeyListening then return end
            listening = true; _anyKeyListening = true
            button.Text = "Press a key"; button.TextColor3 = getThemeColor()
            local started = _tick()
            local function accept(input)
                if not listening then return end
                if input.KeyCode == Enum.KeyCode.Escape then stop(); return end
                if not isBindableInput(input) then return end
                local controller = isGamepadInput(input)
                if controller and _tick() - started < 0.2 then return end
                if controller and (input.KeyCode == Enum.KeyCode.ButtonL2 or input.KeyCode == Enum.KeyCode.ButtonR2)
                    and input.Position.Z < 0.55 then return end
                if controller then entry.gp = input.KeyCode; entry.kb = nil
                else entry.kb = input.KeyCode; entry.gp = nil end
                stop(); pcall(saveAllSettings, true)
            end
            connection = UIS.InputBegan:Connect(accept)
            axisConnection = UIS.InputChanged:Connect(accept)
            timeout = task.delay(10, stop)
        end)
        table.insert(keyButtonRefs, {btn = button, entry = entry})
        return button
    end
    local function addKeybindRow(page, text, entry)
        local row = mkRow(page, 46)
        mkLabel(row, text)
        mkKeyButton(row, entry)
    end

    local speedPage = contentPages["Speed"]

    mkSect(speedPage, "Movement Speeds")
    do local row = mkRow(speedPage, 42); mkLabel(row, "Normal Speed"); normalBox = mkBox(row, NS, 50, 56, function(v) if v == v and v > 0 and v < math.huge then NS = v; if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end end end) end
    do local row = mkRow(speedPage, 42); mkLabel(row, "Carry Speed"); carryBox = mkBox(row, CS, 50, 56, function(v) if v == v and v > 0 and v < math.huge then CS = v; if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end end end) end
    do local row = mkRow(speedPage, 42); mkLabel(row, "Lagger Normal Speed"); laggerBox = mkBox(row, LAGGER_SPEED, 50, 56, function(v) if v == v and v > 0 and v < math.huge then LAGGER_SPEED = v; if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end end end) end
    do local row = mkRow(speedPage, 42); mkLabel(row, "Lagger Carry Speed"); lagger2Box = mkBox(row, LAGGER_CARRY_SPEED, 50, 56, function(v) if v == v and v > 0 and v < math.huge then LAGGER_CARRY_SPEED = v; if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end end end) end

    mkSect(speedPage, "Speed Engine")
    do
        local row = mkRow(speedPage, 46)
        mkLabel(row, "Speed Motor")
        kiraUI.motorLabel = mkSelector(row, useVenixCarryEngine and "Carry Boost" or "Direct", {"Direct", "Carry Boost"}, function(direction, update)
            useVenixCarryEngine = not useVenixCarryEngine
            update(useVenixCarryEngine and "Carry Boost" or "Direct")
            if _G.__RivalHubRefreshSpeedEngine then _G.__RivalHubRefreshSpeedEngine() end
        end)
    end
    kiraUI.carrySetter = mkToggle(speedPage, "Auto Carry", function(on)
        kiraVenixCarry:setSoftStealEnabled(on)
        if on then useVenixCarryEngine = true end
        if kiraUI.motorLabel then kiraUI.motorLabel.Text = useVenixCarryEngine and "Carry Boost" or "Direct" end
        if _G.__RivalHubRefreshSpeedEngine then _G.__RivalHubRefreshSpeedEngine() end
    end)
    do
        local row = mkRow(speedPage, 46); mkLabel(row, "Auto Carry Speed")
        kiraUI.carrySpeedBox = mkBox(row, kiraVenixCarry.softStealSpeed, 60, 72, function(v)
            kiraVenixCarry:setSoftStealSpeed(v); kiraUI.carrySpeedBox.Text = tostring(kiraVenixCarry.softStealSpeed)
        end)
    end
    do
        local row = mkRow(speedPage, 46); mkLabel(row, "Auto Carry Radius")
        kiraUI.carryRadiusBox = mkBox(row, kiraVenixCarry.softStealRadius, 60, 72, function(v)
            kiraVenixCarry:setSoftStealRadius(v); kiraUI.carryRadiusBox.Text = tostring(kiraVenixCarry.softStealRadius)
        end)
    end
    mkSect(speedPage, "Active Mode")
    do
        local row = mkRow(speedPage, 42)
        mkLabel(row, "Current Mode")
        modeValLbl = Instance.new("TextLabel", row)
        modeValLbl.Size = UDim2.new(0, 110, 1, 0)
        modeValLbl.Position = UDim2.new(1, -118, 0, 0)
        modeValLbl.BackgroundTransparency = 1
        modeValLbl.Text = "Normal"
        modeValLbl.TextColor3 = WHITE
        modeValLbl.Font = Enum.Font.GothamBold
        modeValLbl.TextSize = 12
        modeValLbl.TextXAlignment = Enum.TextXAlignment.Right
        modeValLbl.ZIndex = 8
        local clk = Instance.new("TextButton", row)
        clk.Size = UDim2.new(1,0,1,0)
        clk.BackgroundTransparency = 1
        clk.Text = ""
        clk.AutoButtonColor = false
        clk.ZIndex = 8
        clk.MouseButton1Click:Connect(function() toggleCarryMode() end)
    end

    mkSect(speedPage, "Auto Movement")
    autoLeftSetVisual = mkToggle(speedPage, "Auto Left", function(on)
        autoLeftEnabled = on
        if on then startAutoLeft() else stopAutoLeft() end
        if mobSetAutoLeft then mobSetAutoLeft(on) end
    end)
    autoRightSetVisual = mkToggle(speedPage, "Auto Right", function(on)
        autoRightEnabled = on
        if on then startAutoRight() else stopAutoRight() end
        if mobSetAutoRight then mobSetAutoRight(on) end
    end)

    mkSect(speedPage, "Infinite Jump")
    infJumpSetVisual = mkToggle(speedPage, "Infinite Jump", function(on)
        if on then InfiniteJump.start() else InfiniteJump.stop() end
    end)
    do
        local row = mkRow(speedPage, 42)
        mkLabel(row, "Jump Mode")
        local modeBtn = Instance.new("TextButton", row)
        modeBtn.Size = UDim2.new(0, 100, 0, 32)
        modeBtn.Position = UDim2.new(1, -108, 0.5, -16)
        modeBtn.BackgroundColor3 = INP
        modeBtn.BackgroundTransparency = 0.4
        modeBtn.BorderSizePixel = 0
        modeBtn.Text = "HOLD"
        modeBtn.TextColor3 = WHITE
        modeBtn.Font = Enum.Font.GothamBold
        modeBtn.TextSize = 11
        modeBtn.ZIndex = 8
        Instance.new("UICorner", modeBtn).CornerRadius = UDim.new(0, 6)
        infJumpModeBtn = modeBtn
        modeBtn.Activated:Connect(function()
            local nextMode = InfiniteJump.mode == "hold" and "manual" or "hold"
            InfiniteJump.setMode(nextMode)
            modeBtn.Text = string.upper(nextMode)
            if InfiniteJump.enabled then InfiniteJump.stop(); InfiniteJump.start() end
            pcall(saveAllSettings, true)
        end)
    end

    local combatPage = contentPages["Custom"]

    mkSect(combatPage, "Cores Arco-Íris")
    local customColorToggleCombat = mkToggle(combatPage, "Ativar Cores (Botões & Steal)", function(on)
        setCustomColorEnabled(on)
    end)
    table.insert(_customColorToggles, customColorToggleCombat)
    if customColorToggleCombat then customColorToggleCombat(customColorEnabled) end

    do
        local row = mkRow(combatPage, 42)
        mkLabel(row, "Cor Selecionada")
        local function currentThemeIndex()
            for i, k in ipairs(COLOR_THEME_KEYS) do
                if k == currentColorTheme then return i end
            end
            return 1
        end
        colorSelectorLabel = mkSelector(row, COLOR_DISPLAY_NAMES[currentColorTheme] or currentColorTheme, COLOR_THEME_KEYS, function(direction, updateLabel)
            local idx = currentThemeIndex() + direction
            if idx < 1 then idx = #COLOR_THEME_KEYS end
            if idx > #COLOR_THEME_KEYS then idx = 1 end
            local chosen = COLOR_THEME_KEYS[idx]
            applyColorTheme(chosen)
            updateLabel(COLOR_DISPLAY_NAMES[chosen] or chosen)
        end)
        if colorSelectorLabel then
            colorSelectorLabel.Text = COLOR_DISPLAY_NAMES[currentColorTheme] or currentColorTheme
            colorSelectorLabel.TextColor3 = (currentColorTheme == "Rainbow") and Color3.fromRGB(255, 215, 0) or getThemeColor()
        end
    end

    local colorButtonRefs = {}
    local function refreshColorButtonSelection()
        for key, ref in pairs(colorButtonRefs) do
            local isSel = (key == currentColorTheme)
            if ref.stroke then
                ref.stroke.Color = isSel and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
                ref.stroke.Thickness = isSel and 2.4 or 1
                ref.stroke.Transparency = isSel and 0 or 0.50
            end
            if ref.check then
                ref.check.Visible = isSel
            end
        end
    end
    _G.__refreshColorButtonSelection = refreshColorButtonSelection

    local colorRows = {
        {"Red", "Orange", "Yellow", "Lime"},
        {"Green", "Cyan", "Blue", "Indigo"},
        {"Violet", "Purple", "Pink", "White", "Rainbow"},
    }

    for _, rowGroup in ipairs(colorRows) do
        local row = mkRow(combatPage, 36)
        row.BackgroundTransparency = 0.35
        local count = #rowGroup
        for colIdx, colorKey in ipairs(rowGroup) do
            local widthScale = 1 / count
            local btn = kiraUI.new("TextButton", row, {
                Name = "ColorBtn_" .. colorKey,
                Size = UDim2.new(widthScale, -4, 0.82, 0),
                Position = UDim2.new((colIdx - 1) * widthScale, 2, 0.09, 0),
                BackgroundColor3 = COLOR_THEMES[colorKey] or Color3.fromRGB(255, 255, 255),
                BorderSizePixel = 0,
                Text = "",
                AutoButtonColor = false,
                ZIndex = 9
            })
            kiraUI.round(btn, 7)
            local btnStroke = kiraUI.stroke(btn, Color3.fromRGB(0, 0, 0), 0.50, 1)

            if colorKey == "Rainbow" then
                local rGrad = kiraUI.new("UIGradient", btn, {
                    Rotation = 45,
                    Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 60, 60)),
                        ColorSequenceKeypoint.new(0.20, Color3.fromRGB(255, 140, 30)),
                        ColorSequenceKeypoint.new(0.40, Color3.fromRGB(255, 225, 40)),
                        ColorSequenceKeypoint.new(0.60, Color3.fromRGB(50, 215, 90)),
                        ColorSequenceKeypoint.new(0.80, Color3.fromRGB(50, 130, 255)),
                        ColorSequenceKeypoint.new(1.00, Color3.fromRGB(165, 60, 255)),
                    })
                })
            end

            local isLight = (colorKey == "White" or colorKey == "Yellow" or colorKey == "Lime")
            local lbl = kiraUI.text(btn, COLOR_DISPLAY_NAMES[colorKey] or colorKey, UDim2.fromScale(0, 0), UDim2.fromScale(1, 1), 9, isLight and Color3.fromRGB(20, 20, 20) or Color3.fromRGB(255, 255, 255), 10)
            lbl.Font = Enum.Font.GothamBold
            lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            lbl.TextStrokeTransparency = isLight and 0.85 or 0.25
            lbl.TextXAlignment = Enum.TextXAlignment.Center

            local check = kiraUI.text(btn, "✓", UDim2.new(1, -13, 0, 2), UDim2.fromOffset(11, 11), 9, isLight and Color3.fromRGB(20, 20, 20) or Color3.fromRGB(255, 255, 255), 11)
            check.Font = Enum.Font.GothamBlack
            check.Visible = (colorKey == currentColorTheme)

            btn.Activated:Connect(function()
                applyColorTheme(colorKey)
                if colorSelectorLabel then
                    colorSelectorLabel.Text = COLOR_DISPLAY_NAMES[colorKey] or colorKey
                    colorSelectorLabel.TextColor3 = (colorKey == "Rainbow") and Color3.fromRGB(255, 215, 0) or getThemeColor()
                end
                refreshColorButtonSelection()
            end)

            colorButtonRefs[colorKey] = {btn = btn, stroke = btnStroke, check = check}
        end
    end

    mkSect(combatPage, "Defense")
    setAntiBatVisual = mkToggle(combatPage, "Anti Bat", function(on)
        if on then startAntiBat() else stopAntiBat() end
    end)
    setAntiFlingVisual = mkToggle(combatPage, "Anti Fling", function(on)
        antiFlingEnabled = on
        if on then startAntiFling() else stopAntiFling() end
        saveAllSettings()
    end)
    if setAntiFlingVisual then setAntiFlingVisual(antiFlingEnabled) end
    mkSect(combatPage, "Anti Ragdoll")
    do
        local row = mkRow(combatPage, 46)
        mkLabel(row, "Anti Ragdoll")
        local modes = {"off", "v1", "v2"}
        local label = mkSelector(row, antiRagdollMode:upper(), modes, function(direction, update)
            local index = antiRagdollMode == "v1" and 2 or (antiRagdollMode == "v2" and 3 or 1)
            index = (index - 1 + direction) % #modes + 1
            setAntiRagdollMode(modes[index])
            update(antiRagdollMode:upper())
        end)
        _G.updateAntiRagdollUI = function(mode) label.Text = mode:upper() end
        setAntiRagVisual = function(on) if not on then setAntiRagdollMode("off") end end
    end

    setAntiDieVisual = mkToggle(combatPage, "Anti Die", function(on)
        antiDieEnabled = on
        if on then AntiDieModule.start() else AntiDieModule.stop() end
        saveAllSettings()
    end)
    if setAntiDieVisual then setAntiDieVisual(antiDieEnabled) end

    setUnwalkVisual = mkToggle(combatPage, "Unwalk", function(on)
        unwalkEnabled = on
        if on then startUnwalk() else stopUnwalk() end
    end)

    mkSect(combatPage, "Attack")
    do
        -- One compact arrow selector for Bat Bypass, Aimbot V2 and Aimbot V3.
        local row = mkRow(combatPage, 42)
        mkLabel(row, "Aimbot / Bat Bypass")
        local modes = {"bat bypass", "v2", "v3"}
        local selectedMode = "v2"
        local function modeLabel(mode)
            if mode == "bat bypass" then return "BAT BYPASS" end
            return mode == "v3" and "AIMBOT V3" or "AIMBOT V2"
        end
        local function currentIndex()
            for i, mode in ipairs(modes) do if mode == selectedMode then return i end end
            return 2
        end
        local selector = mkSelector(row, modeLabel(selectedMode), modes, function(direction, update)
            local index = (currentIndex() - 1 + direction) % #modes + 1
            local previous = selectedMode
            selectedMode = modes[index]
            update(modeLabel(selectedMode))
            if selectedMode == "bat bypass" then
                autoBatVersion = "v2"
                if autoBatSetVisual then pcall(autoBatSetVisual, true) end
            else
                autoBatVersion = selectedMode
                if previous == "bat bypass" and autoBatSetVisual then pcall(autoBatSetVisual, false) end
                resetState()
            end
            if saveAllSettings then pcall(saveAllSettings, true) end
        end)
    end
    do local row = mkRow(combatPage, 42); mkLabel(row, "Bypass Speed"); batSpeedBox = mkBox(row, BYPASS_AIMBOT_SPEED, 50, 56, function(v) if v == v and v > 0 and v < math.huge then BAT_AIMBOT_SPEED = v; BYPASS_AIMBOT_SPEED = v; State.bypassBatSpeed = v; saveAllSettings() end end) end

    mkSect(combatPage, "Targeting")
    bodyLockSetVisual = mkToggle(combatPage, "Lock Enemy", function(on)
        bodyLockEnabled = on
        if on then
            if _blSuppressCount == 0 then startBodyLock() end
        else
            stopBodyLock()
        end
    end)
    do
        local row = mkRow(combatPage, 42)
        mkLabel(row, "Lock Enemy Range")
        bodyLockRangeBox = mkBox(row, bodyLockRange, 50, 56, function(v)
            if v and v > 0 then
                bodyLockRange = _clamp(_floor(v), 5, 200)
                if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
            end
        end)
    end

    mkSect(combatPage, "Counters")
    setBatCounterVisual = mkToggle(combatPage, "Bat Counter", function(on)
        batCounterEnabled = on
        if on then startBatCounter() else stopBatCounter() end
    end)
    setBatCounterV2Visual = mkToggle(combatPage, "Bat Counter V2", function(on)
        batCounterV2Enabled = on
        if on then startBatCounterV2() else stopBatCounterV2() end
    end)
    mkSect(combatPage, "Drop")
    dropBrainrotSetVisual = mkToggle(combatPage, "Drop Brainrot", function(on)
        if on then
            executeDropWithToggle(function(v)
                dropBrainrotSetVisual(v)
                if mobSetDropBR then mobSetDropBR(v) end
            end)
        end
    end)
    setDropVisual = dropBrainrotSetVisual
    do
        local row = mkRow(combatPage, 42)
        mkLabel(row, "Drop Mode")
        dropModeBtnRef = mkSelector(row, dropMode == 1 and "Fling" or "Jump Drop", {"Fling", "Jump Drop"}, function(dir, update)
            if dropActive then stopDropBrainrot() end
            dropMode = dropMode == 1 and 2 or 1
            update(dropMode == 1 and "Fling" or "Jump Drop")
        end)
    end

    local visualPage = contentPages["Visual"]

    mkSect(visualPage, "Interface")
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Background")

        local options = (BACKGROUND_OPTIONS and #BACKGROUND_OPTIONS > 0) and BACKGROUND_OPTIONS or {"None", "Kira Verão", "Kira Natal", "Kira Halloween"}

        local function currentOptionIndex()
            for i, opt in ipairs(options) do
                if opt == backgroundMode then return i end
            end
            if backgroundMode == "None" then return 1 end
            local n = tonumber(string.match(tostring(backgroundMode), "(%d+)")) or 1
            return _clamp(n + 1, 1, #options)
        end

        backgroundSelectorLabel = mkSelector(row, backgroundMode, options, function(direction, updateLabel)
            local idx = currentOptionIndex() + direction
            if idx < 1 then idx = #options end
            if idx > #options then idx = 1 end
            applyBackgroundMode(options[idx])
            updateLabel(backgroundMode)
        end)
        if backgroundSelectorLabel then
            backgroundSelectorLabel.TextScaled = true
        end
        applyBackgroundMode(backgroundMode)
    end
    local customColorToggleVisual = mkToggle(visualPage, "Cores Botões & Auto Steal", function(on)
        setCustomColorEnabled(on)
    end)
    table.insert(_customColorToggles, customColorToggleVisual)
    if customColorToggleVisual then customColorToggleVisual(customColorEnabled) end

    setLockUIVisual = mkToggle(visualPage, "Lock UI", function(on) toggleLockUI(on) end)
    setHideButtonsVisual = mkToggle(visualPage, "Hide Button", function(on) toggleHideButtons(on) end)
    if setHideButtonsVisual then setHideButtonsVisual(hideButtonsEnabled) end

    kiraUI.lockButtonsSetter = mkToggle(visualPage, "Lock Mobile Buttons", function(on) mobileButtonsLocked = on end)
    kiraUI.effectsSetter = mkToggle(visualPage, "Halloween Effects", function(on) kiraUI.effects = on end)
    mkSect(visualPage, "Display")
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "UI Scale")
        uiScaleBox = mkBox(row, uiScaleValue, 50, 56, function(v)
            local n = _clamp(_floor(v+0.5), 50, 150)
            uiScaleValue = n
            if kiraUI.fit then kiraUI.fit() end
            if pbScale then pbScale.Scale = n/100 end
        end)
    end
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Button Scale")
        local box = mkBox(row, _floor(floatingButtonScale * 100), 50, 56, function(v)
            local val = _clamp(v, 50, 200)
            floatingButtonScale = val / 100
            applyFloatingButtonScale()
            saveAllSettings()
        end)
    end

    mkSect(visualPage, "Character")
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Anim Pack")
        local currentIndex = 1
        for i, entry in ipairs(ANIM_PACK_ORDER) do
            if entry[2] == currentAnimPack then currentIndex = i; break end
        end
        local container = Instance.new("Frame", row)
        container.Size = UDim2.new(0, 160, 1, 0)
        container.Position = UDim2.new(1, -168, 0, 0)
        container.BackgroundTransparency = 1
        container.ZIndex = 8
        local leftBtn = Instance.new("TextButton", container)
        leftBtn.Size = UDim2.new(0, 28, 0, 26)
        leftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        leftBtn.BackgroundColor3 = INP
        leftBtn.BackgroundTransparency = 0.10
        leftBtn.BorderSizePixel = 0
        leftBtn.Text = "<"
        leftBtn.TextColor3 = WHITE
        leftBtn.Font = Enum.Font.GothamBold
        leftBtn.TextSize = 13
        leftBtn.AutoButtonColor = false
        leftBtn.ZIndex = 9
        Instance.new("UICorner", leftBtn).CornerRadius = UDim.new(0, 6)
        animSelectorLabel = Instance.new("TextLabel", container)
        animSelectorLabel.Size = UDim2.new(0, 80, 0, 26)
        animSelectorLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        animSelectorLabel.BackgroundTransparency = 1
        animSelectorLabel.Text = ANIM_PACK_ORDER[currentIndex][2]
        animSelectorLabel.TextColor3 = WHITE
        animSelectorLabel.Font = Enum.Font.GothamBold
        animSelectorLabel.TextSize = 12
        animSelectorLabel.TextXAlignment = Enum.TextXAlignment.Center
        animSelectorLabel.ZIndex = 9
        local rightBtn = Instance.new("TextButton", container)
        rightBtn.Size = UDim2.new(0, 28, 0, 26)
        rightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        rightBtn.BackgroundColor3 = INP
        rightBtn.BackgroundTransparency = 0.10
        rightBtn.BorderSizePixel = 0
        rightBtn.Text = ">"
        rightBtn.TextColor3 = WHITE
        rightBtn.Font = Enum.Font.GothamBold
        rightBtn.TextSize = 13
        rightBtn.AutoButtonColor = false
        rightBtn.ZIndex = 9
        Instance.new("UICorner", rightBtn).CornerRadius = UDim.new(0, 6)
        local function updateAnimSelector(direction)
            local idx = 1
            for i, entry in ipairs(ANIM_PACK_ORDER) do
                if entry[2] == currentAnimPack then idx = i; break end
            end
            local newIdx = idx + direction
            if newIdx < 1 then newIdx = #ANIM_PACK_ORDER end
            if newIdx > #ANIM_PACK_ORDER then newIdx = 1 end
            local packName = ANIM_PACK_ORDER[newIdx][2]
            if packName == "Off" then stopAnimPack() else startAnimPack(packName) end
        end
        leftBtn.MouseButton1Click:Connect(function() updateAnimSelector(-1) end)
        rightBtn.MouseButton1Click:Connect(function() updateAnimSelector(1) end)
    end

    do
        mkSect(visualPage, "KiraVS Skins")
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Skin")
        local container = Instance.new("Frame", row)
        container.Size = UDim2.new(0, 160, 1, 0)
        container.Position = UDim2.new(1, -168, 0, 0)
        container.BackgroundTransparency = 1
        container.ZIndex = 8

        local leftBtn = Instance.new("TextButton", container)
        leftBtn.Size = UDim2.new(0, 28, 0, 26)
        leftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        leftBtn.BackgroundColor3 = INP
        leftBtn.BackgroundTransparency = 0.10
        leftBtn.BorderSizePixel = 0
        leftBtn.Text = "<"
        leftBtn.TextColor3 = WHITE
        leftBtn.Font = Enum.Font.GothamBold
        leftBtn.TextSize = 13
        leftBtn.AutoButtonColor = false
        leftBtn.ZIndex = 9
        Instance.new("UICorner", leftBtn).CornerRadius = UDim.new(0, 6)

        outfitSelectorLabel = Instance.new("TextLabel", container)
        outfitSelectorLabel.Size = UDim2.new(0, 80, 0, 26)
        outfitSelectorLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        outfitSelectorLabel.BackgroundTransparency = 1
        outfitSelectorLabel.Text = (OUTFITS[currentOutfitIndex] and OUTFITS[currentOutfitIndex].label) or "OFF"
        outfitSelectorLabel.TextColor3 = WHITE
        outfitSelectorLabel.Font = Enum.Font.GothamBold
        outfitSelectorLabel.TextSize = 11
        outfitSelectorLabel.TextXAlignment = Enum.TextXAlignment.Center
        outfitSelectorLabel.ZIndex = 9

        local rightBtn = Instance.new("TextButton", container)
        rightBtn.Size = UDim2.new(0, 28, 0, 26)
        rightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        rightBtn.BackgroundColor3 = INP
        rightBtn.BackgroundTransparency = 0.10
        rightBtn.BorderSizePixel = 0
        rightBtn.Text = ">"
        rightBtn.TextColor3 = WHITE
        rightBtn.Font = Enum.Font.GothamBold
        rightBtn.TextSize = 13
        rightBtn.AutoButtonColor = false
        rightBtn.ZIndex = 9
        Instance.new("UICorner", rightBtn).CornerRadius = UDim.new(0, 6)

        local function updateSkinSelector(direction)
            local newIndex = currentOutfitIndex + direction
            if newIndex < 1 then newIndex = #OUTFITS end
            if newIndex > #OUTFITS then newIndex = 1 end
            currentOutfitIndex = newIndex
            outfitSelectorLabel.Text = OUTFITS[newIndex].label
            task.spawn(function()
                local ok, err = pcall(function() applyOutfitByIndex(newIndex) end)
                if not ok then warn("[Rival Hub] Falha ao aplicar skin " .. OUTFITS[newIndex].label .. ": " .. tostring(err)) end
                pcall(saveAllSettings)
            end)
        end
        leftBtn.Activated:Connect(function() updateSkinSelector(-1) end)
        rightBtn.Activated:Connect(function() updateSkinSelector(1) end)
    end

    mkSect(visualPage, "Effects")
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "kira Title Color")
        local container = Instance.new("Frame", row)
        container.Size = UDim2.new(0, 160, 1, 0)
        container.Position = UDim2.new(1, -168, 0, 0)
        container.BackgroundTransparency = 1
        container.ZIndex = 8
        local leftBtn = Instance.new("TextButton", container)
        leftBtn.Size = UDim2.new(0, 28, 0, 26)
        leftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        leftBtn.BackgroundColor3 = INP
        leftBtn.BackgroundTransparency = 0.10
        leftBtn.BorderSizePixel = 0
        leftBtn.Text = "<"
        leftBtn.TextColor3 = WHITE
        leftBtn.Font = Enum.Font.GothamBold
        leftBtn.TextSize = 13
        leftBtn.AutoButtonColor = false
        leftBtn.ZIndex = 9
        Instance.new("UICorner", leftBtn).CornerRadius = UDim.new(0, 6)
        rivalTitleSelectorLabel = Instance.new("TextLabel", container)
        rivalTitleSelectorLabel.Size = UDim2.new(0, 80, 0, 26)
        rivalTitleSelectorLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        rivalTitleSelectorLabel.BackgroundTransparency = 1
        rivalTitleSelectorLabel.Text = currentRivalTitleTheme
        rivalTitleSelectorLabel.TextColor3 = WHITE
        rivalTitleSelectorLabel.Font = Enum.Font.GothamBold
        rivalTitleSelectorLabel.TextSize = 12
        rivalTitleSelectorLabel.TextXAlignment = Enum.TextXAlignment.Center
        rivalTitleSelectorLabel.ZIndex = 9
        local rightBtn = Instance.new("TextButton", container)
        rightBtn.Size = UDim2.new(0, 28, 0, 26)
        rightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        rightBtn.BackgroundColor3 = INP
        rightBtn.BackgroundTransparency = 0.10
        rightBtn.BorderSizePixel = 0
        rightBtn.Text = ">"
        rightBtn.TextColor3 = WHITE
        rightBtn.Font = Enum.Font.GothamBold
        rightBtn.TextSize = 13
        rightBtn.AutoButtonColor = false
        rightBtn.ZIndex = 9
        Instance.new("UICorner", rightBtn).CornerRadius = UDim.new(0, 6)

        local RIVAL_THEME_LIST = { "Orange", "Blue", "Red", "Green", "Black", "Gray" }
        local function cycleRivalTitle(dir)
            local idx = 1
            for i, name in ipairs(RIVAL_THEME_LIST) do
                if name == currentRivalTitleTheme then idx = i; break end
            end
            local newIdx = idx + dir
            if newIdx < 1 then newIdx = #RIVAL_THEME_LIST end
            if newIdx > #RIVAL_THEME_LIST then newIdx = 1 end
            applyRivalTitleTheme(RIVAL_THEME_LIST[newIdx])
        end
        leftBtn.MouseButton1Click:Connect(function() cycleRivalTitle(-1) end)
        rightBtn.MouseButton1Click:Connect(function() cycleRivalTitle(1) end)
    end
    setVividVisual = mkToggle(visualPage, "Vivid Graphics", function(on)
        toggleVividGraphics(on)
        if setVividVisual then setVividVisual(on) end
    end)
    if setVividVisual then setVividVisual(vividGraphicsEnabled) end
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Stretch Rez")
        local stretchPill, stretchDot = mkPill(row, 48)
        local stretchOn = false
        local function setStretch(s)
            stretchOn = s
            animPill(stretchPill, stretchDot, s)
            if s then enableStretch() else disableStretch() end
            stretchEnabled = s
        end
        local stretchClk = Instance.new("TextButton", stretchPill)
        stretchClk.Size = UDim2.new(1,0,1,0)
        stretchClk.BackgroundTransparency = 1
        stretchClk.Text = ""
        stretchClk.AutoButtonColor = false
        stretchClk.ZIndex = 10
        stretchClk.MouseButton1Click:Connect(function() setStretch(not stretchOn) end)
        _G.stretchToggleSetter = setStretch
    end
    setAntiLagVisual = mkToggle(visualPage, "Anti Lag", function(on)
        if on then enableAntiLag() else disableAntiLag() end
        antiLagEnabled = on == true
        pcall(saveAllSettings, true)
    end)
    do
        local row = mkRow(visualPage, 42)
        mkLabel(row, "Sky Theme")
        local container = Instance.new("Frame", row)
        container.Size = UDim2.new(0, 160, 1, 0)
        container.Position = UDim2.new(1, -168, 0, 0)
        container.BackgroundTransparency = 1
        container.ZIndex = 8
        local leftBtn = Instance.new("TextButton", container)
        leftBtn.Size = UDim2.new(0, 28, 0, 26)
        leftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        leftBtn.BackgroundColor3 = INP
        leftBtn.BackgroundTransparency = 0.10
        leftBtn.BorderSizePixel = 0
        leftBtn.Text = "<"
        leftBtn.TextColor3 = WHITE
        leftBtn.Font = Enum.Font.GothamBold
        leftBtn.TextSize = 13
        leftBtn.AutoButtonColor = false
        leftBtn.ZIndex = 9
        Instance.new("UICorner", leftBtn).CornerRadius = UDim.new(0, 6)
        skySelectorLabel = Instance.new("TextLabel", container)
        skySelectorLabel.Size = UDim2.new(0, 96, 0, 26)
        skySelectorLabel.Position = UDim2.new(0.5, -48, 0.5, -13)
        skySelectorLabel.BackgroundTransparency = 1
        skySelectorLabel.Text = skyTheme
        skySelectorLabel.TextColor3 = WHITE
        skySelectorLabel.Font = Enum.Font.GothamBold
        skySelectorLabel.TextSize = 11
        skySelectorLabel.TextXAlignment = Enum.TextXAlignment.Center
        skySelectorLabel.ZIndex = 9
        local rightBtn = Instance.new("TextButton", container)
        rightBtn.Size = UDim2.new(0, 28, 0, 26)
        rightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        rightBtn.BackgroundColor3 = INP
        rightBtn.BackgroundTransparency = 0.10
        rightBtn.BorderSizePixel = 0
        rightBtn.Text = ">"
        rightBtn.TextColor3 = WHITE
        rightBtn.Font = Enum.Font.GothamBold
        rightBtn.TextSize = 13
        rightBtn.AutoButtonColor = false
        rightBtn.ZIndex = 9
        Instance.new("UICorner", rightBtn).CornerRadius = UDim.new(0, 6)
        local function updateSkySelector(direction)
            local idx = 1
            for i, name in ipairs(SKY_PRESETS_LIST) do
                if name == skyTheme then idx = i; break end
            end
            local newIdx = idx + direction
            if newIdx < 1 then newIdx = #SKY_PRESETS_LIST end
            if newIdx > #SKY_PRESETS_LIST then newIdx = 1 end
            local name = SKY_PRESETS_LIST[newIdx]
            skyTheme = name
            pcall(applyCustomSky, name)
            if skySelectorLabel then skySelectorLabel.Text = name end
            pcall(saveAllSettings, true)
        end
        leftBtn.MouseButton1Click:Connect(function() updateSkySelector(-1) end)
        rightBtn.MouseButton1Click:Connect(function() updateSkySelector(1) end)
    end

    mkSect(visualPage, "Overlays")
    setESPVIsual = mkToggle(visualPage, "Player ESP", function(on) toggleESP(on) end)
    setESPLineVisual = mkToggle(visualPage, "ESP Line", function(on)
        if on then startESPLine() else stopESPLine() end
    end)

    local configPage = contentPages["Settings"]

    mkSect(configPage, "Automation")

    local mkToggleOrange = mkToggle

    setInstaGrab = mkToggleOrange(configPage, "Auto Steal", function(on)
        CONFIG.AUTO_STEAL_ENABLED = on == true
        if on then pcall(startAutoSteal) else stopAutoSteal() end
        updateProgressBarVisibility()
        pcall(saveAllSettings, true)
    end)
    do
        local row = mkRow(configPage, 46)
        mkLabel(row, "Auto Steal Mode")
        autoStealModeBtn = mkSelector(row, autoStealMode, AUTO_STEAL_VARIANT_NAMES, function(direction, update)
            local index = (autoStealVariant - 1 + direction) % #AUTO_STEAL_VARIANT_NAMES + 1
            local enabled = CONFIG.AUTO_STEAL_ENABLED
            if enabled then stopAutoSteal() end
            setAutoStealMode(AUTO_STEAL_VARIANT_NAMES[index])
            update(autoStealMode)
            if enabled then startAutoSteal() end
        end)
    end

    do
        local row = mkRow(configPage, 42)
        mkLabel(row, "Steal Radius")
        local function setRadius(v, skipSave)
            v = _clamp(_floor(v + 0.5), 5, 300)
            CONFIG.STEAL_RANGE = v
            Steal.StealRadius = autoStealVariant == 6 and 10 or CONFIG.STEAL_RANGE
            if radInput then radInput.Text = tostring(v) end
            if not skipSave then pcall(saveAllSettings) end
        end
        radInput = mkBox(row, CONFIG.STEAL_RANGE, 64, 112, function(v)
            if v and v >= 5 and v <= 300 then
                CONFIG.STEAL_RANGE = _floor(v+0.5)
                Steal.StealRadius = autoStealVariant == 6 and 10 or CONFIG.STEAL_RANGE
                radInput.Text = tostring(CONFIG.STEAL_RANGE)
                saveAllSettings()
            end
        end)
        for _, spec in ipairs({{-1, -140}, {1, -40}}) do
            local dir, xoff = spec[1], spec[2]
            local btn = kiraUI.new("TextButton", row, {Size = UDim2.fromOffset(28, 28),
                Position = UDim2.new(1, xoff, 0.5, -14), BackgroundColor3 = INP, BorderSizePixel = 0,
                Text = dir == -1 and "-" or "+", TextSize = 18, Font = Enum.Font.GothamBold,
                TextColor3 = getThemeColor(), AutoButtonColor = false, ZIndex = 9})
            kiraUI.round(btn, 6)
            kiraUI.stroke(btn, ROW_BORDER, 0.68, 1)
            -- Clique dá 1 de passo; segurar vai seguindo sozinho.
            btn.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or
                    input.UserInputType == Enum.UserInputType.Touch then
                    setRadius(CONFIG.STEAL_RANGE + dir, true)
                    if btn:GetAttribute("kiraHolding") then return end
                    btn:SetAttribute("kiraHolding", true)
                    task.spawn(function()
                        task.wait(0.4)
                        while btn.Parent and btn:GetAttribute("kiraHolding") do
                            setRadius(CONFIG.STEAL_RANGE + dir, true)
                            task.wait(0.07)
                        end
                    end)
                end
            end)
            btn.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or
                    input.UserInputType == Enum.UserInputType.Touch then
                    btn:SetAttribute("kiraHolding", false)
                    pcall(saveAllSettings)
                end
            end)
        end
    end

    do
        local row = mkRow(configPage, 44)
        row.Size = UDim2.new(1, 0, 0, 44)
        local plBtn = Instance.new("TextButton", row)
        plBtn.Size = UDim2.new(1, -12, 0.8, 0)
        plBtn.Position = UDim2.new(0, 6, 0.1, 0)
        plBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
        plBtn.BorderSizePixel = 0
        plBtn.Text = "OPEN PING LAGGER"
        plBtn.TextColor3 = Color3.fromRGB(240, 214, 175)
        plBtn.Font = Enum.Font.GothamBold
        plBtn.TextSize = 13
        plBtn.AutoButtonColor = true
        plBtn.ZIndex = 8
        kiraUI.round(plBtn, 8)
        plBtn.Activated:Connect(function()
            local existing
            pcall(function()
                local holder = gethui and gethui() or game:GetService("CoreGui")
                existing = holder:FindFirstChild("SpeexPingMenu") or LP.PlayerGui:FindFirstChild("SpeexPingMenu")
            end)
            if existing then
                existing.Enabled = not existing.Enabled
            else
                task.spawn(function()
                    local ok, err = pcall(_G.__RivalOpenPingLagger)
                    if not ok then warn("[Ping Lagger] " .. tostring(err)) end
                end)
            end
        end)
    end

    mkSect(configPage, "Management")
    do
        local row = mkRow(configPage, 44)
        row.Size = UDim2.new(1, 0, 0, 44)
        local saveBtn = Instance.new("TextButton", row)
        saveBtn.Size = UDim2.new(1, -12, 0.8, 0)
        saveBtn.Position = UDim2.new(0, 6, 0.1, 0)
        saveBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
        saveBtn.BorderSizePixel = 0
        saveBtn.Text = "SAVE CONFIG"
        saveBtn.TextColor3 = Color3.fromRGB(240, 214, 175)
        saveBtn.Font = Enum.Font.GothamBold
        saveBtn.TextSize = 13
        saveBtn.AutoButtonColor = false
        saveBtn.ZIndex = 8
        Instance.new("UICorner", saveBtn).CornerRadius = UDim.new(0, 6)
        local saveStroke = Instance.new("UIStroke", saveBtn)
        saveStroke.Color = getThemeColor()
        saveStroke.Thickness = 1.5
        saveStroke.Transparency = 0.4
        saveBtn.MouseEnter:Connect(function()
            TS:Create(saveBtn, TweenInfo.new(0.15), {BackgroundColor3 = getThemeColor()}):Play()
            TS:Create(saveBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(255,255,255)}):Play()
        end)
        saveBtn.MouseLeave:Connect(function()
            TS:Create(saveBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(27, 30, 33)}):Play()
            TS:Create(saveBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(240, 214, 175)}):Play()
        end)
        saveBtn.MouseButton1Click:Connect(function()
            local ok = saveAllSettings(true)
            saveBtn.Text = ok and "SAVED" or "ERROR"
            task.delay(1.2, function()
                if saveBtn and saveBtn.Parent then saveBtn.Text = "SAVE CONFIG" end
            end)
        end)
    end
    do
        local row = mkRow(configPage, 44)
        row.Size = UDim2.new(1, 0, 0, 44)
        local resetPosBtn = Instance.new("TextButton", row)
        resetPosBtn.Size = UDim2.new(1, -12, 0.8, 0)
        resetPosBtn.Position = UDim2.new(0, 6, 0.1, 0)
        resetPosBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
        resetPosBtn.BorderSizePixel = 0
        resetPosBtn.Text = "RESET POSITIONS"
        resetPosBtn.TextColor3 = Color3.fromRGB(240, 214, 175)
        resetPosBtn.Font = Enum.Font.GothamBold
        resetPosBtn.TextSize = 13
        resetPosBtn.AutoButtonColor = false
        resetPosBtn.ZIndex = 8
        Instance.new("UICorner", resetPosBtn).CornerRadius = UDim.new(0, 6)
        local resetStroke = Instance.new("UIStroke", resetPosBtn)
        resetStroke.Color = getThemeColor()
        resetStroke.Thickness = 1.5
        resetStroke.Transparency = 0.4
        resetPosBtn.MouseEnter:Connect(function()
            TS:Create(resetPosBtn, TweenInfo.new(0.15), {BackgroundColor3 = getThemeColor()}):Play()
            TS:Create(resetPosBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(255,255,255)}):Play()
        end)
        resetPosBtn.MouseLeave:Connect(function()
            TS:Create(resetPosBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(27, 30, 33)}):Play()
            TS:Create(resetPosBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(240, 214, 175)}):Play()
        end)
        local resetDebounce = false
        resetPosBtn.MouseButton1Click:Connect(function()
            if resetDebounce then return end
            resetDebounce = true
            resetFloatingPositions()
            resetPosBtn.Text = "RESET"
            task.delay(1.2, function()
                if resetPosBtn and resetPosBtn.Parent then
                    resetPosBtn.Text = "RESET POSITIONS"
                    resetDebounce = false
                end
            end)
        end)
    end
    do
        local row = mkRow(configPage, 44)
        row.Size = UDim2.new(1, 0, 0, 44)
        local delBtn = Instance.new("TextButton", row)
        delBtn.Size = UDim2.new(1, -12, 0.8, 0)
        delBtn.Position = UDim2.new(0, 6, 0.1, 0)
        delBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
        delBtn.BorderSizePixel = 0
        delBtn.Text = "RESET ALL SETTINGS"
        delBtn.TextColor3 = Color3.fromRGB(170, 30, 50)
        delBtn.Font = Enum.Font.GothamBold
        delBtn.TextSize = 13
        delBtn.AutoButtonColor = false
        delBtn.ZIndex = 8
        Instance.new("UICorner", delBtn).CornerRadius = UDim.new(0, 6)
        local delStroke = Instance.new("UIStroke", delBtn)
        delStroke.Color = Color3.fromRGB(200, 60, 80)
        delStroke.Thickness = 1.5
        delStroke.Transparency = 0.4
        delBtn.MouseEnter:Connect(function()
            TS:Create(delBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(200, 50, 70)}):Play()
            TS:Create(delBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(255,255,255)}):Play()
        end)
        delBtn.MouseLeave:Connect(function()
            TS:Create(delBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(27, 30, 33)}):Play()
            TS:Create(delBtn, TweenInfo.new(0.15), {TextColor3 = Color3.fromRGB(170, 30, 50)}):Play()
        end)
        local deleteState = 0
        local originalDeleteText = "RESET ALL SETTINGS"
        local delDebounce = false
        delBtn.MouseButton1Click:Connect(function()
            if delDebounce then return end
            if deleteState == 0 then
                deleteState = 1
                delBtn.Text = "CONFIRM?"
                delBtn.BackgroundColor3 = Color3.fromRGB(255, 220, 220)
                delBtn.TextColor3 = Color3.fromRGB(160, 0, 0)
                task.delay(2, function()
                    if delBtn and delBtn.Parent and deleteState == 1 then
                        deleteState = 0
                        delBtn.Text = originalDeleteText
                        delBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
                        delBtn.TextColor3 = Color3.fromRGB(170, 30, 50)
                    end
                end)
            elseif deleteState == 1 then
                delDebounce = true
                local invoked, success = pcall(resetToFactoryDefaults)
                success = invoked and success
                delBtn.Text = success and "SETTINGS RESET" or "ERROR"
                delBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
                delBtn.TextColor3 = Color3.fromRGB(170, 30, 50)
                deleteState = 0
                task.delay(1.5, function()
                    if delBtn and delBtn.Parent then
                        delBtn.Text = originalDeleteText
                        delBtn.BackgroundColor3 = Color3.fromRGB(27, 30, 33)
                        delBtn.TextColor3 = Color3.fromRGB(170, 30, 50)
                        delDebounce = false
                    end
                end)
            end
        end)
    end

    local keyPage = contentPages["Keybinds"]
    mkSect(keyPage, "Keybinds")
    mkSect(keyPage, "Movement")
    addKeybindRow(keyPage, "Carry Mode", KB.CarryToggle)
    addKeybindRow(keyPage, "Lagger Mode", KB.LaggerMode)
    addKeybindRow(keyPage, "Auto Left", KB.AutoLeft)
    addKeybindRow(keyPage, "Auto Right", KB.AutoRight)

    mkSect(keyPage, "Combat")
    addKeybindRow(keyPage, "Bat Bypass", KB.AutoBat)
    addKeybindRow(keyPage, "TP Bate", KB.BatV2)

    mkSect(keyPage, "Utility")
    addKeybindRow(keyPage, "Drop Brainrot", KB.DropBrainrot)
    addKeybindRow(keyPage, "TP Down", KB.TPFloor)
    addKeybindRow(keyPage, "Hide GUI", KB.GuiHide)

    -- Insta Reset: botão móvel + atalho configurável na página Keybinds.
    local instaResetRunning = false
    local instaResetCancelled = false
    local instaResetWorker = nil
    local instaResetCharacter = nil

    local function stopInstaReset()
        instaResetCancelled = true
        if instaResetWorker then
            pcall(task.cancel, instaResetWorker)
            instaResetWorker = nil
        end
        instaResetRunning = false
        instaResetCharacter = nil
    end

    local function runInstaReset()
        if instaResetRunning then return end
        local character = LP.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        if not character or not humanoid or humanoid.Health <= 0 then return end

        instaResetRunning = true
        instaResetCancelled = false
        instaResetCharacter = character
        instaResetWorker = task.spawn(function()
            local originalHipHeight = humanoid.HipHeight
            local originalCollision = {}
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then originalCollision[part] = part.CanCollide end
            end

            for _ = 1, 40 do
                if instaResetCancelled or not character.Parent or humanoid.Health <= 0 or LP.Character ~= character then
                    break
                end
                pcall(function()
                    humanoid.HipHeight = 1e6
                    humanoid.AutoRotate = true
                    for _, part in ipairs(character:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end)
                task.wait(0.05)
            end

            if not instaResetCancelled and character.Parent and humanoid.Health > 0 and LP.Character == character then
                pcall(function() humanoid.Health = 0 end)
            end
            if character.Parent then
                pcall(function()
                    humanoid.HipHeight = originalHipHeight
                    for part, wasCollidable in pairs(originalCollision) do
                        if part and part.Parent then part.CanCollide = wasCollidable end
                    end
                end)
            end
            instaResetRunning, instaResetWorker, instaResetCharacter, instaResetCancelled = false, nil, nil, false
        end)
    end

    LP.CharacterAdded:Connect(function()
        stopInstaReset()
    end)

    local instaResetGui = Instance.new("ScreenGui")
    instaResetGui.Name = "KiraInstaResetGui"
    instaResetGui.ResetOnSpawn = false
    instaResetGui.IgnoreGuiInset = true
    instaResetGui.DisplayOrder = 120
    instaResetGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    pcall(function()
        local core = game:GetService("CoreGui")
        if syn and syn.protect_gui then syn.protect_gui(instaResetGui) end
        instaResetGui.Parent = core
    end)
    if not instaResetGui.Parent then instaResetGui.Parent = pgui end

    local instaResetButton = kiraUI.new("TextButton", instaResetGui, {
        Name = "InstaResetButton", Size = UDim2.fromOffset(54, 54),
        Position = UDim2.new(1, -70, 0.56, -27),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255), BorderSizePixel = 0,
        Text = "RESET", TextColor3 = Color3.fromRGB(250, 241, 224),
        Font = Enum.Font.GothamBold, TextSize = 10, AutoButtonColor = true,
        Active = true, ZIndex = 100
    })
    kiraUI.round(instaResetButton, 12)
    kiraUI.new("UIGradient", instaResetButton, {Rotation = 90,
        Color = ColorSequence.new(Color3.fromRGB(39, 30, 25), Color3.fromRGB(12, 16, 22))})
    kiraUI.stroke(instaResetButton, getThemeColor(), 0.2, 2)
    instaResetButton.Activated:Connect(runInstaReset)

    -- Arrastar o botão com mouse ou toque.
    local instaDragging, instaDragStart, instaStartPos = false, nil, nil
    instaResetButton.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            instaDragging, instaDragStart, instaStartPos = true, input.Position, instaResetButton.Position
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if instaDragging and instaDragStart and instaStartPos and
            (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - instaDragStart
            instaResetButton.Position = UDim2.new(instaStartPos.X.Scale, instaStartPos.X.Offset + delta.X,
                instaStartPos.Y.Scale, instaStartPos.Y.Offset + delta.Y)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            instaDragging = false
        end
    end)

    -- Linha de configuração do atalho dentro da aba Keybinds.
    mkSect(keyPage, "Insta Reset")
    local instaBindRow = mkRow(keyPage, 46)
    mkLabel(instaBindRow, "INSTA RESET KEY")
    mkKeyButton(instaBindRow, KB.InstaReset)

    -- Dispara o reset pela tecla salva (teclado ou controle).
    UIS.InputBegan:Connect(function(input, gameProcessed)
        if _anyKeyListening then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if gameProcessed or UIS:GetFocusedTextBox() then return end
        elseif not isGamepadInput(input) then
            return
        end
        if not isBindableInput(input) then return end
        if kbMatch(KB.InstaReset, input.KeyCode) then runInstaReset() end
    end)

    local spacer = Instance.new("Frame", keyPage)
    spacer.Size = UDim2.new(1, 0, 0, 16)
    spacer.BackgroundTransparency = 1
    spacer.LayoutOrder = getNextOrder(keyPage)
    spacer.ZIndex = 7

    pbFrame = kiraUI.new("Frame", gui, {Name = "RivalHubAutoStealHud", Size = UDim2.fromOffset(510, 54),
        Position = UDim2.new(1, -528, 0, 18), BackgroundColor3 = Color3.fromRGB(10, 10, 15),
        BorderSizePixel = 0, Active = true, ZIndex = 50})
    kiraUI.hudScale = kiraUI.new("UIScale", pbFrame, {Scale = 1})
    kiraUI.round(pbFrame, 27)
    local hudStroke = kiraUI.stroke(pbFrame, Color3.fromRGB(91, 77, 92), 0.45, 1.2)
    hudStroke.Name = "OuterStroke"

    -- Barra de progresso rosa, à esquerda, no estilo da referência enviada.
    local track = kiraUI.new("Frame", pbFrame, {Name = "StealProgressTrack",
        Size = UDim2.new(0.55, -7, 1, -8), Position = UDim2.fromOffset(4, 4),
        BackgroundColor3 = Color3.fromRGB(34, 30, 39), BorderSizePixel = 0,
        ClipsDescendants = true, ZIndex = 51})
    kiraUI.round(track, 24)
    progressFill = kiraUI.new("Frame", track, {Name = "StealProgressFill",
        Size = UDim2.new(stealHud.value, 0, 1, 0), BackgroundColor3 = Color3.fromRGB(255, 35, 35),
        BorderSizePixel = 0, ZIndex = 52})
    kiraUI.round(progressFill, 24)
    kiraUI.new("UIGradient", progressFill, {Rotation = 0, Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(150, 0, 0)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 35, 35))
    })})

    progressPct = kiraUI.text(track, "0%", UDim2.fromOffset(15, 0), UDim2.new(0, 54, 1, 0),
        14, WHITE, 54)
    progressPct.Name = "StealProgressPercent"
    progressPct.Font = Enum.Font.GothamBlack
    progressPct.TextXAlignment = Enum.TextXAlignment.Left

    progressStatus = kiraUI.text(track, "STEAL", UDim2.fromOffset(78, 0), UDim2.new(1, -85, 1, 0),
        14, WHITE, 54)
    progressStatus.Name = "StealProgressStatus"
    progressStatus.Font = Enum.Font.GothamBlack
    progressStatus.TextXAlignment = Enum.TextXAlignment.Left

    -- Divisor e estatísticas de desempenho no lado direito.
    kiraUI.new("Frame", pbFrame, {Name = "StatsDivider", Size = UDim2.new(0, 2, 0.62, 0),
        Position = UDim2.new(0.56, 0, 0.19, 0), BackgroundColor3 = Color3.fromRGB(112, 106, 121),
        BackgroundTransparency = 0.25, BorderSizePixel = 0, ZIndex = 53})

    stealHud.stats = kiraUI.text(pbFrame, stealHud.statsText,
        UDim2.new(0.585, 0, 0, 0), UDim2.new(0.405, -8, 1, 0),
        12, Color3.fromRGB(242, 239, 247), 53)
    stealHud.stats.Name = "PerformanceStats"
    stealHud.stats.TextXAlignment = Enum.TextXAlignment.Center
    stealHud.stats.Font = Enum.Font.GothamBold
    updateStealProgress(stealHud.value, stealHud.state)
    drag(pbFrame)
    kiraUI.fit()
    do
        local viewConnection
        local function followCamera()
            if viewConnection then viewConnection:Disconnect() end
            local cam = Workspace.CurrentCamera
            if cam then viewConnection = cam:GetPropertyChangedSignal("ViewportSize"):Connect(kiraUI.fit) end
            kiraUI.fit()
        end
        local cameraConnection = Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(followCamera)
        followCamera()
        gui.Destroying:Connect(function()
            cameraConnection:Disconnect()
            if viewConnection then viewConnection:Disconnect() end
        end)
    end
    kiraUI.effectsLoop(gui)
    task.spawn(function() pcall(function() game:GetService("ContentProvider"):PreloadAsync({bg}) end) end)
end

do
    
    local function findGlobalFunc(...)
        local sources = {}
        pcall(function() if getgenv then table.insert(sources, getgenv()) end end)
        pcall(function() table.insert(sources, _G) end)
        pcall(function() if getfenv then table.insert(sources, getfenv()) end end)
        for i = 1, select("#", ...) do
            local name = select(i, ...)
            for _, env in ipairs(sources) do
                if type(env) == "table" then
                    local ok, val = pcall(function() return env[name] end)
                    if ok and type(val) == "function" then return val end
                end
            end
        end
        return nil
    end

    local setHiddenProp = findGlobalFunc(
        "sethiddenproperty", "set_hidden_property",
        "sethiddenprop", "set_hidden_prop"
    )
    local function setHidden(instance, prop, value)
        if not instance then return false end
        if setHiddenProp then
            local ok = pcall(setHiddenProp, instance, prop, value)
            if ok then return true end
        end
        return false
    end

    local TP = {
        enabled   = false,
        conn      = nil,
        char      = nil,
        h         = nil,
        hrp       = nil,
        cooldown  = false,
        
        batSpeed     = 40,   
        currentVersion = "V1", 
        silentAim    = false,
    }

    local function isBatTool(tool)
        if not tool or not tool:IsA("Tool") then return false end
        local n = string.lower(tool.Name)
        return n:find("bat", 1, true) ~= nil or n:find("slap", 1, true) ~= nil
    end

    local function getBat()
        local c = LP.Character or TP.char
        if not c then return nil end
        local hum = c:FindFirstChildOfClass("Humanoid")
        for _, ch in ipairs(c:GetChildren()) do
            if isBatTool(ch) then return ch end
        end
        local bp = LP:FindFirstChildOfClass("Backpack") or LP:FindFirstChild("Backpack")
        if bp then
            for _, ch in ipairs(bp:GetChildren()) do
                if isBatTool(ch) then
                    if hum then
                        pcall(function() hum:EquipTool(ch) end)
                    else
                        pcall(function() ch.Parent = c end)
                    end
                    return ch
                end
            end
        end
        return nil
    end

    local function getHitDelay()
        local t = 0.12 - ((TP.batSpeed - 10) / 70) * 0.105
        return math.clamp(t, 0.015, 0.12)
    end

    local function tryHit()
        if TP.cooldown then return end
        TP.cooldown = true
        pcall(function()
            local bat = getBat()
            if not bat then return end
            pcall(function() bat:Activate() end)
            for _, d in ipairs(bat:GetDescendants()) do
                if d:IsA("RemoteEvent") then pcall(function() d:FireServer() end) end
            end
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then pcall(function() ev:FireServer() end) end
        end)
        task.delay(getHitDelay(), function() TP.cooldown = false end)
    end

    local function getClosest()
        local myHrp = TP.hrp
        if not myHrp or not myHrp.Parent then
            local c = LP.Character
            myHrp = c and c:FindFirstChild("HumanoidRootPart")
            TP.hrp = myHrp
        end
        if not myHrp then return nil, math.huge end
        local best, bestDist = nil, math.huge
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP and p.Character then
                local hum = p.Character:FindFirstChildOfClass("Humanoid")
                local tr  = p.Character:FindFirstChild("HumanoidRootPart")
                if hum and hum.Health > 0 and tr and tr.Parent then
                    local d = (myHrp.Position - tr.Position).Magnitude
                    if d < bestDist then bestDist = d; best = p end
                end
            end
        end
        return best, bestDist
    end

    local function setupChar(newChar)
        TP.char = newChar
        task.wait(0.1)
        TP.h   = newChar:WaitForChild("Humanoid", 5)
        TP.hrp = newChar:WaitForChild("HumanoidRootPart", 5)
    end

    local function stopBatV2()
        TP.enabled = false
        if TP.conn then TP.conn:Disconnect(); TP.conn = nil end
        TP.char = nil; TP.h = nil; TP.hrp = nil
        TP.cooldown = false
        if _unsuppressBodyLock then pcall(_unsuppressBodyLock, true) end
    end

    local function startBatV2()
        if autoBatEnabled then disableAutoBat() end
        stopBatV2()
        TP.enabled = true
        TP.char = LP.Character
        if TP.char then
            TP.h   = TP.char:FindFirstChildOfClass("Humanoid")
            TP.hrp = TP.char:FindFirstChild("HumanoidRootPart")
        end
        if _suppressBodyLock then pcall(_suppressBodyLock) end

        TP.conn = RunService.Heartbeat:Connect(function()
            if not TP.enabled then return end
            local c = LP.Character
            if not c then return end
            TP.char = c
            TP.hrp  = c:FindFirstChild("HumanoidRootPart")
            TP.h    = c:FindFirstChildOfClass("Humanoid")
            if not TP.hrp or not TP.h or TP.h.Health <= 0 then return end

            local target, dist = getClosest()
            if not target or not target.Character then return end
            local tr = target.Character:FindFirstChild("HumanoidRootPart")
            if not tr then return end

            setHidden(TP.hrp, "PhysicsRepRootPart", tr)

            local targetPos = tr.Position + Vector3.new(0, 0.9, 0)
            if dist > 4 then
                TP.hrp.CFrame = CFrame.new(targetPos, tr.Position)
            elseif dist > 1.5 then
                TP.hrp.CFrame = CFrame.new(TP.hrp.Position:Lerp(targetPos, 0.55), tr.Position)
            else
                TP.hrp.CFrame = CFrame.new(targetPos, tr.Position)
            end

            pcall(function() TP.hrp.AssemblyAngularVelocity = Vector3.zero end)

            local cam = workspace.CurrentCamera
            if not TP.silentAim and cam then
                cam.CFrame = CFrame.new(cam.CFrame.Position, tr.Position + Vector3.new(0, 0.5, 0))
            end

            if TP.currentVersion == "V1" then
                tryHit()
            end
        end)
    end

    _G.__RivalHubStartBatV2 = startBatV2
    _G.__RivalHubStopBatV2  = stopBatV2
    _G.__RivalHubIsBatV2    = function() return TP.enabled == true end
    _G.__RivalHubBatV2      = TP

    LP.CharacterAdded:Connect(function(newChar)
        task.wait(0.5)
        setupChar(newChar)
        if TP.enabled then
            startBatV2()
        end
    end)
    if LP.Character then task.spawn(function() setupChar(LP.Character) end) end
end

function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name = "RivalHubMobilePanel"
    panel.ResetOnSpawn = false
    panel.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    panel.IgnoreGuiInset = true
    panel.DisplayOrder = 40
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    local okPanel = pcall(function() panel.Parent = game:GetService("CoreGui") end)
    if not okPanel then panel.Parent = LP:WaitForChild("PlayerGui") end

    local BTN_W, BTN_H = 64, 64
    local buttons = {}
    local buttonNames = {"DropBR", "AutoLeft", "AutoBat", "AutoRight", "TpDown", "Carry", "Lagger1", "Lagger2", "BatV2"}
    local buttonTexts = {"DROP\nBR", "AUTO\nLEFT", "BAT\nBYPASS", "AUTO\nRIGHT", "TP\nDOWN", "CARRY\nSPD", "LAGGER\nNORMAL", "LAGGER\nCARRY", "TP\nBATE"}

    local function createButton(name, text, order, isToggle, callback)
        local btn = kiraUI.new("TextButton", panel, {Name = name, Size = UDim2.fromOffset(BTN_W, BTN_H),
            BackgroundColor3 = Color3.fromRGB(255, 255, 255), BorderSizePixel = 0, Text = "",
            AutoButtonColor = false, Active = true, ZIndex = 10})
        local saved = savedButtonPositions[name]
        local x, y = getDefaultButtonPosition(name)
        btn.Position = UDim2.fromOffset(saved and tonumber(saved.X) or x, saved and tonumber(saved.Y) or y)
        kiraUI.round(btn, 11)
        kiraUI.new("UIGradient", btn, {Name = "BtnGrad", Rotation = 90,
            Color = ColorSequence.new(Color3.fromRGB(30, 34, 40), Color3.fromRGB(10, 14, 19))})
        kiraUI.stroke(btn, Color3.fromRGB(110, 94, 74), 0.54, 1)
        local dot = kiraUI.new("Frame", btn, {Name = "StatusDot", Size = UDim2.fromOffset(4, 4),
            Position = UDim2.new(1, -10, 0, 10), BackgroundColor3 = Color3.fromRGB(65, 72, 81), BorderSizePixel = 0, ZIndex = 11})
        kiraUI.round(dot, 2)
        local label = kiraUI.text(btn, text, UDim2.fromOffset(6, 10), UDim2.new(1, -12, 0, 37), 10,
            Color3.fromRGB(231, 228, 220), 11)
        label.Name = "TextLabel"
        label.Font = Enum.Font.GothamBold
        label.TextXAlignment = Enum.TextXAlignment.Center
        label.TextWrapped = true
        label.TextTruncate = Enum.TextTruncate.None
        local status = kiraUI.text(btn, "OFF", UDim2.fromOffset(6, 52), UDim2.new(1, -12, 0, 8), 6,
            Color3.fromRGB(126, 132, 139), 11)
        status.Name = "Status"
        status.TextXAlignment = Enum.TextXAlignment.Center
        local scale = kiraUI.new("UIScale", btn, {Scale = floatingButtonScale})
        table.insert(_floatingUIScales, scale)
        local active = false
        local function setActive(value)
            active = value == true
            btn:SetAttribute("MobActive", active)
            paintFloatingBtn(btn, active)
        end
        local pointer, dragStart, origin, moved = nil, nil, nil, 0
        local function clampPosition(pos)
            local camera = Workspace.CurrentCamera
            if not camera then return pos end
            local viewport = camera.ViewportSize
            local width = BTN_W * floatingButtonScale
            return UDim2.fromOffset(_clamp(pos.X.Offset, 6, math.max(6, viewport.X - width - 6)),
                _clamp(pos.Y.Offset, 6, math.max(6, viewport.Y - width - 6)))
        end
        local function finish(input)
            if not pointer then return end
            if pointer.UserInputType == Enum.UserInputType.Touch and input ~= pointer then return end
            if pointer.UserInputType == Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
            pointer = nil
            kiraUI.activeInput = nil
            _isDraggingButton = false
            TS:Create(scale, TweenInfo.new(0.12), {Scale = floatingButtonScale}):Play()
            if moved < 8 then
                local ok, err = pcall(callback, setActive, active)
                if not ok then warn("[kira Button] " .. name .. ": " .. tostring(err)) end
            elseif not uiLocked and not mobileButtonsLocked then
                savedButtonPositions[name] = {X = btn.Position.X.Offset, Y = btn.Position.Y.Offset}
            end
            paintFloatingBtn(btn, active)
            pcall(saveAllSettings)
        end
        btn.InputBegan:Connect(function(input)
            if kiraUI.activeInput then return end
            if input.UserInputType ~= Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.Touch then return end
            pointer, dragStart, origin, moved = input, input.Position, btn.Position, 0
            kiraUI.activeInput = input
            _isDraggingButton = true
            TS:Create(scale, TweenInfo.new(0.09), {Scale = floatingButtonScale * 0.95}):Play()
        end)
        local moveConn = UIS.InputChanged:Connect(function(input)
            if not pointer then return end
            if pointer.UserInputType == Enum.UserInputType.Touch then
                if input ~= pointer then return end
            elseif input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
            local delta = input.Position - dragStart
            moved = math.max(moved, delta.Magnitude)
            if moved >= 8 and not uiLocked and not mobileButtonsLocked then
                btn.Position = clampPosition(UDim2.fromOffset(origin.X.Offset + delta.X, origin.Y.Offset + delta.Y))
            end
        end)
        local endConn = UIS.InputEnded:Connect(finish)
        btn.InputEnded:Connect(finish)
        btn.Destroying:Connect(function()
            moveConn:Disconnect(); endConn:Disconnect()
            if pointer then kiraUI.activeInput = nil; _isDraggingButton = false end
        end)
        btn.Position = clampPosition(btn.Position)
        buttons[name] = {btn = btn, setActive = setActive, label = label, clampPosition = clampPosition}
        setActive(false)
        return setActive
    end

    for i, name in ipairs(buttonNames) do
        local text = buttonTexts[i]
        local callback
        if name == "DropBR" then
            callback = function(setActive)
                if autoBatEnabled then return end
                setActive(true)
                executeDropWithToggle(function(v)
                    if dropBrainrotSetVisual then dropBrainrotSetVisual(v) end
                end)
                task.delay(0.3, function() setActive(false) end)
            end
        elseif name == "AutoLeft" then
            callback = function(setActive)
                autoLeftEnabled = not autoLeftEnabled
                setActive(autoLeftEnabled)
                if autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
                if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
            end
        elseif name == "AutoBat" then
            callback = function(setActive)
                if not autoBatEnabled then enableAutoBat() else disableAutoBat() end
                setActive(autoBatEnabled)
            end
        elseif name == "AutoRight" then
            callback = function(setActive)
                autoRightEnabled = not autoRightEnabled
                setActive(autoRightEnabled)
                if autoRightEnabled then startAutoRight() else stopAutoRight() end
                if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
            end
        elseif name == "TpDown" then
            callback = function(setActive)
                doTpDown()
                setActive(true)
                task.delay(0.2, function() setActive(false) end)
            end
        elseif name == "Carry" then
            callback = function(setActive)
                if not speedMode then
                    speedMode = true; laggerToggled = false; laggerCarryToggled = false; setActive(true)
                    if buttons.Lagger1 and buttons.Lagger1.setActive then buttons.Lagger1.setActive(false) end
                    if buttons.Lagger2 and buttons.Lagger2.setActive then buttons.Lagger2.setActive(false) end
                else
                    speedMode = false; setActive(false)
                end
                refreshSpeedModeLabel()
                if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
            end
        elseif name == "Lagger1" then
            callback = function(setActive)
                if speedMode then speedMode = false; if mobSetCarry then mobSetCarry(false) end end
                if not laggerToggled then
                    laggerToggled = true; laggerCarryToggled = false; setActive(true)
                    if buttons.Lagger2 and buttons.Lagger2.setActive then buttons.Lagger2.setActive(false) end
                else
                    laggerToggled = false; setActive(false)
                end
                refreshSpeedModeLabel()
                if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
            end
        elseif name == "Lagger2" then
            callback = function(setActive)
                if speedMode then speedMode = false; if mobSetCarry then mobSetCarry(false) end end
                if not laggerCarryToggled then
                    laggerCarryToggled = true; laggerToggled = false; setActive(true)
                    if buttons.Lagger1 and buttons.Lagger1.setActive then buttons.Lagger1.setActive(false) end
                else
                    laggerCarryToggled = false; setActive(false)
                end
                refreshSpeedModeLabel()
                if _G.__RivalHubApplySpeedNow then pcall(_G.__RivalHubApplySpeedNow) end
            end
        elseif name == "BatV2" then
            callback = function(setActive)
                if not _G.__RivalHubIsBatV2() then
                    _G.__RivalHubStartBatV2()
                    setActive(true)
                else
                    _G.__RivalHubStopBatV2()
                    setActive(false)
                end
            end
        end
        local setActive = createButton(name, text, i-1, true, callback)
        if name == "AutoBat" then mobSetAutoBat = setActive end
        if name == "AutoLeft" then mobSetAutoLeft = setActive end
        if name == "AutoRight" then mobSetAutoRight = setActive end
        if name == "DropBR" then mobSetDropBR = setActive end
        if name == "TpDown" then mobSetTpDown = setActive end
        if name == "Carry" then mobSetCarry = setActive end
        if name == "Lagger1" then mobSetLagger1 = setActive end
        if name == "Lagger2" then mobSetLagger2 = setActive end
        if name == "BatV2" then mobSetBatV2 = setActive end
    end

    if buttons.AutoBat and buttons.AutoBat.setActive then buttons.AutoBat.setActive(autoBatEnabled) end
    if buttons.AutoLeft and buttons.AutoLeft.setActive then buttons.AutoLeft.setActive(autoLeftEnabled) end
    if buttons.AutoRight and buttons.AutoRight.setActive then buttons.AutoRight.setActive(autoRightEnabled) end
    if buttons.Carry and buttons.Carry.setActive then buttons.Carry.setActive(speedMode) end
    if buttons.Lagger1 and buttons.Lagger1.setActive then buttons.Lagger1.setActive(laggerToggled) end
    if buttons.Lagger2 and buttons.Lagger2.setActive then buttons.Lagger2.setActive(laggerCarryToggled) end
    if buttons.BatV2 and buttons.BatV2.setActive then
        buttons.BatV2.setActive(_G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2() or false)
    end

    local viewportConn
    if Workspace.CurrentCamera then
        viewportConn = Workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
            for _, entry in pairs(buttons) do entry.btn.Position = entry.clampPosition(entry.btn.Position) end
        end)
    end
    panel.Destroying:Connect(function() if viewportConn then viewportConn:Disconnect() end end)
    return panel
end

batCounterV2Enabled = false
batCounterV2Debounce = false
batCounterV2Conn = nil
batCounterV2HitCooldown = false
BAT_COUNTER_V2_SWING_CD = 0.08
setBatCounterV2Visual = nil

local _batCounterV2OriginalTpState = false
local _batCounterV2SuppressCount = 0
local _lastBatCounterV2Time = 0
local _batCounterV2Cooldown = 0.01

local function isPlayerRagdolled(player)
    if not player or not player.Character then return false end
    local hum = player.Character:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    local state = hum:GetState()
    return state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown
end

local function hasAnimalEquipped(player)
    if not player or not player.Character then return false end
    for _, child in ipairs(player.Character:GetChildren()) do
        if child:IsA("Tool") then
            if child:FindFirstChild("Handle") or child.Name:find("Animal") or child.Name:find("Pet") then return true end
        end
    end
    return false
end

local function getAttackingPlayerWithAnimal()
    local myChar = LP.Character
    if not myChar then return nil end
    local myRoot = myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end
    local closest, minDist = nil, _huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            if hasAnimalEquipped(plr) then
                local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - myRoot.Position).Magnitude
                if dist < 12 and dist < minDist then
                    minDist = dist; closest = plr
                end
            end
            end
        end
    end
    return closest
end

local function getBatV2Counter()
    local char = LP.Character
    if not char then return nil end
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Tool") then
            local name = child.Name:lower()
            if name:find("bat") or name:find("slap") or name:find("sword") or name:find("knife") then return child end
        end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, child in ipairs(bp:GetChildren()) do
            if child:IsA("Tool") then
                local name = child.Name:lower()
                if name:find("bat") or name:find("slap") or name:find("sword") or name:find("knife") then
                    child.Parent = char
                    return child
                end
            end
        end
    end
    if bp then
        for _, child in ipairs(bp:GetChildren()) do
            if child:IsA("Tool") then child.Parent = char; return child end
        end
    end
    return nil
end

local function tryHitBatCounterV2()
    if batCounterV2HitCooldown then return end
    batCounterV2HitCooldown = true
    pcall(function()
        local bat = getBatV2Counter()
        if bat then
            bat:Activate()
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then ev:FireServer() end
        end
    end)
    task.delay(BAT_COUNTER_V2_SWING_CD, function() batCounterV2HitCooldown = false end)
end

local function executeBatCounterV2()
    local now = _tick()
    if now - (_lastBatCounterV2Time or 0) < (_batCounterV2Cooldown or 0.01) then return end
    if batCounterV2Debounce then return end
    batCounterV2Debounce = true
    _lastBatCounterV2Time = now
    local attacker = getAttackingPlayerWithAnimal()
    if not attacker then batCounterV2Debounce = false; return end
    if isPlayerRagdolled(attacker) then batCounterV2Debounce = false; return end
    if not hasAnimalEquipped(attacker) then batCounterV2Debounce = false; return end
    local function doCounterHit()
        local char = LP.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local attChar = attacker.Character
        if not attChar then return end
        local attRoot = attChar:FindFirstChild("HumanoidRootPart")
        if not attRoot then return end
        if not hasAnimalEquipped(attacker) then batCounterV2Debounce = false; return end
        if sethiddenproperty then sethiddenproperty(hrp, "PhysicsRepRootPart", attRoot) end
        local targetPos = attRoot.Position + _V3new(0, 0.9, 0)
        if (hrp.Position - targetPos).Magnitude > 8 then hrp.CFrame = _CFnew(targetPos) end
        tryHitBatCounterV2()
        task.delay(0.05, function() tryHitBatCounterV2() end)
        task.delay(0.1, function() tryHitBatCounterV2() end)
    end
    doCounterHit()
    task.delay(0.2, function()
        batCounterV2Debounce = false
    end)
end

function stopBatCounterV2()
    if batCounterV2Conn then
        batCounterV2Conn:Disconnect()
        batCounterV2Conn = nil
    end
    batCounterV2Debounce = false
    batCounterV2HitCooldown = false
end

function startBatCounterV2()
    if batCounterV2Conn then return end
    batCounterV2Conn = RunService.Heartbeat:Connect(function()
        if not batCounterV2Enabled then return end
        if batCounterV2Debounce then return end
        local character = LP.Character
        if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return end
        local state = humanoid:GetState()
        local isBeingAttacked = state == Enum.HumanoidStateType.Physics or
                               state == Enum.HumanoidStateType.Ragdoll or
                               state == Enum.HumanoidStateType.FallingDown or
                               state == Enum.HumanoidStateType.GettingUp or
                               state == Enum.HumanoidStateType.Stunned
        if isBeingAttacked then
            local attacker = getAttackingPlayerWithAnimal()
            if attacker then
                if not isPlayerRagdolled(attacker) then
                    if hasAnimalEquipped(attacker) then
                        executeBatCounterV2()
                    end
                end
            end
        end
    end)
end

function toggleBatCounterV2()
    batCounterV2Enabled = not batCounterV2Enabled
    if batCounterV2Enabled then startBatCounterV2() else stopBatCounterV2() end
    return batCounterV2Enabled
end

function updateUIFromLoaded()
    task.wait()
    if normalBox then normalBox.Text = tostring(NS) end
    if carryBox then carryBox.Text = tostring(CS) end
    if radInput then radInput.Text = tostring(CONFIG.STEAL_RANGE) end
    if laggerBox then laggerBox.Text = tostring(LAGGER_SPEED) end
    if lagger2Box then lagger2Box.Text = tostring(LAGGER_CARRY_SPEED) end
    if batSpeedBox then batSpeedBox.Text = tostring(BYPASS_AIMBOT_SPEED) end
    if uiScaleBox then uiScaleBox.Text = tostring(uiScaleValue) end
    if dropModeBtnRef then dropModeBtnRef.Text = dropMode == 1 and "Fling" or "Jump Drop" end
    if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
    if infJumpModeBtn then infJumpModeBtn.Text = string.upper(InfiniteJump.mode == "manual" and "manual" or "hold") end
    if autoStealModeBtn then autoStealModeBtn.Text = autoStealMode end
    refreshSpeedModeLabel()

    local savedInfJumpEnabled = InfiniteJump.enabled == true
    InfiniteJump.stop()
    if savedInfJumpEnabled then
        InfiniteJump.start()
        if infJumpSetVisual then infJumpSetVisual(true) end
    elseif infJumpSetVisual then
        infJumpSetVisual(false)
    end

    for _, ref in ipairs(keyButtonRefs) do
        local entry = ref.entry
        local label = (entry.gp and entry.gp.Name) or (entry.kb and entry.kb.Name) or "None"
        ref.btn.Text = label
    end

    if savedProgressBarPos and pbFrame then
        pbFrame.Position = UDim2.new(savedProgressBarPos.XScale or 0.5, savedProgressBarPos.XOffset or -160, savedProgressBarPos.YScale or 0.80, savedProgressBarPos.YOffset or 0)
    end

    applyBackgroundMode(backgroundMode)
    applyFloatingButtonScale()
    if kiraUI.fit then kiraUI.fit() end
    if kiraUI.motorLabel then kiraUI.motorLabel.Text = useVenixCarryEngine and "Carry Boost" or "Direct" end
    if kiraUI.carrySetter then kiraUI.carrySetter(kiraVenixCarry.softStealEnabled) end
    if kiraUI.carrySpeedBox then kiraUI.carrySpeedBox.Text = tostring(kiraVenixCarry.softStealSpeed) end
    if kiraUI.carryRadiusBox then kiraUI.carryRadiusBox.Text = tostring(kiraVenixCarry.softStealRadius) end
    if kiraUI.effectsSetter then kiraUI.effectsSetter(kiraUI.effects) end
    if kiraUI.lockButtonsSetter then kiraUI.lockButtonsSetter(mobileButtonsLocked) end

    if uiLocked and setLockUIVisual then setLockUIVisual(true) end

    if antiRagdollMode == "v1" or antiRagdollMode == "v2" then
        if _G.updateAntiRagdollUI then _G.updateAntiRagdollUI(antiRagdollMode) end
        setAntiRagdollMode(antiRagdollMode)
    else
        if _G.updateAntiRagdollUI then _G.updateAntiRagdollUI("off") end
    end

    if antiDieEnabled then
        if setAntiDieVisual then setAntiDieVisual(true) end
        AntiDieModule.start()
    else
        if setAntiDieVisual then setAntiDieVisual(false) end
    end

    if antiBatEnabled then
        if setAntiBatVisual then setAntiBatVisual(true) end
        startAntiBat()
    else
        if setAntiBatVisual then setAntiBatVisual(false) end
    end

    if antiFlingEnabled then
        if setAntiFlingVisual then setAntiFlingVisual(true) end
        startAntiFling()
    else
        if setAntiFlingVisual then setAntiFlingVisual(false) end
        stopAntiFling()
    end

    if CONFIG.AUTO_STEAL_ENABLED and setInstaGrab then setInstaGrab(true); pcall(startAutoSteal) end
    if autoBatEnabled then autoBatEnabled = false; enableAutoBat() end

    if batCounterEnabled and setBatCounterVisual then
        setBatCounterVisual(true)
        startBatCounter()
    end
    if batCounterV2Enabled and setBatCounterV2Visual then
        setBatCounterV2Visual(true)
        startBatCounterV2()
    end
    if unwalkEnabled and setUnwalkVisual then
        setUnwalkVisual(true)
        task.spawn(function() task.wait(0.5); startUnwalk() end)
    end
    if antiLagEnabled then
        if setAntiLagVisual then setAntiLagVisual(true) end
        enableAntiLag()
    else
        if setAntiLagVisual then setAntiLagVisual(false) end
        disableAntiLag()
    end
    if espEnabled then
        toggleESP(true)
        if setESPVIsual then setESPVIsual(true) end
    else
        toggleESP(false)
        if setESPVIsual then setESPVIsual(false) end
    end
    if espLineEnabled then
        if setESPLineVisual then setESPLineVisual(true) end
        startESPLine()
    else
        if setESPLineVisual then setESPLineVisual(false) end
        stopESPLine()
    end

    if vividGraphicsEnabled then
        enableVividGraphics()
        if setVividVisual then setVividVisual(true) end
    else
        disableVividGraphics()
        if setVividVisual then setVividVisual(false) end
    end

    if stretchEnabled then
        enableStretch()
        if _G.stretchToggleSetter then _G.stretchToggleSetter(true) end
    else
        if _G.stretchToggleSetter then _G.stretchToggleSetter(false) end
    end

    if mobSetAutoBat then mobSetAutoBat(autoBatEnabled) end
    if mobSetAutoLeft then mobSetAutoLeft(autoLeftEnabled) end
    if mobSetAutoRight then mobSetAutoRight(autoRightEnabled) end
    if mobSetCarry then mobSetCarry(speedMode) end
    if mobSetLagger1 then mobSetLagger1(laggerToggled) end
    if mobSetLagger2 then mobSetLagger2(laggerCarryToggled) end
    if mobSetBatV2 then mobSetBatV2(_G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2() or false) end

    if bodyLockEnabled and bodyLockSetVisual then
        if _blSuppressCount == 0 then
            bodyLockSetVisual(true)
            startBodyLock()
        else
            bodyLockSetVisual(false)
        end
    end

    updateProgressBarVisibility()
    startEnemySpeed()

    toggleLockUI(uiLocked)

    pcall(function() applyOutfitByIndex(currentOutfitIndex) end)
    if outfitSelectorLabel and OUTFITS[currentOutfitIndex] then
        outfitSelectorLabel.Text = OUTFITS[currentOutfitIndex].label
    end
end

local _bootErrors = {}

local function _bootSection(name, fn)
    print("[Rival Hub Boot] Iniciando:", name)
    local ok, err = pcall(fn)
    if not ok then
        print("[Rival Hub Boot] FALLÓ:", name, tostring(err))
        table.insert(_bootErrors, {name = name, err = tostring(err)})
        warn("[Rival Hub Boot] Error en '" .. name .. "': " .. tostring(err))
    else
        print("[Rival Hub Boot] OK:", name)
    end
    return ok
end

_bootSection("buildGui", function()
    buildGui()
end)

if gui and main then
    _bootSection("loadAllSettings", function()
        if loadAllSettings() then
            updateUIFromLoaded()
        end
    end)

    _bootSection("createMobilePanel", function()
        MobilePanel = createMobilePanel()
    end)

    if hideButtonsEnabled then
        _bootSection("applyHideButtons", function()
            applyHideButtons(true)
            if setHideButtonsVisual then setHideButtonsVisual(true) end
        end)
    end

    _bootSection("localizeRivalHubInterface", function()
        if gui then localizeRivalHubInterface(gui) end
        if MobilePanel then localizeRivalHubInterface(MobilePanel) end
    end)

    if LP.Character then
        task.spawn(function()
            task.wait(0.1)
            if waitForCharReady(LP.Character, 5) then
                pcall(function() captureOriginalAppearance(LP.Character) end)
                pcall(function() setupMovementAndIndicators(LP.Character) end)
                if currentAnimPack ~= "Off" then
                    pcall(function() startAnimPack(currentAnimPack) end)
                end
                pcall(function() applyOutfitByIndex(currentOutfitIndex) end)
            end
        end)
    end

    if #_bootErrors > 0 then
        task.defer(function()
            pcall(function()
                local env = (getgenv and getgenv()) or _G
                local writeFn = env.writefile or (syn and syn.writefile)
                if type(writeFn) == "function" then
                    local log = "[Rival Hub Boot Errors] " .. os.date("%Y-%m-%d %H:%M:%S") .. "\n"
                    for _, e in ipairs(_bootErrors) do
                        log = log .. e.name .. ": " .. e.err .. "\n"
                    end
                    pcall(writeFn, "RivalHub_boot_errors.txt", log)
                    warn("[Rival Hub] Errores escritos a RivalHub_boot_errors.txt")
                end
            end)
        end)

        task.defer(function()
            pcall(function()
                local errGui = Instance.new("ScreenGui")
                errGui.Name = "RivalHubBootErrors"
                errGui.ResetOnSpawn = false
                errGui.DisplayOrder = 999
                local okCg, cg = pcall(function() return game:GetService("CoreGui") end)
                if okCg and cg then
                    pcall(function() errGui.Parent = cg end)
                end
                if not errGui.Parent then errGui.Parent = LP:WaitForChild("PlayerGui") end

                local frame = Instance.new("Frame", errGui)
                frame.Size = UDim2.new(0, 460, 0, 60 + #_bootErrors * 22)
                frame.Position = UDim2.new(0, 10, 1, -80 - #_bootErrors * 22)
                frame.BackgroundColor3 = Color3.fromRGB(40, 10, 15)
                frame.BorderSizePixel = 0
                Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 0)
                local stroke = Instance.new("UIStroke", frame)
                stroke.Color = Color3.fromRGB(255, 80, 90)
                stroke.Thickness = 1.5

                local title = Instance.new("TextLabel", frame)
                title.Size = UDim2.new(1, -20, 0, 24)
                title.Position = UDim2.new(0, 10, 0, 8)
                title.BackgroundTransparency = 1
                title.Text = "⚠ kira: " .. #_bootErrors .. " sección(es) fallaron"
                title.TextColor3 = Color3.fromRGB(255, 100, 110)
                title.Font = Enum.Font.GothamBold
                title.TextSize = 13
                title.TextXAlignment = Enum.TextXAlignment.Left

                for i, e in ipairs(_bootErrors) do
                    local lbl = Instance.new("TextLabel", frame)
                    lbl.Size = UDim2.new(1, -20, 0, 18)
                    lbl.Position = UDim2.new(0, 10, 0, 34 + (i - 1) * 20)
                    lbl.BackgroundTransparency = 1
                    lbl.Text = "• " .. e.name .. ": " .. e.err:sub(1, 110)
                    lbl.TextColor3 = Color3.fromRGB(255, 180, 180)
                    lbl.Font = Enum.Font.Code
                    lbl.TextSize = 10
                    lbl.TextXAlignment = Enum.TextXAlignment.Left
                    lbl.TextTruncate = Enum.TextTruncate.AtEnd
                end

                local copyBtn = Instance.new("TextButton", frame)
                copyBtn.Size = UDim2.new(0, 100, 0, 22)
                copyBtn.Position = UDim2.new(0, 10, 1, -30)
                copyBtn.BackgroundColor3 = Color3.fromRGB(80, 20, 30)
                copyBtn.Text = "COPIAR LOG"
                copyBtn.TextColor3 = Color3.fromRGB(255, 200, 200)
                copyBtn.Font = Enum.Font.GothamBold
                copyBtn.TextSize = 11
                copyBtn.BorderSizePixel = 0
                Instance.new("UICorner", copyBtn).CornerRadius = UDim.new(0, 0)
                copyBtn.MouseButton1Click:Connect(function()
                    local buf = ""
                    for _, e in ipairs(_bootErrors) do
                        buf = buf .. e.name .. ": " .. e.err .. "\n"
                    end
                    pcall(function()
                        if setclipboard then setclipboard(buf) end
                    end)
                    copyBtn.Text = "¡COPIADO!"
                    task.delay(1.2, function() copyBtn.Text = "COPIAR LOG" end)
                end)

                local closeBtn = Instance.new("TextButton", frame)
                closeBtn.Size = UDim2.new(0, 60, 0, 22)
                closeBtn.Position = UDim2.new(1, -70, 1, -30)
                closeBtn.BackgroundColor3 = Color3.fromRGB(80, 20, 30)
                closeBtn.Text = "CERRAR"
                closeBtn.TextColor3 = Color3.fromRGB(255, 200, 200)
                closeBtn.Font = Enum.Font.GothamBold
                closeBtn.TextSize = 11
                closeBtn.BorderSizePixel = 0
                Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 0)
                closeBtn.MouseButton1Click:Connect(function()
                    pcall(function() errGui:Destroy() end)
                end)

                task.delay(60, function()
                    pcall(function() errGui:Destroy() end)
                end)
            end)
        end)
    end
else
    warn("[Rival Hub Boot] Fatal: buildGui falló. El script no puede continuar.")
end

if LP and LP.CharacterAdded then
    pcall(function()
        local _respawnQueue = 0
        LP.CharacterAdded:Connect(function(char)
            pcall(function()
                _respawnQueue = _respawnQueue + 1
                local myId = _respawnQueue

                local resumeSteal = CONFIG.AUTO_STEAL_ENABLED
                stopAutoSteal()
                CONFIG.AUTO_STEAL_ENABLED = resumeSteal
                if stopAutoLeft then stopAutoLeft() end
                if stopAutoRight then stopAutoRight() end
                if stopBatCounter then stopBatCounter() end
                if stopBatCounterV2 then stopBatCounterV2() end
                if stopUnwalk then stopUnwalk() end
                if stopDropBrainrot then stopDropBrainrot() end
                local resumeBypass = autoBatEnabled
                if autoBatEnabled and disableAutoBat then disableAutoBat() end
                if bodyLockEnabled and stopBodyLock then stopBodyLock() end

                local deadline = _tick() + 5
                while (not char.Parent) or (not char:FindFirstChild("HumanoidRootPart")) or (not char:FindFirstChildOfClass("Humanoid")) do
                    if _tick() > deadline then return end
                    if myId ~= _respawnQueue then return end
                    task.wait(0.05)
                end

                _hookedVelParts = {}
                local _hrpRespawn = _setupVelChecked(char)
                _hookVelHRP(_hrpRespawn)

                if not _originalAppearance then task.defer(function() captureOriginalAppearance(char) end) end
                if setupMovementAndIndicators then setupMovementAndIndicators(char) end
                if resumeBypass then enableAutoBat() end
                if antiRagdollMode == "v1" then
                    AntiRagdollV1.start()
                elseif antiRagdollMode == "v2" then
                    startAntiRagdollV2()
                end
                if antiBatEnabled then startAntiBat() end
                if antiFlingEnabled then startAntiFling() end
                if AntiDieModule.enabled then task.defer(function() activateOnCharacter(char) end) end
                if CONFIG.AUTO_STEAL_ENABLED then pcall(startAutoSteal) end
                if bodyLockEnabled and _blSuppressCount == 0 then startBodyLock() end

                if batCounterEnabled then startBatCounter() end
                if batCounterV2Enabled then startBatCounterV2() end
                if unwalkEnabled then startUnwalk() end
                if currentAnimPack ~= "Off" then task.wait(0.3); startAnimPack(currentAnimPack) end

                updateProgressBarVisibility()
                refreshSpeedModeLabel()

                pcall(function() applyOutfitByIndex(currentOutfitIndex) end)
                if outfitSelectorLabel and OUTFITS[currentOutfitIndex] then
                    outfitSelectorLabel.Text = OUTFITS[currentOutfitIndex].label
                end
            end)
        end)
    end)
end

local lastLaggerToggle = 0
local LAGGER_COOLDOWN = 0.3

UIS.InputBegan:Connect(function(input, gpe)
    if _anyKeyListening then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if gpe or UIS:GetFocusedTextBox() then return end
    elseif not isGamepadInput(input) then
        return
    end
    if not isBindableInput(input) then return end

    local kc = input.KeyCode
    if not kc then return end

    if kbMatch(KB.LaggerMode, kc) then
        if _tick() - lastLaggerToggle >= LAGGER_COOLDOWN then
            lastLaggerToggle = _tick()
            toggleLaggerCycle()
        end
        return
    end
    if kbMatch(KB.CarryToggle, kc) then toggleCarryMode(); return end
    if kbMatch(KB.DropBrainrot, kc) then
        if not dropActive then
            if dropBrainrotSetVisual then dropBrainrotSetVisual(true) end
            executeDropWithToggle(dropBrainrotSetVisual)
        end
        return
    end
    if kbMatch(KB.TPFloor, kc) then doTpDown(); return end
    if kbMatch(KB.AutoLeft, kc) then
        autoLeftEnabled = not autoLeftEnabled
        if autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
        if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
        if mobSetAutoLeft then mobSetAutoLeft(autoLeftEnabled) end
        return
    end
    if kbMatch(KB.AutoRight, kc) then
        autoRightEnabled = not autoRightEnabled
        if autoRightEnabled then startAutoRight() else stopAutoRight() end
        if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
        if mobSetAutoRight then mobSetAutoRight(autoRightEnabled) end
        return
    end
    if kbMatch(KB.AutoBat, kc) then
        if not autoBatEnabled then
            enableAutoBat()
            if autoBatSetVisual then autoBatSetVisual(true) end
            if mobSetAutoBat then mobSetAutoBat(true) end
        else
            disableAutoBat()
            if autoBatSetVisual then autoBatSetVisual(false) end
            if mobSetAutoBat then mobSetAutoBat(false) end
        end
        return
    end
    if kbMatch(KB.BatV2, kc) then
        if _G.__RivalHubIsBatV2 and _G.__RivalHubIsBatV2() then
            if _G.__RivalHubStopBatV2 then pcall(_G.__RivalHubStopBatV2) end
            if mobSetBatV2 then mobSetBatV2(false) end
        else
            if _G.__RivalHubStartBatV2 then pcall(_G.__RivalHubStartBatV2) end
            if mobSetBatV2 then mobSetBatV2(true) end
        end
        return
    end
    if kbMatch(KB.GuiHide, kc) then
        if main then
            if main.Visible then hideGui() else showGui() end
        end
        return
    end
end)

do 
    local function isHoldingBrainrot()
        local char = LP and LP.Character
        if not char then return false end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and tonumber(hum.WalkSpeed) and hum.WalkSpeed < 25 then return true end
        for _, child in ipairs(char:GetChildren()) do
            local name = tostring(child.Name or ""):lower()
            if (child:IsA("Tool") or child:IsA("Model")) and
               (name:find("brainrot", 1, true) or name:find("animal", 1, true) or
                name:find("carry", 1, true) or name:find("grab", 1, true) or
                name:find("steal", 1, true) or name:find("hold", 1, true)) then
                return true
            end
        end
        return false
    end

end


do
    local SoundPlayers = game:GetService("Players")
    local SoundLP = SoundPlayers.LocalPlayer
    local SWORD_SOUND = "rbxassetid://5713085119"
    local hookedTools = setmetatable({}, {__mode = "k"})
    local customSounds = setmetatable({}, {__mode = "k"})

    local function isSwordTool(tool)
        if not tool or not tool:IsA("Tool") then return false end
        local name = tool.Name:lower()
        return name:find("bat", 1, true) ~= nil or name:find("slap", 1, true) ~= nil
            or name:find("sword", 1, true) ~= nil or name:find("knife", 1, true) ~= nil
            or name:find("espada", 1, true) ~= nil
    end

    local function makeCustomSound(parent)
        local old = customSounds[parent]
        if old and old.Parent then
            old.SoundId = SWORD_SOUND
            return old
        end
        local sound = Instance.new("Sound")
        sound.Name = "ShelCustomToolSound"
        sound.SoundId = SWORD_SOUND
        sound.Volume = 1
        sound.Looped = false
        sound.Parent = parent
        customSounds[parent] = sound
        return sound
    end

    local function playCustomSound(parent)
        if not parent or not parent.Parent then return end
        local sound = makeCustomSound(parent)
        pcall(function()
            sound:Stop()
            sound.TimePosition = 0.2
            sound.Volume = 1
            sound:Play()
        end)
    end

    local function hookTool(tool)
        if not isSwordTool(tool) or hookedTools[tool] then return end
        hookedTools[tool] = true
        local hookedSounds = setmetatable({}, {__mode = "k"})

        local function hookSound(sound)
            if not sound:IsA("Sound") or sound.Name == "ShelCustomToolSound" or hookedSounds[sound] then return end
            hookedSounds[sound] = true
            local triggering = false
            local function trigger()
                if triggering or not sound.Parent then return end
                if sound.Playing or sound.TimePosition > 0 then
                    triggering = true
                    pcall(function()
                        sound.Volume = 0
                        sound:Stop()
                    end)
                    playCustomSound(sound.Parent)
                    triggering = false
                end
            end
            sound:GetPropertyChangedSignal("Playing"):Connect(trigger)
            sound:GetPropertyChangedSignal("TimePosition"):Connect(trigger)
        end

        for _, descendant in ipairs(tool:GetDescendants()) do
            hookSound(descendant)
        end
        tool.DescendantAdded:Connect(hookSound)
    end

    local function watchContainer(container)
        if not container then return end
        for _, child in ipairs(container:GetChildren()) do
            if child:IsA("Tool") then task.defer(hookTool, child) end
        end
        if not container:GetAttribute("ShelToolSoundWatch") then
            container:SetAttribute("ShelToolSoundWatch", true)
            container.ChildAdded:Connect(function(child)
                if not child:IsA("Tool") then return end
                task.wait()
                hookTool(child)
            end)
        end
    end

    local function scanSoundTools()
        watchContainer(SoundLP:FindFirstChildOfClass("Backpack"))
        watchContainer(SoundLP.Character)
    end

    scanSoundTools()
    SoundLP.ChildAdded:Connect(function(child)
        if child:IsA("Backpack") then watchContainer(child) end
    end)
    SoundLP.CharacterAdded:Connect(function(character)
        task.wait(0.5)
        watchContainer(character)
        scanSoundTools()
    end)
    task.spawn(function()
        while task.wait(1) do scanSoundTools() end
    end)

    _G.ShelSwordSound = {
        apply = scanSoundTools,
        SwordSound = SWORD_SOUND,
    }
end

do
    local BombPlayers = game:GetService("Players")
    local BombLP = BombPlayers.LocalPlayer
    local bombHooked = setmetatable({}, {__mode = "k"})

    local function isBombTool(obj)
        if not obj or not obj:IsA("Tool") then return false end
        local n = obj.Name:lower()
        return n:find("medusa", 1, true) ~= nil or n:find("head", 1, true) ~= nil
            or n:find("stone", 1, true) ~= nil
    end

    local function hookBombTool(tool)
        if not isBombTool(tool) or bombHooked[tool] then return end
        bombHooked[tool] = true

        local function muteOriginal(obj)
            if not obj:IsA("Sound") or obj.Name == "ShelBombSound" then return end
            pcall(function()
                obj.Volume = 0
                obj:Stop()
            end)
            obj:GetPropertyChangedSignal("Volume"):Connect(function()
                if obj.Parent and obj.Volume ~= 0 then obj.Volume = 0 end
            end)
        end
        for _, obj in ipairs(tool:GetDescendants()) do
            muteOriginal(obj)
        end
        tool.DescendantAdded:Connect(muteOriginal)

        -- Som customizado da Medusa removido; os sons originais da ferramenta continuam silenciados.
    end

    local function watchBombContainer(container)
        if not container then return end
        for _, child in ipairs(container:GetChildren()) do
            if child:IsA("Tool") then task.defer(hookBombTool, child) end
        end
        if not container:GetAttribute("ShelBombSoundWatch") then
            container:SetAttribute("ShelBombSoundWatch", true)
            container.ChildAdded:Connect(function(child)
                if not child:IsA("Tool") then return end
                task.wait()
                hookBombTool(child)
            end)
        end
    end

    local function scanBombTools()
        watchBombContainer(BombLP:FindFirstChildOfClass("Backpack"))
        watchBombContainer(BombLP.Character)
    end

    scanBombTools()
    BombLP.ChildAdded:Connect(function(child)
        if child:IsA("Backpack") then watchBombContainer(child) end
    end)
    BombLP.CharacterAdded:Connect(function(character)
        task.wait(0.5)
        watchBombContainer(character)
        scanBombTools()
    end)
    task.spawn(function()
        while task.wait(1) do scanBombTools() end
    end)

    _G.ShelBombSound = {
        apply = scanBombTools,
    }
end

