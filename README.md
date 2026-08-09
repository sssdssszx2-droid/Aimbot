-- ==========================================
--  THAI ESP & AIMBOT HUB (EVO X UI VER)
--  ใช้ UI Library (UILibrary_base.lua) เป็นหน้าตา
--  ระบบเดิม (ESP / Aimbot / Anti-AFK / FPS HUD) คงเดิม 100%
-- ==========================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local VirtualUser = game:GetService("VirtualUser")
local Stats = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- ==========================================
--  ระบบหลังบ้าน: สุ่มชื่อ Instance (Anti-Detection)
-- ==========================================
local function GenerateRandomString(length)
    local str = ""
    for i = 1, length do
        str = str .. string.char(math.random(97, 122))
    end
    return str
end

local SafeGuiName = GenerateRandomString(12)

-- ==========================================
--  ค่าเริ่มต้น (Settings)
-- ==========================================
local Settings = {
    -- ESP Settings
    ESP_Enabled = true,
    Box = true,
    BoxColor = Color3.fromRGB(255, 50, 50),
    Tracer = true,
    TracerColor = Color3.fromRGB(255, 255, 255),
    Aura = true,
    AuraColor = Color3.fromRGB(138, 43, 226),
    Name = true,
    Health = true,
    TeamCheck = false,
    TextColor = Color3.fromRGB(255, 255, 255),

    -- Aimbot Settings
    Aimbot_Enabled = true,
    ShowFOV = true,
    FOVAtCenter = true,
    FOVSize = 130,
    Smoothness = 6,
    HumanizeAim = true,
    VisibleCheck = true,
    FOVColor = Color3.fromRGB(0, 255, 200),
    AimPart = "Head",

    -- Dynamic Keybinds
    AimKey = Enum.UserInputType.MouseButton2,
    AimKeyName = "MouseButton2",
    MenuKey = Enum.KeyCode.RightShift,
    MenuVisible = true,

    -- System & Utility Settings
    AntiAFK = true,
    ShowStatsHUD = true,
    FPS = 0,
    Ping = 0
}

local ColorPresets = {
    {Name = "🔴 แดง", Color = Color3.fromRGB(255, 50, 50)},
    {Name = "🟢 เขียว", Color = Color3.fromRGB(50, 255, 100)},
    {Name = "🔵 ฟ้า", Color = Color3.fromRGB(50, 150, 255)},
    {Name = "🟣 ม่วง", Color = Color3.fromRGB(180, 50, 255)},
    {Name = "🟡 เหลือง", Color = Color3.fromRGB(255, 220, 50)},
    {Name = "⚪ ขาว", Color = Color3.fromRGB(255, 255, 255)},
    {Name = "🟠 ส้ม", Color = Color3.fromRGB(255, 130, 0)}
}

local ESPData = {}
local isAiming = false

local cachedPlayerList = {}
local espFrame = 0
local LastAimScan = 0
local AimTarget = nil
local AimTargetCharacter = nil
local AimTargetPart = nil

-- ==========================================
--  ระบบ Anti-AFK (เด้งหลุดเมื่อพับจอ/ไม่ขยับตัว)
-- ==========================================
LocalPlayer.Idled:Connect(function()
    if Settings.AntiAFK then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

-- ============================================================
--  ล้าง UI Library (EVO X) — ถ้ายังไม่ได้รัน UILibrary_base.lua
--  สคริปต์นี้จะโหลดตัวในตัวให้อัตโนมัติ (กัน UI พัง)
-- ============================================================
if not (_G.EVOX_UI_Library or _G.UILibrary) then
    local function _InitEmbeddedUI()
        -- ============================================================
        --  EVO X  —  UI Library (Dark Panel)  [UILibrary_base.lua]
        --  ใช้เป็น UI พื้นฐานสำหรับทุกสคริปต์
        -- ============================================================
        local UIS = game:GetService("UserInputService")
        local TweenSer = game:GetService("TweenService")
        local P = game:GetService("Players")
        local LP = P.LocalPlayer

        local UILibrary = {}
        UILibrary.__index = UILibrary

        -- สีจาก CSS
        local C = {
            panel   = Color3.fromRGB(21, 24, 29),
            card    = Color3.fromRGB(26, 30, 36),
            card2   = Color3.fromRGB(24, 28, 33),
            border  = Color3.fromRGB(38, 43, 51),
            input   = Color3.fromRGB(33, 38, 46),
            sidebar = Color3.fromRGB(16, 19, 24),
            text    = Color3.fromRGB(232, 234, 237),
            sub     = Color3.fromRGB(139, 147, 169),
            muted   = Color3.fromRGB(90, 98, 112),
            accent  = Color3.fromRGB(91, 140, 255),
            accentSoft = Color3.fromRGB(96, 108, 136),
            toggleOff = Color3.fromRGB(51, 57, 68),
        }

        function UILibrary.new(config)
            config = config or {}
            local self = setmetatable({}, UILibrary)
            self.Title = config.Title or "EVO X"
            self.Visible = true
            self.CurrentTab = config.StartTab or "ESP"
            self.ToggleKey = config.ToggleKey or Enum.KeyCode.G

            self.ScreenGui = Instance.new("ScreenGui")
            self.ScreenGui.Name = config.Name or "EVO_X_Gui"
            self.ScreenGui.ResetOnSpawn = false

            pcall(function()
                for _, root in ipairs({game:GetService("CoreGui"), gethui and gethui() or nil}) do
                    if root then
                        local old = root:FindFirstChild(self.ScreenGui.Name)
                        if old then old:Destroy() end
                    end
                end
            end)

            local parented = false
            pcall(function()
                if gethui then
                    self.ScreenGui.Parent = gethui()
                    parented = true
                elseif syn and syn.protect_gui then
                    syn.protect_gui(self.ScreenGui)
                    self.ScreenGui.Parent = game:GetService("CoreGui")
                    parented = true
                else
                    self.ScreenGui.Parent = game:GetService("CoreGui")
                    parented = true
                end
            end)
            if not parented then
                pcall(function()
                    self.ScreenGui.Parent = LP:WaitForChild("PlayerGui")
                end)
            end

            self.baseSize = config.Size or UDim2.new(0, 470, 0, 560)
            self.basePosition = config.Position or UDim2.new(0.5, -235, 0.5, -280)

            self.MainFrame = Instance.new("Frame")
            self.MainFrame.Size = self.baseSize
            self.MainFrame.Position = self.basePosition
            self.MainFrame.BackgroundColor3 = C.panel
            self.MainFrame.BorderSizePixel = 0
            self.MainFrame.ClipsDescendants = true
            self.MainFrame.Parent = self.ScreenGui
            Instance.new("UICorner", self.MainFrame).CornerRadius = UDim.new(0, 14)
            local stroke = Instance.new("UIStroke")
            stroke.Color = C.border
            stroke.Thickness = 1
            stroke.Parent = self.MainFrame

            self.TitleBar = Instance.new("Frame")
            self.TitleBar.Size = UDim2.new(1, 0, 0, 42)
            self.TitleBar.BackgroundTransparency = 1
            self.TitleBar.ZIndex = 5
            self.TitleBar.Parent = self.MainFrame

            local dotColors = {Color3.fromRGB(255, 95, 87), Color3.fromRGB(254, 188, 46), Color3.fromRGB(40, 200, 64)}
            for i, col in ipairs(dotColors) do
                local d = Instance.new("Frame")
                d.Size = UDim2.new(0, 10, 0, 10)
                d.Position = UDim2.new(0, 14 + (i - 1) * 18, 0.5, -5)
                d.BackgroundColor3 = col
                d.ZIndex = 6
                d.Parent = self.TitleBar
                Instance.new("UICorner", d).CornerRadius = UDim.new(1, 0)
            end

            local titleText = Instance.new("TextLabel")
            titleText.Size = UDim2.new(0.7, 0, 1, 0)
            titleText.Position = UDim2.new(0, 70, 0, 0)
            titleText.BackgroundTransparency = 1
            titleText.Text = self.Title
            titleText.TextColor3 = C.text
            titleText.TextXAlignment = Enum.TextXAlignment.Left
            titleText.Font = Enum.Font.GothamBold
            titleText.TextSize = 14
            titleText.Parent = self.TitleBar

            self.StatLabel = Instance.new("TextLabel")
            self.StatLabel.Size = UDim2.new(0, 130, 0, 20)
            self.StatLabel.Position = UDim2.new(1, -140, 0.5, -10)
            self.StatLabel.BackgroundTransparency = 1
            self.StatLabel.Text = "FPS: -- | Ping: --"
            self.StatLabel.TextColor3 = C.muted
            self.StatLabel.Font = Enum.Font.GothamMedium
            self.StatLabel.TextSize = 10
            self.StatLabel.TextXAlignment = Enum.TextXAlignment.Right
            self.StatLabel.Parent = self.TitleBar

            local dragging, dragStart, startPos
            self.TitleBar.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = true
                    dragStart = input.Position
                    startPos = self.MainFrame.Position
                end
            end)
            UIS.InputChanged:Connect(function(input)
                if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                    local delta = input.Position - dragStart
                    self.MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                end
            end)
            UIS.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = false
                    self.basePosition = self.MainFrame.Position
                end
            end)

            UIS.InputBegan:Connect(function(input, gp)
                if gp then return end
                if input.KeyCode == self.ToggleKey then
                    self:ToggleVisibility()
                end
            end)

            self.Sidebar = Instance.new("Frame")
            self.Sidebar.Size = UDim2.new(0, 120, 1, -42)
            self.Sidebar.Position = UDim2.new(0, 0, 0, 42)
            self.Sidebar.BackgroundColor3 = C.sidebar
            self.Sidebar.BorderSizePixel = 0
            self.Sidebar.Parent = self.MainFrame

            self.ContentArea = Instance.new("Frame")
            self.ContentArea.Size = UDim2.new(1, -120, 1, -42)
            self.ContentArea.Position = UDim2.new(0, 120, 0, 42)
            self.ContentArea.BackgroundTransparency = 1
            self.ContentArea.Parent = self.MainFrame

            local brand = Instance.new("TextLabel")
            brand.Size = UDim2.new(1, -20, 0, 40)
            brand.Position = UDim2.new(0, 10, 0, 14)
            brand.BackgroundTransparency = 1
            brand.Text = "EVO X\nAimbot  v2.0"
            brand.TextColor3 = C.text
            brand.Font = Enum.Font.GothamBold
            brand.TextSize = 13
            brand.TextXAlignment = Enum.TextXAlignment.Left
            brand.TextYAlignment = Enum.TextYAlignment.Top
            brand.Parent = self.Sidebar

            local nav = Instance.new("Frame")
            nav.Size = UDim2.new(1, -16, 0, 190)
            nav.Position = UDim2.new(0, 8, 0, 70)
            nav.BackgroundTransparency = 1
            nav.Parent = self.Sidebar
            local navList = Instance.new("UIListLayout")
            navList.Padding = UDim.new(0, 6)
            navList.Parent = nav

            local footer = Instance.new("Frame")
            footer.Size = UDim2.new(1, -20, 0, 34)
            footer.Position = UDim2.new(0, 10, 1, -44)
            footer.BackgroundTransparency = 1
            footer.Parent = self.Sidebar
            local avatar = Instance.new("Frame")
            avatar.Size = UDim2.new(0, 26, 0, 26)
            avatar.Position = UDim2.new(0, 0, 0.5, -13)
            avatar.BackgroundColor3 = C.accent
            avatar.ClipsDescendants = true
            avatar.Parent = footer
            Instance.new("UICorner", avatar).CornerRadius = UDim.new(1, 0)
            local avatarImage = Instance.new("ImageLabel")
            avatarImage.Size = UDim2.new(1, 0, 1, 0)
            avatarImage.BackgroundTransparency = 1
            avatarImage.Image = "rbxasset://textures/ui/GuiImagePlaceholder.png"
            avatarImage.Parent = avatar
            task.spawn(function()
                pcall(function()
                    avatarImage.Image = P:GetUserThumbnailAsync(LP.UserId, Enum.ThumbnailType.AvatarBust, Enum.ThumbnailSize.Size420x420)
                end)
            end)
            local avName = Instance.new("TextLabel")
            avName.Size = UDim2.new(1, -34, 0, 14)
            avName.Position = UDim2.new(0, 32, 0, 2)
            avName.BackgroundTransparency = 1
            avName.Text = LP.DisplayName
            avName.TextColor3 = C.sub
            avName.Font = Enum.Font.GothamBold
            avName.TextSize = 11
            avName.TextXAlignment = Enum.TextXAlignment.Left
            avName.Parent = footer
            local avHandle = Instance.new("TextLabel")
            avHandle.Size = UDim2.new(1, -34, 0, 13)
            avHandle.Position = UDim2.new(0, 32, 0, 16)
            avHandle.BackgroundTransparency = 1
            avHandle.Text = "@" .. LP.Name .. "  •  " .. tostring(LP.UserId)
            avHandle.TextColor3 = C.muted
            avHandle.Font = Enum.Font.GothamBold
            avHandle.TextSize = 9
            avHandle.TextXAlignment = Enum.TextXAlignment.Left
            avHandle.Parent = footer

            self.Tabs = {}
            self.TabButtons = {}

            local tabList = config.Tabs or {
                {name = "ESP", icon = "◤"},
                {name = "Auto", icon = "⚡"},
                {name = "Emotes", icon = "♫"},
                {name = "Settings", icon = "◈"}
            }

            for _, tab in ipairs(tabList) do
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -8, 0, 38)
                btn.BackgroundTransparency = 1
                btn.Text = tab.icon .. "   " .. tab.name
                btn.TextColor3 = C.sub
                btn.Font = Enum.Font.GothamBold
                btn.TextSize = 12
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.TextStrokeTransparency = 1
                btn.Parent = nav
                Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

                local content = Instance.new("ScrollingFrame")
                content.Size = UDim2.new(1, -24, 1, -24)
                content.Position = UDim2.new(0, 12, 0, 12)
                content.BackgroundTransparency = 1
                content.BorderSizePixel = 0
                content.ScrollBarThickness = 3
                content.ScrollBarImageColor3 = C.accent
                content.CanvasSize = UDim2.new(0, 0, 0, 0)
                content.Visible = false
                content.Parent = self.ContentArea
                local pad = Instance.new("UIPadding")
                pad.PaddingTop = UDim.new(0, 6)
                pad.PaddingBottom = UDim.new(0, 6)
                pad.Parent = content
                local layout = Instance.new("UIListLayout")
                layout.SortOrder = Enum.SortOrder.LayoutOrder
                layout.Padding = UDim.new(0, 8)
                layout.Parent = content
                layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
                    content.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 12)
                end)

                self.Tabs[tab.name] = content
                self.TabButtons[tab.name] = btn

                btn.MouseButton1Click:Connect(function()
                    self:SwitchTab(tab.name)
                end)
            end

            self:SwitchTab(self.CurrentTab)
            return self
        end

        function UILibrary:SwitchTab(tabName)
            self.CurrentTab = tabName
            for name, content in pairs(self.Tabs) do
                content.Visible = (name == tabName)
                local btn = self.TabButtons[name]
                if btn then
                    if name == tabName then
                        btn.BackgroundColor3 = C.accentSoft
                        btn.TextColor3 = C.accent
                    else
                        btn.BackgroundTransparency = 1
                        btn.TextColor3 = C.sub
                    end
                end
            end
        end

        function UILibrary:ToggleVisibility()
            self.Visible = not self.Visible
            if self.Visible then
                self.MainFrame.Visible = true
                TweenSer:Create(self.MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    Size = self.baseSize,
                    Position = self.basePosition,
                }):Play()
                TweenSer:Create(self.MainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                    BackgroundTransparency = 0,
                }):Play()
            else
                TweenSer:Create(self.MainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
                    BackgroundTransparency = 1,
                }):Play()
                TweenSer:Create(self.MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
                    Size = UDim2.new(0, 0, 0, 0),
                    Position = UDim2.new(self.basePosition.X.Scale, self.basePosition.X.Offset, self.basePosition.Y.Scale, self.basePosition.Y.Offset + 40),
                }):Play()
            end
        end

        function UILibrary:NewCard(content, height)
            local card = Instance.new("Frame")
            card.Size = UDim2.new(1, 0, 0, height)
            card.BackgroundColor3 = C.card
            card.BorderSizePixel = 0
            card.Parent = content
            Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)
            return card
        end

        function UILibrary:AddToggle(tabName, labelText, subtitle, defaultState, callback)
            local content = self.Tabs[tabName]
            if not content then return end
            local card = self:NewCard(content, 46)

            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, -70, 0, 20)
            label.Position = UDim2.new(0, 14, 0, 8)
            label.BackgroundTransparency = 1
            label.Text = labelText
            label.TextColor3 = C.text
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Font = Enum.Font.GothamMedium
            label.TextSize = 13
            label.Parent = card

            if subtitle and subtitle ~= "" then
                local sub = Instance.new("TextLabel")
                sub.Size = UDim2.new(1, -70, 0, 14)
                sub.Position = UDim2.new(0, 14, 0, 27)
                sub.BackgroundTransparency = 1
                sub.Text = subtitle
                sub.TextColor3 = C.muted
                sub.TextXAlignment = Enum.TextXAlignment.Left
                sub.Font = Enum.Font.Gotham
                sub.TextSize = 10
                sub.Parent = card
            end

            local sw = Instance.new("Frame")
            sw.Size = UDim2.new(0, 44, 0, 24)
            sw.Position = UDim2.new(1, -16, 0.5, -12)
            sw.AnchorPoint = Vector2.new(1, 0)
            sw.BackgroundColor3 = C.toggleOff
            sw.BorderSizePixel = 0
            sw.Parent = card
            Instance.new("UICorner", sw).CornerRadius = UDim.new(1, 0)

            local knob = Instance.new("Frame")
            knob.Size = UDim2.new(0, 20, 0, 20)
            knob.Position = UDim2.new(0, 2, 0.5, -10)
            knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            knob.BorderSizePixel = 0
            knob.ZIndex = 3
            knob.Parent = sw
            Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

            local hit = Instance.new("TextButton")
            hit.Size = UDim2.new(1, 0, 1, 0)
            hit.BackgroundTransparency = 1
            hit.Text = ""
            hit.ZIndex = 4
            hit.Parent = sw

            local state = defaultState or false
            local function paint(s)
                local bg = s and C.accent or C.toggleOff
                local pos = s and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
                TweenSer:Create(sw, TweenInfo.new(0.2), {BackgroundColor3 = bg}):Play()
                TweenSer:Create(knob, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = pos}):Play()
            end
            hit.MouseButton1Click:Connect(function()
                state = not state
                paint(state)
                if callback then callback(state) end
            end)
            paint(state)
        end

        function UILibrary:AddButton(tabName, labelText, callback)
            local content = self.Tabs[tabName]
            if not content then return end
            local card = self:NewCard(content, 42)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -16, 1, -8)
            btn.Position = UDim2.new(0, 8, 0, 4)
            btn.BackgroundColor3 = C.input
            btn.BorderSizePixel = 0
            btn.Text = labelText
            btn.TextColor3 = C.text
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 12
            btn.Parent = card
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
            btn.MouseButton1Click:Connect(function()
                if callback then callback() end
            end)
            return btn
        end

        function UILibrary:AddEmoteCycler(tabName, list, onPlay)
            local content = self.Tabs[tabName]
            if not content or #list == 0 then return end

            local idx = 1
            local card = self:NewCard(content, 64)

            local main = Instance.new("TextButton")
            main.Size = UDim2.new(1, -108, 0, 46)
            main.Position = UDim2.new(0, 8, 0, 9)
            main.BackgroundColor3 = C.accent
            main.BorderSizePixel = 0
            main.TextColor3 = Color3.fromRGB(255, 255, 255)
            main.Font = Enum.Font.GothamBold
            main.TextSize = 12
            main.TextWrapped = true
            main.Parent = card
            Instance.new("UICorner", main).CornerRadius = UDim.new(0, 8)

            local prev = Instance.new("TextButton")
            prev.Size = UDim2.new(0, 36, 0, 46)
            prev.Position = UDim2.new(1, -100, 0, 9)
            prev.Text = "◀"
            prev.Font = Enum.Font.GothamBold
            prev.TextSize = 16
            prev.TextColor3 = C.sub
            prev.Parent = card
            prev.BackgroundTransparency = 1

            local nextBtn = Instance.new("TextButton")
            nextBtn.Size = UDim2.new(0, 36, 0, 46)
            nextBtn.Position = UDim2.new(1, -58, 0, 9)
            nextBtn.Text = "▶"
            nextBtn.Font = Enum.Font.GothamBold
            nextBtn.TextSize = 16
            nextBtn.TextColor3 = C.sub
            nextBtn.Parent = card
            nextBtn.BackgroundTransparency = 1

            local function update()
                local e = list[idx]
                main.Text = e.Name .. "   (" .. idx .. "/" .. #list .. ")"
            end
            prev.MouseButton1Click:Connect(function()
                idx = idx - 1; if idx < 1 then idx = #list end; update()
            end)
            nextBtn.MouseButton1Click:Connect(function()
                idx = idx + 1; if idx > #list then idx = 1 end; update()
            end)
            main.MouseButton1Click:Connect(function()
                if onPlay then onPlay(idx, list[idx]) end
            end)
            update()
        end

        function UILibrary:AddSlider(tabName, labelText, subtitle, min, max, default, callback)
            local content = self.Tabs[tabName]
            if not content then return end
            local card = self:NewCard(content, 64)

            local title = Instance.new("TextLabel")
            title.Size = UDim2.new(1, -90, 0, 20)
            title.Position = UDim2.new(0, 14, 0, 8)
            title.BackgroundTransparency = 1
            title.Text = labelText
            title.TextColor3 = C.text
            title.TextXAlignment = Enum.TextXAlignment.Left
            title.Font = Enum.Font.GothamMedium
            title.TextSize = 13
            title.Parent = card

            if subtitle and subtitle ~= "" then
                local sub = Instance.new("TextLabel")
                sub.Size = UDim2.new(1, -90, 0, 14)
                sub.Position = UDim2.new(0, 14, 0, 27)
                sub.BackgroundTransparency = 1
                sub.Text = subtitle
                sub.TextColor3 = C.muted
                sub.TextXAlignment = Enum.TextXAlignment.Left
                sub.Font = Enum.Font.Gotham
                sub.TextSize = 10
                sub.Parent = card
            end

            local valueBox = Instance.new("TextLabel")
            valueBox.Size = UDim2.new(0, 44, 0, 24)
            valueBox.Position = UDim2.new(1, -16, 0, 8)
            valueBox.AnchorPoint = Vector2.new(1, 0)
            valueBox.BackgroundColor3 = C.input
            valueBox.BorderSizePixel = 0
            valueBox.TextColor3 = C.sub
            valueBox.Font = Enum.Font.GothamBold
            valueBox.TextSize = 11
            valueBox.Parent = card
            Instance.new("UICorner", valueBox).CornerRadius = UDim.new(0, 6)

            local track = Instance.new("Frame")
            track.Size = UDim2.new(1, -90, 0, 4)
            track.Position = UDim2.new(0, 14, 0, 52)
            track.BackgroundColor3 = C.toggleOff
            track.BorderSizePixel = 0
            track.Parent = card
            Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

            local thumb = Instance.new("TextButton")
            thumb.Size = UDim2.new(0, 16, 0, 16)
            thumb.Position = UDim2.new(0, 0, 0, -6)
            thumb.BackgroundColor3 = C.accent
            thumb.Text = ""
            thumb.Parent = track
            Instance.new("UICorner", thumb).CornerRadius = UDim.new(1, 0)

            local value = default or min
            local function paint()
                local pct = (value - min) / (max - min)
                thumb.Position = UDim2.new(math.clamp(pct, 0, 1), 0, 0, -6)
                valueBox.Text = string.format("%.2f", value)
            end
            thumb.MouseButton1Down:Connect(function()
                local conn; local last = value
                conn = UIS.InputChanged:Connect(function(inp)
                    if inp.UserInputType ~= Enum.UserInputType.MouseMovement then return end
                    local pos = track.AbsolutePosition
                    local wid = track.AbsoluteSize.X
                    local x = inp.Position.X - pos.X
                    local pct = math.clamp(x / wid, 0, 1)
                    value = min + (max - min) * pct
                    value = math.floor(value * 100) / 100
                    paint()
                    if math.abs(last - value) > 0.001 then
                        last = value
                        if callback then callback(value) end
                    end
                end)
                local endc
                endc = UIS.InputEnded:Connect(function(inp)
                    if inp.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
                    conn:Disconnect(); endc:Disconnect()
                end)
            end)
            paint()
        end

        function UILibrary:AddKeybind(tabName, labelText, currentKey, callback)
            local content = self.Tabs[tabName]
            if not content then return end
            local card = self:NewCard(content, 42)

            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, -120, 1, 0)
            label.Position = UDim2.new(0, 14, 0, 0)
            label.BackgroundTransparency = 1
            label.Text = labelText
            label.TextColor3 = C.text
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Font = Enum.Font.GothamMedium
            label.TextSize = 13
            label.Parent = card

            local keyBtn = Instance.new("TextButton")
            keyBtn.Size = UDim2.new(0, 96, 0, 26)
            keyBtn.Position = UDim2.new(1, -16, 0, 8)
            keyBtn.AnchorPoint = Vector2.new(1, 0)
            keyBtn.BackgroundColor3 = C.input
            keyBtn.BorderSizePixel = 0
            keyBtn.Text = "[" .. currentKey.Name .. "]"
            keyBtn.TextColor3 = C.sub
            keyBtn.Font = Enum.Font.GothamBold
            keyBtn.TextSize = 12
            keyBtn.Parent = card
            Instance.new("UICorner", keyBtn).CornerRadius = UDim.new(0, 6)

            local listening = false
            keyBtn.MouseButton1Click:Connect(function()
                if listening then return end
                listening = true
                keyBtn.Text = "[...]"
                keyBtn.BackgroundColor3 = C.accent
                local conn
                conn = UIS.InputBegan:Connect(function(input, gp)
                    if gp then return end
                    if input.UserInputType == Enum.UserInputType.Keyboard then
                        conn:Disconnect()
                        listening = false
                        keyBtn.Text = "[" .. input.KeyCode.Name .. "]"
                        keyBtn.BackgroundColor3 = C.input
                        if callback then callback(input.KeyCode) end
                    end
                end)
            end)
        end

        _G.EVOX_UI_Library = UILibrary
        _G.UILibrary = UILibrary
        return UILibrary
    end

    _InitEmbeddedUI()
end

local UILibrary = _G.EVOX_UI_Library or _G.UILibrary
if not UILibrary then
    error("ยังโหลด UI Library ไม่ได้ กรุณารันไฟล์ UILibrary_base.lua ก่อน แล้วค่อยรันสคริปต์นี้")
end

-- ============================================================
--  ส่วนขยาย UI Library: ตัวเลือกสี (Color Picker) แบบ EVO X
-- ============================================================
local function ClosestColorIndex(presets, color)
    local best, bestDist = 1, math.huge
    for i, p in ipairs(presets) do
        local c = p.Color
        local d = (c.R - color.R)^2 + (c.G - color.G)^2 + (c.B - color.B)^2
        if d < bestDist then
            bestDist, best = d, i
        end
    end
    return best
end

if not UILibrary.AddColorPicker then
    function UILibrary:AddColorPicker(tabName, labelText, presets, defaultColor, callback)
        local content = self.Tabs[tabName]
        if not content then return end
        local card = self:NewCard(content, 54)

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -84, 0, 20)
        label.Position = UDim2.new(0, 14, 0, 8)
        label.BackgroundTransparency = 1
        label.Text = labelText
        label.TextColor3 = Color3.fromRGB(232, 234, 237)
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Font = Enum.Font.GothamMedium
        label.TextSize = 13
        label.Parent = card

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, -84, 0, 14)
        nameLabel.Position = UDim2.new(0, 14, 0, 28)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = ""
        nameLabel.TextColor3 = Color3.fromRGB(139, 147, 169)
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Font = Enum.Font.Gotham
        nameLabel.TextSize = 10
        nameLabel.Parent = card

        local swatch = Instance.new("TextButton")
        swatch.Size = UDim2.new(0, 36, 0, 28)
        swatch.Position = UDim2.new(1, -14, 0.5, 0)
        swatch.AnchorPoint = Vector2.new(1, 0.5)
        swatch.Text = ""
        swatch.BorderSizePixel = 0
        swatch.BackgroundColor3 = Color3.fromRGB(51, 57, 68)
        swatch.Parent = card
        Instance.new("UICorner", swatch).CornerRadius = UDim.new(0, 6)

        local inner = Instance.new("Frame")
        inner.Size = UDim2.new(0, 18, 0, 18)
        inner.Position = UDim2.new(0.5, -9, 0.5, -9)
        inner.BorderSizePixel = 0
        inner.Parent = swatch
        Instance.new("UICorner", inner).CornerRadius = UDim.new(1, 0)

        local idx = ClosestColorIndex(presets, defaultColor or presets[1].Color)
        local function paint()
            local p = presets[idx]
            inner.BackgroundColor3 = p.Color
            nameLabel.Text = "สี : " .. p.Name
            nameLabel.TextColor3 = p.Color
        end
        swatch.MouseButton1Click:Connect(function()
            idx = idx % #presets + 1
            paint()
            if callback then callback(presets[idx].Color) end
        end)
        paint()
        if callback then callback(presets[idx].Color) end
    end
end

-- ============================================================
--  สร้าง UI หลัก (EVO X Library)
-- ============================================================
local UI = UILibrary.new({
    Name = "EVO_X_Aimbot_Gui",
    Title = "🛡️ THAI HUB (FULL ADVANCED)",
    Position = UDim2.new(0.05, 20, 0, 40),
    ToggleKey = Settings.MenuKey,
    StartTab = "ESP",
    Tabs = {
        {name = "ESP", icon = "👁"},
        {name = "Aimbot", icon = "🎯"},
        {name = "Settings", icon = "⚙️"},
    },
})

-- ============================================================
--  สร้างเมนูในแต่ละแท็บ
-- ============================================================

-- --- TAB ESP ---
UI:AddToggle("ESP", "เปิด/ปิด ESP หลัก", "แสดงข้อมูลทั้งหมด", Settings.ESP_Enabled, function(state)
    Settings.ESP_Enabled = state
end)

UI:AddToggle("ESP", "กรอบสี่เหลี่ยม (Box)", "กล่อง/ชุดรอบผู้เล่น", Settings.Box, function(state)
    Settings.Box = state
end)

UI:AddColorPicker("ESP", "🎨 สีของ Box", ColorPresets, Settings.BoxColor, function(color)
    Settings.BoxColor = color
end)

UI:AddToggle("ESP", "เส้นเชื่อม (Tracer)", "ลากเส้นจากจุดกลางจอ", Settings.Tracer, function(state)
    Settings.Tracer = state
end)

UI:AddColorPicker("ESP", "🎨 สีของ Tracer", ColorPresets, Settings.TracerColor, function(color)
    Settings.TracerColor = color
end)

UI:AddToggle("ESP", "ออร่าทะลุกำแพง (Aura)", "ไฮไลท์ผู้เล่นทะลุกำแพง", Settings.Aura, function(state)
    Settings.Aura = state
end)

UI:AddColorPicker("ESP", "🎨 สีของ Aura", ColorPresets, Settings.AuraColor, function(color)
    Settings.AuraColor = color
end)

UI:AddToggle("ESP", "แสดงชื่อผู้เล่น (Name)", "แสดงชื่อบนหัว", Settings.Name, function(state)
    Settings.Name = state
end)

UI:AddToggle("ESP", "แสดงหลอดเลือด (Health)", "แสดงแถบ HP เหนือหัว", Settings.Health, function(state)
    Settings.Health = state
end)

UI:AddToggle("ESP", "ไม่แสดงพวกเดียวกัน (Team Check)", "กรองทีมเดียวกัน", Settings.TeamCheck, function(state)
    Settings.TeamCheck = state
end)

-- --- TAB: AIMBOT ---
UI:AddToggle("Aimbot", "เปิด/ปิด Aimbot", "เล็งอัตโนมัติในรัศมี FOV", Settings.Aimbot_Enabled, function(state)
    Settings.Aimbot_Enabled = state
end)

UI:AddToggle("Aimbot", "วงกลมอยู่กลางจอ", "ปิด = ตามเมาท์", Settings.FOVAtCenter, function(state)
    Settings.FOVAtCenter = state
end)

UI:AddToggle("Aimbot", "แสดงวงกลม FOV", "แสดงขอบเขตเล็ง", Settings.ShowFOV, function(state)
    Settings.ShowFOV = state
end)

UI:AddSlider("Aimbot", "ขนาดรัศมี FOV", "เล็ก-ใหญ่ สำหรับล็อกเป้า", 30, 400, Settings.FOVSize, function(v)
    Settings.FOVSize = math.floor(v)
end)

UI:AddSlider("Aimbot", "ความสมูท (Smoothness)", "1 = แรงสุด", 1, 20, Settings.Smoothness, function(v)
    Settings.Smoothness = math.floor(v)
end)

UI:AddToggle("Aimbot", "เล็งเนียนแบบมนุษย์ (Humanize)", "เล็งด้วยการจุ่มเล็กน้อย", Settings.HumanizeAim, function(state)
    Settings.HumanizeAim = state
end)

UI:AddToggle("Aimbot", "เช็คสิ่งกีดขวาง (Visible Check)", "เล็งเฉพาะเป้าที่เห็น", Settings.VisibleCheck, function(state)
    Settings.VisibleCheck = state
end)

UI:AddColorPicker("Aimbot", "🎨 สีวงกลม FOV", ColorPresets, Settings.FOVColor, function(color)
    Settings.FOVColor = color
end)

-- ปุ่มตั้งค่า Keybind ล็อกเป้า (รองรับทั้งเมาส์ และคีย์บอร์ด)
local isBindingAim = false
local AimBindBtn = nil
local function UpdateAimBindBtn()
    if AimBindBtn then
        AimBindBtn.Text = "ปุ่มล็อกเป้า : [ " .. Settings.AimKeyName .. " ]  (คลิกเพื่อเปลี่ยน)"
        AimBindBtn.TextColor3 = Color3.fromRGB(232, 234, 237)
    end
end
do
    local content = UI.Tabs["Aimbot"]
    local card = UILibrary.NewCard(UI, content, 44)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -16, 1, -8)
    btn.Position = UDim2.new(0, 8, 0, 4)
    btn.BackgroundColor3 = Color3.fromRGB(33, 38, 46)
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.GothamMedium
    btn.TextSize = 12
    btn.TextWrapped = true
    btn.TextColor3 = Color3.fromRGB(232, 234, 237)
    btn.Parent = card
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    btn.MouseButton1Click:Connect(function()
        if isBindingAim then return end
        isBindingAim = true
        btn.Text = "กดปุ่มคีย์บอร์ดหรือเมาส์ขวาที่ต้องการ..."
        btn.TextColor3 = Color3.fromRGB(91, 140, 255)
    end)
    AimBindBtn = btn
    UpdateAimBindBtn()
end

-- --- TAB: SETTINGS ---
UI:AddKeybind("Settings", "ปุ่มเปิด/ปิดเมนู", Settings.MenuKey, function(newKey)
    Settings.MenuKey = newKey
    UI.ToggleKey = newKey
end)

UI:AddToggle("Settings", "ระบบ Anti-AFK", "กันเด้งหลุด", Settings.AntiAFK, function(state)
    Settings.AntiAFK = state
end)

UI:AddToggle("Settings", "แสดง FPS & Ping บนจอ (HUD)", "แสดง Widget ตัวเลขบนจอ", Settings.ShowStatsHUD, function(state)
    Settings.ShowStatsHUD = state
end)

-- ============================================================
--  HUD แสดง FPS & Ping บนจอ (Overlay Widget)
-- ============================================================
local StatsHUD = Instance.new("Frame")
StatsHUD.Name = GenerateRandomString(6)
StatsHUD.Size = UDim2.new(0, 160, 0, 30)
StatsHUD.Position = UDim2.new(0.82, 0, 0.02, 0)
StatsHUD.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
StatsHUD.BorderSizePixel = 0
StatsHUD.Active = true
StatsHUD.Parent = UI.ScreenGui

local HUDCorner = Instance.new("UICorner")
HUDCorner.CornerRadius = UDim.new(0, 6)
HUDCorner.Parent = StatsHUD

local HUDStroke = Instance.new("UIStroke")
HUDStroke.Color = Color3.fromRGB(60, 60, 90)
HUDStroke.Thickness = 1
HUDStroke.Parent = StatsHUD

local HUDText = Instance.new("TextLabel")
HUDText.Size = UDim2.new(1, 0, 1, 0)
HUDText.BackgroundTransparency = 1
HUDText.Text = "FPS: -- | Ping: -- ms"
HUDText.TextColor3 = Color3.fromRGB(0, 255, 200)
HUDText.Font = Enum.Font.GothamBold
HUDText.TextSize = 13
HUDText.Parent = StatsHUD

local HUDDragging, HUDDragStart, HUDStartPos
StatsHUD.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        HUDDragging = true
        HUDDragStart = input.Position
        HUDStartPos = StatsHUD.Position
    end
end)
StatsHUD.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        HUDDragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if HUDDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - HUDDragStart
        StatsHUD.Position = UDim2.new(HUDStartPos.X.Scale, HUDStartPos.X.Offset + delta.X, HUDStartPos.Y.Scale, HUDStartPos.Y.Offset + delta.Y)
    end
end)

-- ============================================================
--  ระบบคำนวณ FPS & Ping Real-time (แสดงใน UI + HUD)
-- ============================================================
local frameCount = 0
local lastFPSTick = tick()

task.spawn(function()
    local updater = RunService.RenderStepped:Connect(function(dt)
        frameCount = frameCount + 1
        if tick() - lastFPSTick >= 1 then
            lastFPSTick = tick()

            pcall(function()
                Settings.Ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            end)

            local fps = frameCount
            frameCount = 0
            Settings.FPS = fps

            local statString = string.format("FPS: %d | Ping: %d ms", Settings.FPS, Settings.Ping)
            UI.StatLabel.Text = statString
            HUDText.Text = statString
        end

        StatsHUD.Visible = Settings.ShowStatsHUD
    end)
end)

-- ============================================================
--  ระบบตรวจจับ Keybind (ล็อกเป้า + ปุ่ม UI)
-- ============================================================
UserInputService.InputBegan:Connect(function(input, gpe)
    -- โหมดตั้งปุ่มล็อกเป้า (รองรับเมาส์ขวา/ซ้าย และคีย์บอร์ด)
    if isBindingAim then
        local isKeyboard = input.UserInputType == Enum.UserInputType.Keyboard
        local isMouseBtn = (input.UserInputType == Enum.UserInputType.MouseButton2) or (input.UserInputType == Enum.UserInputType.MouseButton3)
        if isKeyboard or isMouseBtn then
            if isKeyboard then
                Settings.AimKey = input.KeyCode
                Settings.AimKeyName = input.KeyCode.Name
            else
                Settings.AimKey = input.UserInputType
                Settings.AimKeyName = input.UserInputType.Name
            end
            isBindingAim = false
            UpdateAimBindBtn()
        end
        return
    end

    if not gpe then
        if (input.UserInputType == Settings.AimKey) or (input.KeyCode == Settings.AimKey) then
            isAiming = true
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if (input.UserInputType == Settings.AimKey) or (input.KeyCode == Settings.AimKey) then
        isAiming = false
    end
end)

-- ============================================================
--  ระบบ AIMBOT Core & Safety Check
-- ============================================================
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 64
FOVCircle.Filled = false

local function IsVisible(targetPart)
    if not Settings.VisibleCheck then return true end
    local ignoreList = {LocalPlayer.Character, targetPart.Parent}
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = ignoreList

    local ray = Workspace:Raycast(Camera.CFrame.Position, targetPart.Position - Camera.CFrame.Position, params)
    return ray == nil
end

local function GetClosestPlayer(originPos)
    local closestPlayer = nil
    local shortestDist = Settings.FOVSize

for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(Settings.AimPart)
            and not (Settings.TeamCheck and player.Team == LocalPlayer.Team) then

            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local part = player.Character[Settings.AimPart]
                local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)

                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - originPos).Magnitude
                    if dist < shortestDist then
                        if IsVisible(part) then
                            shortestDist = dist
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- ============================================================
--  ระบบ ESP Core Logic
-- ============================================================
local function CreateESP(player)
    if player == LocalPlayer then return end

    local objects = {
        Box = Drawing.new("Square"),
        Tracer = Drawing.new("Line"),
        AuraCircle = Drawing.new("Circle"),
        NameDraw = Drawing.new("Text"),
        HPBackDraw = Drawing.new("Square"),
        HPFillDraw = Drawing.new("Square")
    }

    objects.Box.Visible = false
    objects.Box.Thickness = 1.5
    objects.Box.Filled = false

    objects.Tracer.Visible = false
    objects.Tracer.Thickness = 1.5

    objects.AuraCircle.Visible = false
    objects.AuraCircle.Thickness = 2
    objects.AuraCircle.NumSides = 48
    objects.AuraCircle.Filled = false

    objects.NameDraw.Visible = false
    objects.NameDraw.Color = Settings.TextColor
    objects.NameDraw.Outline = true
    objects.NameDraw.OutlineColor = Color3.new(0, 0, 0)
    objects.NameDraw.Size = 13
    objects.NameDraw.Center = true
    objects.NameDraw.Font = 2
    objects.NameDraw.Text = ""

    objects.HPBackDraw.Visible = false
    objects.HPBackDraw.Filled = true
    objects.HPBackDraw.Color = Color3.fromRGB(25, 25, 25)
    objects.HPBackDraw.Transparency = 0.3
    objects.HPBackDraw.Thickness = 1

    objects.HPFillDraw.Visible = false
    objects.HPFillDraw.Filled = true
    objects.HPFillDraw.Thickness = 0

    ESPData[player] = objects
end

local function RemoveESP(player)
    if ESPData[player] then
        local objs = ESPData[player]
        pcall(function() objs.Box:Remove() end)
        pcall(function() objs.Tracer:Remove() end)
        pcall(function() objs.NameDraw:Remove() end)
        pcall(function() objs.HPBackDraw:Remove() end)
        pcall(function() objs.HPFillDraw:Remove() end)
        if objs.AuraCircle then pcall(function() objs.AuraCircle:Remove() end) end
        ESPData[player] = nil
    end
end

-- ============================================================
--  Render Loop หลัก (Optimized: Caching + Throttling)
--  - Aimbot หาเป้าหมายแค่ทุก 0.1s (แทนทุกเฟรม)
--  - ESP วาดทุก 2 เฟรม
--  - ใช้ PlayerList แบบ cache
-- ============================================================
RunService.RenderStepped:Connect(function()
    local fovCenterPos
    if Settings.FOVAtCenter then
        fovCenterPos = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    else
        fovCenterPos = UserInputService:GetMouseLocation()
    end

    FOVCircle.Visible = Settings.Aimbot_Enabled and Settings.ShowFOV
    FOVCircle.Radius = Settings.FOVSize
    FOVCircle.Position = fovCenterPos
    FOVCircle.Color = Settings.FOVColor

    if Settings.Aimbot_Enabled and isAiming then
        local now = tick()
        if now - LastAimScan >= 0.1 then
            LastAimScan = now
            AimTarget = GetClosestPlayer(fovCenterPos)
            if AimTarget then
                AimTargetCharacter = AimTarget.Character
                AimTargetPart = AimTargetCharacter and AimTargetCharacter:FindFirstChild(Settings.AimPart)
            else
                AimTargetCharacter, AimTargetPart = nil, nil
            end
        end

        if AimTarget and AimTargetCharacter and AimTargetPart then
            local targetPos = AimTargetPart.Position

            if Settings.HumanizeAim then
                local jitter = Vector3.new(
                    (math.random(-10, 10) / 100),
                    (math.random(-10, 10) / 100),
                    (math.random(-10, 10) / 100)
                )
                targetPos = targetPos + jitter
            end

            local currentCFrame = Camera.CFrame
            local smoothVal = Settings.Smoothness
            if Settings.HumanizeAim then
                smoothVal = smoothVal + (math.random(-5, 5) / 10)
            end

            local alpha = math.clamp(1 / math.max(smoothVal, 1), 0.01, 1)
            Camera.CFrame = currentCFrame:Lerp(CFrame.new(currentCFrame.Position, targetPos), alpha)
        end
    end

    -- ===== ESP (วาดทุก 2 เฟรม เพื่อลดโหลด) =====
    espFrame = espFrame + 1
    local drawEsp = espFrame % 2 == 0

    if drawEsp then
        for _, player in ipairs(cachedPlayerList) do
            local objs = ESPData[player]
            if objs then

            local character = player.Character
            local hrp = character and character:FindFirstChild("HumanoidRootPart")
            local head = character and character:FindFirstChild("Head")
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")
            local isSameTeam = Settings.TeamCheck and (player.Team == LocalPlayer.Team)

            if Settings.ESP_Enabled and not isSameTeam and character and hrp and head and humanoid and humanoid.Health > 0 then
                local topCenter = Vector2.new(Camera.ViewportSize.X / 2, 0)
                local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                if onScreen then
                    local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.6, 0))
                    local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 1.5

                    if Settings.Box then
                        objs.Box.Size = Vector2.new(width, height)
                        objs.Box.Position = Vector2.new(hrpPos.X - width / 2, hrpPos.Y - height / 2)
                        objs.Box.Color = Settings.BoxColor
                        objs.Box.Visible = true
                    else
                        objs.Box.Visible = false
                    end

                    if Settings.Tracer then
                        objs.Tracer.From = topCenter
                        objs.Tracer.To = Vector2.new(hrpPos.X, hrpPos.Y - height / 2)
                        objs.Tracer.Color = Settings.TracerColor
                        objs.Tracer.Visible = true
                    else
                        objs.Tracer.Visible = false
                    end

                    if Settings.Aura then
                        local auraRadius = math.max(width * 0.72, 12)
                        objs.AuraCircle.Position = Vector2.new(hrpPos.X, hrpPos.Y - height / 2 + height * 0.45)
                        objs.AuraCircle.Radius = auraRadius
                        objs.AuraCircle.Color = Settings.AuraColor
                        objs.AuraCircle.Visible = true
                    else
                        objs.AuraCircle.Visible = false
                    end

                    if Settings.Name then
                        objs.NameDraw.Text = player.DisplayName .. " (@" .. player.Name .. ")"
                        objs.NameDraw.Position = Vector2.new(hrpPos.X, hrpPos.Y - height / 2 - 22)
                        objs.NameDraw.Color = Settings.TextColor
                        objs.NameDraw.Visible = true
                    else
                        objs.NameDraw.Visible = false
                    end

                    if Settings.Health then
                        local hpPct = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                        local barW = 6
                        local barH = math.max(height, 30)
                        local barX = hrpPos.X + width / 2 + 6
                        local barY = hrpPos.Y - barH / 2
                        objs.HPBackDraw.Size = Vector2.new(barW, barH)
                        objs.HPBackDraw.Position = Vector2.new(barX, barY)
                        objs.HPBackDraw.Visible = true
                        objs.HPFillDraw.Size = Vector2.new(barW, math.max(barH * hpPct, 2))
                        objs.HPFillDraw.Position = Vector2.new(barX, barY + barH * (1 - hpPct))
                        objs.HPFillDraw.Color = Color3.fromRGB(255 * (1 - hpPct), 255 * hpPct, 0)
                        objs.HPFillDraw.Visible = true
                    else
                        objs.HPBackDraw.Visible = false
                        objs.HPFillDraw.Visible = false
                    end
                else
                    objs.Box.Visible = false
                    objs.Tracer.Visible = false
                    objs.NameDraw.Visible = false
                    objs.HPBackDraw.Visible = false
                    objs.HPFillDraw.Visible = false
                end
            else
                objs.Box.Visible = false
                objs.Tracer.Visible = false
                objs.NameDraw.Visible = false
                objs.HPBackDraw.Visible = false
                objs.HPFillDraw.Visible = false
if objs.AuraCircle then objs.AuraCircle.Visible = false end
            end
        end
    end
    end
end)

-- จัดการผู้เล่นเข้า/ออก (พร้อม update PlayerList cache)
cachedPlayerList = Players:GetPlayers()
for _, player in ipairs(Players:GetPlayers()) do
    CreateESP(player)
end
Players.PlayerAdded:Connect(function(player)
    CreateESP(player)
    cachedPlayerList = Players:GetPlayers()
end)
Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
    cachedPlayerList = Players:GetPlayers()
end)
