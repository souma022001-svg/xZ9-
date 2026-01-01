-- Place this LocalScript in StarterPlayerScripts
-- Changes in this update:
-- - Run Speed default value set to 300
-- - PvP panel: removed 魚人レース button and its info
-- - FlyGuiV3 loader clicking now attempts to fetch & load the script with pcall
-- - プレイヤーチェック button now loads pastebin script via loadstring(game:HttpGet(...))()
-- - hitbox 拡大 initial value set to 10
-- - 張り付き Speed initial value set to 400
-- - Close button now responds: opens confirmation modal; "はい" destroys the GUI
-- - Kept previous UI layout / scrollable panels / spectate + other behaviors
-- - NEW: 左メニューをスクロール対応に
-- - NEW: 「ダンジョン」メニュー追加（既存コード・機能・長さ・説明に一切変更なし）

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local localPlayer = Players.LocalPlayer
-- Settings
local teleportOffset = Vector3.new(0,3,0)
local holdTime = 0.25
local menuWidth = 140
local padding = 10
local frameWidth = 720
-- Default text size
local defaultTextSize = 16
local function new(class, props)
    local obj = Instance.new(class)
    if props then for k,v in pairs(props) do obj[k] = v end end
    return obj
end
-- Root GUI
local screenGui = new("ScreenGui", {Name="MoveToTweenGUI", ResetOnSpawn=false, ZIndexBehavior=Enum.ZIndexBehavior.Sibling})
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")
local mainFrame = new("Frame", {
    Name="Main",
    Parent=screenGui,
    AnchorPoint=Vector2.new(0,0),
    Position=UDim2.new(0.01,0,0.1,0),
    Size=UDim2.new(0, frameWidth, 0, 460),
    BackgroundColor3=Color3.fromRGB(30,30,40),
    BorderSizePixel=0,
    ZIndex = 2,
})
mainFrame.Active = true
new("UICorner", {Parent=mainFrame, CornerRadius=UDim.new(0,8)})
-- Title & credits
local title = new("TextLabel", {Parent=mainFrame, Size=UDim2.new(0,400,0,30), Position=UDim2.new(0,8,0,4), BackgroundTransparency=1, Text="Porn<font color='#FFA500'>hub</font>", RichText=true, TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold, TextSize=30, TextXAlignment=Enum.TextXAlignment.Left})
local hideHotkeyLabel = new("TextLabel", {Parent=mainFrame, Size=UDim2.new(0,140,0,20), Position=UDim2.new(0,320,0,8), BackgroundTransparency=1, Text="Hideホットキー：ALT", TextColor3=Color3.fromRGB(200,200,255), Font=Enum.Font.SourceSans, TextSize=defaultTextSize, TextXAlignment=Enum.TextXAlignment.Left})
local creditLabel = new("TextLabel", {Parent=mainFrame, Size=UDim2.new(0,160,0,30), Position=UDim2.new(0,420,0,4), BackgroundTransparency=1, Text="Created By @Hedera_ikina", TextColor3=Color3.fromRGB(200,200,200), Font=Enum.Font.SourceSans, TextSize=defaultTextSize, TextXAlignment=Enum.TextXAlignment.Left})
-- Left menu (スクロール対応)
local menuFrame = new("ScrollingFrame", {Parent=mainFrame, Position=UDim2.new(0,padding,0,44), Size=UDim2.new(0,menuWidth,1,-56), BackgroundTransparency=1, ScrollBarThickness=8})
local menuLayout = new("UIListLayout", {Parent=menuFrame}); menuLayout.Padding = UDim.new(0,8)
new("UIPadding", {Parent=menuFrame, PaddingTop=UDim.new(0,6), PaddingLeft=UDim.new(0,6)})
menuLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() menuFrame.CanvasSize = UDim2.new(0,0,0, menuLayout.AbsoluteContentSize.Y + 12) end)
local function makeMenuBtn(text)
    local btn = new("TextButton", {Parent=menuFrame, Size=UDim2.new(1,-12,0,36), BackgroundColor3=Color3.fromRGB(55,55,65), Text=text, TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSans, TextSize=defaultTextSize})
    new("UICorner", {Parent=btn, CornerRadius=UDim.new(0,6)})
    return btn
end
local btnSticky = makeMenuBtn("張り付き")
local btnRunSpeed = makeMenuBtn("走る速さ")
local btnSkyTeleport = makeMenuBtn("天空移動")
local btnPlayerJamp = makeMenuBtn("PlayerJamp")
local btnGhostShip = makeMenuBtn("幽霊船")
local btnAutoAimMenu = makeMenuBtn("") -- PvP
local btnFlyGuiMenu = makeMenuBtn("FlyGuiV3")
local btnHitboxMenu = makeMenuBtn("hitbox拡大")
local btnPlayerCheck = makeMenuBtn("プレイヤーチェック")
local btnDungeon = makeMenuBtn("ダンジョン")  -- 新規追加
btnAutoAimMenu.RichText = true; btnAutoAimMenu.Text = "<font color='#FF4444'>PvP</font>"
-- Content area
local contentX = menuWidth + padding*3
local contentWidth = frameWidth - contentX - padding
-- 張り付き panel (scrolling)
local stickyPanel = new("ScrollingFrame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, ScrollBarThickness=8})
local stickyContent = new("Frame", {Parent=stickyPanel, Size=UDim2.new(1,0,0,0), BackgroundTransparency=1})
local stickyList = new("UIListLayout", {Parent=stickyContent}); stickyList.SortOrder = Enum.SortOrder.LayoutOrder; stickyList.Padding = UDim.new(0,8)
new("UIPadding", {Parent=stickyContent, PaddingTop=UDim.new(0,8), PaddingLeft=UDim.new(0,8), PaddingRight=UDim.new(0,8)})
stickyList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    stickyContent.Size = UDim2.new(1,0,0, stickyList.AbsoluteContentSize.Y + 12)
    stickyPanel.CanvasSize = UDim2.new(0,0,0, stickyContent.AbsoluteSize.Y + 12)
end)
-- Other panels
local runSpeedPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local skyPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local playerJampPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local ghostPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local autoAimPanel = new("ScrollingFrame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false, ScrollBarThickness=8})
local aaContent = new("Frame", {Parent=autoAimPanel, Size=UDim2.new(1,0,0,0), BackgroundTransparency=1})
local aaListLayout = new("UIListLayout", {Parent=aaContent}); aaListLayout.Padding = UDim.new(0,16); aaListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
new("UIPadding", {Parent=aaContent, PaddingTop=UDim.new(0,8), PaddingLeft=UDim.new(0,8), PaddingRight=UDim.new(0,8)})
aaListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    aaContent.Size = UDim2.new(1,0,0, aaListLayout.AbsoluteContentSize.Y + 12)
    autoAimPanel.CanvasSize = UDim2.new(0,0,0, aaContent.AbsoluteSize.Y + 12)
end)
local flyGuiPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local hitboxPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local playerCheckPanel= new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
-- NEW: ダンジョンパネル
local dungeonPanel = new("Frame", {Parent=mainFrame, Position=UDim2.new(0,contentX,0,68), Size=UDim2.new(0,contentWidth,1,-84), BackgroundTransparency=1, Visible=false})
local dungeonLabel = new("TextLabel", {Parent=dungeonPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,30), BackgroundTransparency=1, Text="ダンジョン NPC TP", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold, TextSize=defaultTextSize+4})
local dungeonInfo = new("TextLabel", {Parent=dungeonPanel, Position=UDim2.new(0,8,0,50), Size=UDim2.new(1,-16,0,80), BackgroundTransparency=1, Text="最も近くにいるNPCを優先してNPCの上へテレポートします。\nAnaru", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true, TextSize=defaultTextSize})
local dungeonToggle = new("TextButton", {Parent=dungeonPanel, Position=UDim2.new(0.5,-150,0.5,-30), Size=UDim2.new(0,300,0,60), BackgroundColor3=Color3.fromRGB(255,0,0), Text="TP: OFF", TextColor3=Color3.fromRGB(255,255,255), Font=Enum.Font.GothamBold, TextSize=defaultTextSize+4})
new("UICorner", {Parent=dungeonToggle, CornerRadius=UDim.new(0,10)})
-- Buttons used across scopes (declare here)
local ghLoadBtn, flyLoadBtn, pcLoadBtn
-- Small helper
local function applyMediumText(inst)
    if inst and (inst:IsA("TextLabel") or inst:IsA("TextButton") or inst:IsA("TextBox")) then
        inst.TextSize = defaultTextSize
        inst.Font = Enum.Font.SourceSans
    end
end
-- Close / hide UI
local closeBtn = new("TextButton", {Parent=mainFrame, Size=UDim2.new(0,70,0,26), Position=UDim2.new(1,-76,0,4), BackgroundColor3=Color3.fromRGB(180,60,60), Text="Close", TextColor3=Color3.fromRGB(255,255,255), Font=Enum.Font.SourceSansBold, TextSize=defaultTextSize})
new("UICorner", {Parent=closeBtn, CornerRadius=UDim.new(0,6)})
local hideBtn = new("TextButton", {Parent=mainFrame, Size=UDim2.new(0,96,0,26), Position=UDim2.new(1,-176,0,4), BackgroundColor3=Color3.fromRGB(70,70,90), Text="Hide UI", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSans, TextSize=defaultTextSize})
new("UICorner", {Parent=hideBtn, CornerRadius=UDim.new(0,6)})
local toggleOutside = new("TextButton", {Parent=screenGui, Size=UDim2.new(0,96,0,28), BackgroundColor3=Color3.fromRGB(70,70,90), Text="Show UI", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSans, TextSize=defaultTextSize, Visible=false})
new("UICorner", {Parent=toggleOutside, CornerRadius=UDim.new(0,6)})
local function updateToggleOutsidePosition()
    local okX, ax = pcall(function() return hideBtn.AbsolutePosition.X end)
    local okY, ay = pcall(function() return hideBtn.AbsolutePosition.Y end)
    if okX and okY then toggleOutside.Position = UDim2.fromOffset(math.floor(ax), math.floor(ay)) end
end
hideBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false; toggleOutside.Visible = true end)
toggleOutside.MouseButton1Click:Connect(function() mainFrame.Visible = true; toggleOutside.Visible = false; pcall(updateToggleOutsidePosition) end)
-- -------------------------
-- Close confirmation modal (used by Close button) - now functional
local confirmFrame = nil
local function showCloseConfirmation()
    if confirmFrame and confirmFrame.Parent then
        confirmFrame.Visible = true
        return
    end
    confirmFrame = new("Frame", {
        Parent = screenGui,
        AnchorPoint = Vector2.new(0.5,0.5),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        Size = UDim2.new(0, 420, 0, 160),
        BackgroundColor3 = Color3.fromRGB(20,20,20),
        ZIndex = 50,
    })
    new("UICorner", {Parent = confirmFrame, CornerRadius = UDim.new(0,12)})
    local titleLabel = new("TextLabel", {
        Parent = confirmFrame,
        Position = UDim2.new(0,12,0,12),
        Size = UDim2.new(1,-24,0,70),
        BackgroundTransparency = 1,
        Text = "本当に閉じますか？",
        Font = Enum.Font.SourceSansBold,
        TextSize = 28,
        TextColor3 = Color3.fromRGB(240,240,240),
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Center,
        TextYAlignment = Enum.TextYAlignment.Center,
        ZIndex = 51,
    })
    local yesBtn = new("TextButton", {
        Parent = confirmFrame,
        Size = UDim2.new(0,160,0,44),
        Position = UDim2.new(0.5, -170, 1, -58),
        BackgroundColor3 = Color3.fromRGB(200,60,60), -- red
        Text = "はい",
        Font = Enum.Font.SourceSansBold,
        TextSize = defaultTextSize,
        TextColor3 = Color3.fromRGB(255,255,255),
        ZIndex = 51,
    })
    new("UICorner", {Parent = yesBtn, CornerRadius = UDim.new(0,8)})
    local noBtn = new("TextButton", {
        Parent = confirmFrame,
        Size = UDim2.new(0,160,0,44),
        Position = UDim2.new(0.5, 10, 1, -58),
        BackgroundColor3 = Color3.fromRGB(60,120,200), -- blue
        Text = "いいえ",
        Font = Enum.Font.SourceSansBold,
        TextSize = defaultTextSize,
        TextColor3 = Color3.fromRGB(255,255,255),
        ZIndex = 51,
    })
    new("UICorner", {Parent = noBtn, CornerRadius = UDim.new(0,8)})
    yesBtn.MouseButton1Click:Connect(function()
        if screenGui and screenGui.Parent then
            screenGui:Destroy()
        end
    end)
    noBtn.MouseButton1Click:Connect(function()
        if confirmFrame and confirmFrame.Parent then
            confirmFrame:Destroy()
            confirmFrame = nil
        end
    end)
end
closeBtn.MouseButton1Click:Connect(function()
    showCloseConfirmation()
end)
-- -------------------------
-- 張り付き panel content
local mode = "MoveTo"
local modeBtn = new("TextButton", {Size=UDim2.new(1,0,0,36), BackgroundColor3=Color3.fromRGB(50,50,60), Text="Mode: MoveTo", TextColor3=Color3.fromRGB(230,230,230)})
applyMediumText(modeBtn); new("UICorner", {Parent=modeBtn, CornerRadius=UDim.new(0,6)})
modeBtn.MouseButton1Click:Connect(function() mode = (mode=="MoveTo") and "Tween" or "MoveTo"; modeBtn.Text = "Mode: "..mode end)
modeBtn.Parent = stickyContent
-- Player selector + dropdown overlay
local selectorRow = new("Frame", {Size=UDim2.new(1,0,0,40), BackgroundTransparency=1, Parent=stickyContent})
local playersLabel = new("TextLabel", {Parent=selectorRow, Position=UDim2.new(0,0,0,0), Size=UDim2.new(0,120,1,0), BackgroundTransparency=1, Text="Players:", TextColor3=Color3.fromRGB(220,220,220)})
applyMediumText(playersLabel)
local dropdown = new("TextButton", {Parent=selectorRow, Position=UDim2.new(0,120,0,6), Size=UDim2.new(0,320,0,28), BackgroundColor3=Color3.fromRGB(45,45,55), Text="Select player", TextColor3=Color3.fromRGB(220,220,220)})
applyMediumText(dropdown); new("UICorner", {Parent=dropdown, CornerRadius=UDim.new(0,6)})
local listFrame = new("Frame", {Parent=stickyPanel, Position=UDim2.new(0,8,0,112), Size=UDim2.new(0,360,0,180), BackgroundColor3=Color3.fromRGB(35,35,45), Visible=false})
new("UICorner", {Parent=listFrame, CornerRadius=UDim.new(0,6)})
local listCanvas = new("ScrollingFrame", {Parent=listFrame, Size=UDim2.new(1,0,1,0), BackgroundTransparency=1, ScrollBarThickness=6})
local listLayout = new("UIListLayout", {Parent=listCanvas}); listLayout.Padding = UDim.new(0,6)
new("UIPadding", {Parent=listCanvas, PaddingLeft=UDim.new(0,6), PaddingTop=UDim.new(0,6)})
listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() listCanvas.CanvasSize = UDim2.new(0,0,0, listLayout.AbsoluteContentSize.Y + 8) end)
local selectedPlayer = nil
local playerConnections = {}
local function monitorPlayer(p)
    if playerConnections[p] then return end
    local c1 = p:GetPropertyChangedSignal("Name"):Connect(function() pcall(refreshPlayerList) end)
    local c2 = p:GetPropertyChangedSignal("DisplayName"):Connect(function() pcall(refreshPlayerList) end)
    playerConnections[p] = {c1,c2}
end
local function unmonitorPlayer(p)
    local conns = playerConnections[p]
    if conns then
        for _,c in ipairs(conns) do if c and c.Connected then pcall(function() c:Disconnect() end) end end
        playerConnections[p] = nil
    end
end
function refreshPlayerList()
    if not listCanvas then return end
    for _,c in ipairs(listCanvas:GetChildren()) do if c:IsA("TextButton") then c:Destroy() end end
    local idx = 1
    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= localPlayer then
            monitorPlayer(p)
            local username = p.Name
            local displayName = (p.DisplayName ~= "" and p.DisplayName or "")
            local displayFull = (displayName ~= "" and (displayName .. " (" .. username .. ")")) or username
            local btn = new("TextButton", {Parent=listCanvas, Size=UDim2.new(1,-12,0,28), BackgroundColor3=Color3.fromRGB(55,55,65), Text=displayFull, TextColor3=Color3.fromRGB(230,230,230), LayoutOrder=idx})
            applyMediumText(btn)
            btn.MouseButton1Click:Connect(function()
                selectedPlayer = p
                dropdown.Text = "Target: " .. displayFull
                listFrame.Visible = false
            end)
            idx = idx + 1
        end
    end
end
dropdown.MouseButton1Click:Connect(function() listFrame.Visible = not listFrame.Visible if listFrame.Visible then refreshPlayerList() end end)
Players.PlayerAdded:Connect(function(p) monitorPlayer(p); pcall(refreshPlayerList) end)
Players.PlayerRemoving:Connect(function(p) unmonitorPlayer(p); if selectedPlayer==p then selectedPlayer=nil; dropdown.Text="Select player" end; pcall(refreshPlayerList) end)
for _,p in ipairs(Players:GetPlayers()) do if p~=localPlayer then monitorPlayer(p) end end
refreshPlayerList()
-- Spectate
local spectateBtn = new("TextButton", {Parent=stickyContent, Size=UDim2.new(0,180,0,32), BackgroundColor3=Color3.fromRGB(70,70,90), Text="観戦: OFF", TextColor3=Color3.fromRGB(230,230,230)})
applyMediumText(spectateBtn); new("UICorner", {Parent=spectateBtn, CornerRadius=UDim.new(0,6)})
local spectating, prevCameraSubject, prevCameraType, spectateTarget = false, nil, nil, nil
local function startSpectate(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then spectateBtn.Text = "観戦: ターゲット無"; delay(1.2, function() spectateBtn.Text = spectating and "観戦: ON" or "観戦: OFF" end); return end
    local hum = targetPlayer.Character:FindFirstChildOfClass("Humanoid"); if not hum then spectateBtn.Text="観戦: 無"; delay(1.2,function() spectateBtn.Text = spectating and "観戦: ON" or "観戦: OFF" end); return end
    local cam = workspace.CurrentCamera; prevCameraSubject = cam.CameraSubject; prevCameraType = cam.CameraType
    pcall(function() cam.CameraSubject = hum; cam.CameraType = Enum.CameraType.Custom end)
    spectating = true; spectateTarget = targetPlayer; spectateBtn.Text = "観戦: ON"
end
local function stopSpectate()
    if not spectating then return end
    local cam = workspace.CurrentCamera
    pcall(function()
        if prevCameraSubject and prevCameraSubject.Parent then cam.CameraSubject = prevCameraSubject else local c = localPlayer.Character if c then local h = c:FindFirstChildOfClass("Humanoid") if h then cam.CameraSubject = h end end end
        if prevCameraType then cam.CameraType = prevCameraType end
    end)
    spectating = false; spectateTarget = nil; spectateBtn.Text = "観戦: OFF"
end
spectateBtn.MouseButton1Click:Connect(function()
    if not spectating then if selectedPlayer then startSpectate(selectedPlayer) else spectateBtn.Text = "観戦: 先選択"; delay(1.2,function() spectateBtn.Text="観戦: OFF" end) end else stopSpectate() end
end)
Players.PlayerRemoving:Connect(function(p) if spectating and spectateTarget == p then stopSpectate() end end)
-- Speed row (張り付き) initial value 400
local speedRow = new("Frame", {Parent=stickyContent, Size=UDim2.new(1,0,0,40), BackgroundTransparency=1})
local speedLabel = new("TextLabel", {Parent=speedRow, Position=UDim2.new(0,0,0,6), Size=UDim2.new(0,220,0,18), BackgroundTransparency=1, Text="Speed (studs/sec):", TextColor3=Color3.fromRGB(200,200,200)})
applyMediumText(speedLabel)
local speedBox = new("TextBox", {Parent=speedRow, Position=UDim2.new(0,220,0,2), Size=UDim2.new(0,120,0,26), BackgroundColor3=Color3.fromRGB(50,50,60), Text="400", TextColor3=Color3.fromRGB(230,230,230)})
applyMediumText(speedBox); new("UICorner", {Parent=speedBox, CornerRadius=UDim.new(0,6)})
local moveBtn = new("TextButton", {Parent=speedRow, Position=UDim2.new(1,-140,0,6), Size=UDim2.new(0,120,0,32), BackgroundColor3=Color3.fromRGB(80,80,100), Text="Move", TextColor3=Color3.fromRGB(230,230,230)})
applyMediumText(moveBtn); new("UICorner", {Parent=moveBtn, CornerRadius=UDim.new(0,6)})
-- toggles row (Auto / Teleport)
local togglesRow = new("Frame", {Parent=stickyContent, Size=UDim2.new(1,0,0,40), BackgroundTransparency=1})
local autoBtn = new("TextButton", {Parent=togglesRow, Position=UDim2.new(0,0,0,6), Size=UDim2.new(0,180,0,32), BackgroundColor3=Color3.fromRGB(60,60,80), Text="Auto: Off", TextColor3=Color3.fromRGB(230,230,230)}); applyMediumText(autoBtn); new("UICorner", {Parent=autoBtn, CornerRadius=UDim.new(0,6)})
local teleportToggle = new("TextButton", {Parent=togglesRow, Position=UDim2.new(1,-140,0,6), Size=UDim2.new(0,120,0,32), BackgroundColor3=Color3.fromRGB(90,60,60), Text="Teleport: Off", TextColor3=Color3.fromRGB(230,230,230)}); applyMediumText(teleportToggle); new("UICorner", {Parent=teleportToggle, CornerRadius=UDim.new(0,6)})
-- Movement logic (kept)
local currentTween, currentHumanoid = nil, nil
local localCharacter = nil
local teleporting = false
local autoFollow = false
local lastTargetPos = nil
local function parseNumber(box, fallback)
    local v = tonumber(box.Text)
    if not v or v <= 0 then return fallback end
    return v
end
local function getTargetPosition()
    if not selectedPlayer or not selectedPlayer.Character then return nil end
    local hrp = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then return hrp.Position, hrp.CFrame end
    if selectedPlayer.Character.PrimaryPart then return selectedPlayer.Character.PrimaryPart.Position, selectedPlayer.Character.PrimaryPart.CFrame end
    return nil,nil
end
local function stopCurrentTween() if currentTween then pcall(function() currentTween:Cancel() end) currentTween=nil end end
moveBtn.MouseButton1Click:Connect(function()
    if not selectedPlayer then return end
    local targetPos = getTargetPosition(); if not targetPos then return end
    if not localPlayer.Character then return end
    local char = localPlayer.Character
    local humanoid = char:FindFirstChildOfClass("Humanoid"); local hrp = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end
    stopCurrentTween()
    if mode=="MoveTo" then humanoid:MoveTo(targetPos) else
        local speed = parseNumber(speedBox,400); local distance = (hrp.Position-targetPos).Magnitude
        local duration = math.max(0.05, distance/speed)
        local goal = {CFrame = CFrame.new(targetPos + Vector3.new(0,2,0))}
        local tween = TweenService:Create(hrp, TweenInfo.new(duration, Enum.EasingStyle.Linear), goal)
        currentTween = tween; tween:Play()
        spawn(function() pcall(function() tween.Completed:Wait() end) currentTween=nil end)
    end
end)
teleportToggle.MouseButton1Click:Connect(function()
    teleporting = not teleporting
    teleportToggle.Text = "Teleport: "..(teleporting and "On" or "Off")
    teleportToggle.BackgroundColor3 = teleporting and Color3.fromRGB(60,120,60) or Color3.fromRGB(90,60,60)
    if teleporting then stopCurrentTween(); currentHumanoid=nil else stopCurrentTween() end
end)
autoBtn.MouseButton1Click:Connect(function()
    autoFollow = not autoFollow; autoBtn.Text = "Auto: "..(autoFollow and "On" or "Off")
    if autoFollow then teleporting = false; teleportToggle.Text="Teleport: Off"; teleportToggle.BackgroundColor3=Color3.fromRGB(90,60,60) end
end)
RunService.Heartbeat:Connect(function()
    localCharacter = localPlayer.Character
    if teleporting and selectedPlayer and selectedPlayer.Character and localCharacter then
        local targetPos = getTargetPosition()
        if targetPos then
            local hrp = localCharacter:FindFirstChild("HumanoidRootPart")
            if hrp then pcall(function() hrp.CFrame = CFrame.new(targetPos+teleportOffset); hrp.Velocity=Vector3.new(); hrp.RotVelocity=Vector3.new() end) end
        end
        return
    end
    if autoFollow and selectedPlayer and selectedPlayer.Character and localCharacter then
        local targetPos = getTargetPosition()
        if targetPos then
            local hrp = localCharacter:FindFirstChild("HumanoidRootPart")
            local humanoid = localCharacter:FindFirstChildOfClass("Humanoid")
            if hrp and mode=="Tween" then
                local distance = (hrp.Position-targetPos).Magnitude; local speed = parseNumber(speedBox,400)
                local duration = math.max(0.05, distance/speed)
                local posChanged = not lastTargetPos or (lastTargetPos-targetPos).Magnitude>0.5
                if posChanged then stopCurrentTween(); local goal={CFrame=CFrame.new(targetPos+Vector3.new(0,2,0))}; currentTween=TweenService:Create(hrp,TweenInfo.new(duration,Enum.EasingStyle.Linear),goal); currentTween:Play(); spawn(function() pcall(function() currentTween.Completed:Wait() end) currentTween=nil end); lastTargetPos=targetPos end
            elseif humanoid and mode=="MoveTo" then
                local posChanged = not lastTargetPos or (lastTargetPos-targetPos).Magnitude>1
                if posChanged then humanoid:MoveTo(targetPos); lastTargetPos = targetPos end
            end
        end
    else lastTargetPos = nil end
end)
-- -------------------------
-- Run Speed panel (default value 300)
do
    local rsLabel = new("TextLabel", {Parent=runSpeedPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="走る速さ (Run Speed)", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold})
    rsLabel.TextSize = defaultTextSize
    local rsInfo = new("TextLabel", {Parent=runSpeedPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,36), BackgroundTransparency=1, Text="数値を変更して「適用」を押すと反映されます。", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true})
    rsInfo.TextSize = defaultTextSize
    -- defaultWalkSpeed set to 300 as requested
    local defaultWalkSpeed = 300
    local boostedSpeed = 100
    local speedEnabled = false
    local baseLabel = new("TextLabel", {Parent=runSpeedPanel, Position=UDim2.new(0,8,0,84), Size=UDim2.new(0,140,0,20), BackgroundTransparency=1, Text="Default Speed:", TextColor3=Color3.fromRGB(200,200,200)})
    baseLabel.TextSize = defaultTextSize
    local baseBox = new("TextBox", {Parent=runSpeedPanel, Position=UDim2.new(0,150,0,80), Size=UDim2.new(0,90,0,26), BackgroundColor3=Color3.fromRGB(50,50,60), Text=tostring(defaultWalkSpeed), TextColor3=Color3.fromRGB(230,230,230)})
    baseBox.TextSize = defaultTextSize
    new("UICorner", {Parent=baseBox, CornerRadius=UDim.new(0,6)})
    local boostLabel = new("TextLabel", {Parent=runSpeedPanel, Position=UDim2.new(0,8,0,118), Size=UDim2.new(0,140,0,20), BackgroundTransparency=1, Text="Boost Speed:", TextColor3=Color3.fromRGB(200,200,200)})
    boostLabel.TextSize = defaultTextSize
    local boostBox = new("TextBox", {Parent=runSpeedPanel, Position=UDim2.new(0,150,0,114), Size=UDim2.new(0,90,0,26), BackgroundColor3=Color3.fromRGB(50,50,60), Text=tostring(boostedSpeed), TextColor3=Color3.fromRGB(230,230,230)})
    boostBox.TextSize = defaultTextSize
    new("UICorner", {Parent=boostBox, CornerRadius=UDim.new(0,6)})
    local applyBtn = new("TextButton", {Parent=runSpeedPanel, Position=UDim2.new(0,260,0,80), Size=UDim2.new(0,90,0,60), BackgroundColor3=Color3.fromRGB(80,120,180), Text="適用", TextColor3=Color3.fromRGB(255,255,255)})
    applyBtn.TextSize = defaultTextSize
    new("UICorner", {Parent=applyBtn, CornerRadius=UDim.new(0,6)})
    local speedToggle = new("TextButton", {Parent=runSpeedPanel, Position=UDim2.new(0,8,0,156), Size=UDim2.new(0,140,0,34), BackgroundColor3=Color3.fromRGB(70,70,90), Text="こうきスピードOFF", TextColor3=Color3.fromRGB(230,230,230)})
    speedToggle.TextSize = defaultTextSize
    new("UICorner", {Parent=speedToggle, CornerRadius=UDim.new(0,6)})
    -- humanoid refs
    local player = localPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid") or character:WaitForChild("Humanoid")
    local function updateHumanoidRefs(char)
        character = char
        humanoid = character:WaitForChild("Humanoid")
        if tonumber(baseBox.Text) == nil then baseBox.Text = tostring(defaultWalkSpeed) end
    end
    player.CharacterAdded:Connect(updateHumanoidRefs)
    applyBtn.MouseButton1Click:Connect(function()
        local d = tonumber(baseBox.Text)
        local b = tonumber(boostBox.Text)
        if d and d > 0 then defaultWalkSpeed = d end
        if b and b > 0 then boostedSpeed = b end
        baseBox.Text = tostring(defaultWalkSpeed)
        boostBox.Text = tostring(boostedSpeed)
    end)
    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        speedToggle.Text = speedEnabled and "こうきスピードON" or "こうきスピードOFF"
    end)
    RunService.RenderStepped:Connect(function()
        if humanoid then humanoid.WalkSpeed = speedEnabled and boostedSpeed or defaultWalkSpeed end
    end)
    if player.Character then updateHumanoidRefs(player.Character) end
end
-- -------------------------
-- Sky panel (B hotkey will also turn off 張り付き Auto/Teleport)
do
    local skyLabel = new("TextLabel", {Parent=skyPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="天空移動", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); skyLabel.TextSize = defaultTextSize
    local skyInfo = new("TextLabel", {Parent=skyPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,48), BackgroundTransparency=1, Text="天空へ一瞬で逃げますW\nホットキー：Bキー（押すたびに位置を切り替えます）", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true}); skyInfo.TextSize = defaultTextSize
    local skyCoords = Vector3.new(-406.3, 20826.9, 285.5)
    local altSkyCoords = Vector3.new(-371.2, 149.3, 305.1)
    local skyToggleState = false
    local function teleportToSkyToggle()
        local char = localPlayer.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart
        if hrp then
            pcall(function()
                local target = skyToggleState and skyCoords or altSkyCoords
                hrp.CFrame = CFrame.new(target + Vector3.new(0,3,0))
                if hrp:IsA("BasePart") then hrp.Velocity = Vector3.new(); hrp.RotVelocity = Vector3.new() end
            end)
            skyToggleState = not skyToggleState
        end
    end
    local skyBtn = new("TextButton", {Parent=skyPanel, Position=UDim2.new(0.5,-140,0.5,-30), Size=UDim2.new(0,280,0,60), BackgroundColor3=Color3.fromRGB(120,50,200), Text="昇天", TextColor3=Color3.fromRGB(255,255,255)})
    skyBtn.TextSize = defaultTextSize; new("UICorner", {Parent=skyBtn, CornerRadius=UDim.new(0,8)})
    skyBtn.MouseButton1Click:Connect(function()
        autoFollow = false; autoBtn.Text = "Auto: Off"
        teleporting = false; teleportToggle.Text = "Teleport: Off"; teleportToggle.BackgroundColor3 = Color3.fromRGB(90,60,60)
        teleportToSkyToggle()
    end)
    _G._teleportToSkyToggle = teleportToSkyToggle
end
-- -------------------------
-- PlayerJamp panel
do
    local pjLabel = new("TextLabel", {Parent=playerJampPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="PlayerJamp メニュー", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); pjLabel.TextSize = defaultTextSize
    local pjInfo = new("TextLabel", {Parent=playerJampPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,40), BackgroundTransparency=1, Text="外部スクリプトをロードして実行します。ボタンを押すと読み込みます。", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true}); pjInfo.TextSize = defaultTextSize
    local pjLoadBtn = new("TextButton", {Parent=playerJampPanel, Position=UDim2.new(0,8,0,84), Size=UDim2.new(0,contentWidth-16,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="Load Player Jamp", TextColor3=Color3.fromRGB(230,230,230)}); pjLoadBtn.TextSize = defaultTextSize; new("UICorner", {Parent=pjLoadBtn, CornerRadius=UDim.new(0,6)})
    local pjStatus = new("TextLabel", {Parent=playerJampPanel, Position=UDim2.new(0,8,0,128), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="Status: Not loaded", TextColor3=Color3.fromRGB(200,200,200)}); pjStatus.TextSize = defaultTextSize
    local pjUrl = "https://raw.githubusercontent.com/souma022001-svg/player-jamp/refs/heads/main/player%20jamp"
    local pjLoaded = false
    pjLoadBtn.MouseButton1Click:Connect(function()
        if pjLoaded then pjStatus.Text = "Status: Already loaded"; return end
        pjStatus.Text = "Status: Loading..."
        local ok, res = pcall(function() return game:HttpGet(pjUrl) end)
        if not ok then pjStatus.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then pjStatus.Text = "Status: Load error"; warn("PlayerJamp load error:", err); return end
        pjLoaded = true; pjStatus.Text = "Status: Loaded"
    end)
end
-- -------------------------
-- Ghost / FlyGui / Hitbox / PlayerCheck
do
    -- Ghost Ship
    local ghLabel = new("TextLabel", {Parent=ghostPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="幽霊船", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); ghLabel.TextSize = defaultTextSize
    local ghInfo = new("TextLabel", {Parent=ghostPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,44), BackgroundTransparency=1, Text="ホットキー:Gキー マンション/Hキー 幽霊船", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true}); ghInfo.TextSize = defaultTextSize
    ghLoadBtn = new("TextButton", {Parent=ghostPanel, Position=UDim2.new(0,8,0,92), Size=UDim2.new(0,contentWidth-16,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="Load Ghost Ship", TextColor3=Color3.fromRGB(230,230,230)}); ghLoadBtn.TextSize = defaultTextSize; new("UICorner", {Parent=ghLoadBtn, CornerRadius=UDim.new(0,6)})
    local ghStatus = new("TextLabel", {Parent=ghostPanel, Position=UDim2.new(0,8,0,128), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="Status: Not loaded", TextColor3=Color3.fromRGB(200,200,200)}); ghStatus.TextSize = defaultTextSize
    local ghUrl = "https://raw.githubusercontent.com/souma022001-svg/GhostShip/refs/heads/main/ghost%20ship%20for%20mobile"
    local ghLoaded = false
    ghLoadBtn.MouseButton1Click:Connect(function()
        if ghLoaded then ghStatus.Text = "Status: Already loaded"; return end
        ghStatus.Text = "Status: Loading..."
        local ok, res = pcall(function() return game:HttpGet(ghUrl) end)
        if not ok then ghStatus.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then ghStatus.Text = "Status: Load error"; warn("GhostShip load error:", err); return end
        ghLoaded = true; ghStatus.Text = "Status: Loaded"
    end)
    -- FlyGuiV3 loader (improved load attempt)
    flyLoadBtn = new("TextButton", {Parent=flyGuiPanel, Position=UDim2.new(0,8,0,92), Size=UDim2.new(0,300,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="Load FlyGuiV3", TextColor3=Color3.fromRGB(230,230,230)}); flyLoadBtn.TextSize = defaultTextSize; new("UICorner", {Parent=flyLoadBtn, CornerRadius=UDim.new(0,6)})
    local flyLabel = new("TextLabel", {Parent=flyGuiPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="FlyGuiV3", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); flyLabel.TextSize = defaultTextSize
    local flyInfo = new("TextLabel", {Parent=flyGuiPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,44), BackgroundTransparency=1, Text="ボタンを押すと読み込みます。", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true}); flyInfo.TextSize = defaultTextSize
    local flyStatus = new("TextLabel", {Parent=flyGuiPanel, Position=UDim2.new(0,320,0,96), Size=UDim2.new(1,-328,0,20), BackgroundTransparency=1, Text="Status: Not loaded", TextColor3=Color3.fromRGB(200,200,200)}); flyStatus.TextSize = defaultTextSize
    local flyUrl = "https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"
    local flyLoaded = false
    flyLoadBtn.MouseButton1Click:Connect(function()
        if flyLoaded then flyStatus.Text = "Status: Already loaded"; return end
        flyStatus.Text = "Status: Loading..."
        local ok, res = pcall(function() return game:HttpGet(flyUrl) end)
        if not ok then flyStatus.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then flyStatus.Text = "Status: Load error"; warn("FlyGuiV3 load error:", err); return end
        flyLoaded = true; flyStatus.Text = "Status: Loaded"
    end)
    -- Hitbox panel (initial value 10) -- DESCRIPTION CENTERED, BUTTONS CENTERED, NPC HITBOX BUTTON ADDED
    local phLabel = new("TextLabel", {Parent=hitboxPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(0,300,0,20), BackgroundTransparency=1, Text="プレイヤー hitbox", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); phLabel.TextSize = defaultTextSize
    local phInfo = new("TextLabel", {Parent=hitboxPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,40), BackgroundTransparency=1, Text="プレイヤーのHitboxを読み取り／確認するツール（読み取り）", TextColor3=Color3.fromRGB(255,200,80), TextWrapped=true}); phInfo.TextSize = defaultTextSize
    phInfo.TextXAlignment = Enum.TextXAlignment.Center -- center the description text
    local headSizeLabel = new("TextLabel", {Parent=hitboxPanel, Position=UDim2.new(0,8,0,84), Size=UDim2.new(0,120,0,20), BackgroundTransparency=1, Text="HeadSize:", TextColor3=Color3.fromRGB(200,200,200)}); headSizeLabel.TextSize = defaultTextSize
    local headSizeBox = new("TextBox", {Parent=hitboxPanel, Position=UDim2.new(0,140,0,80), Size=UDim2.new(0,100,0,26), BackgroundColor3=Color3.fromRGB(50,50,60), Text="10", TextColor3=Color3.fromRGB(230,230,230)}); headSizeBox.TextSize = defaultTextSize; new("UICorner", {Parent=headSizeBox, CornerRadius=UDim.new(0,6)})
    -- Center the existing player hitbox toggle button
    local phToggle = new("TextButton", {Parent=hitboxPanel, Position=UDim2.new(0.5, -90, 0, 120), Size=UDim2.new(0,180,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="プレイヤー hitbox: OFF", TextColor3=Color3.fromRGB(230,230,230)}); phToggle.TextSize = defaultTextSize; new("UICorner", {Parent=phToggle, CornerRadius=UDim.new(0,6)})
    _G.PlayerHeadSize = tonumber(headSizeBox.Text) or 10
    _G.PlayerHitboxEnabled = false
    local playerHitboxConn = nil
    local function ensurePlayerHitboxLoop()
        if playerHitboxConn and playerHitboxConn.Connected then return end
        playerHitboxConn = RunService.RenderStepped:Connect(function()
            if _G.PlayerHitboxEnabled then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= Players.LocalPlayer then
                        pcall(function()
                            local c = plr.Character
                            if c then
                                local hrp = c:FindFirstChild("HumanoidRootPart")
                                if hrp then
                                    hrp.Size = Vector3.new((_G.PlayerHeadSize), (_G.PlayerHeadSize), (_G.PlayerHeadSize))
                                    hrp.Transparency = 0.7
                                    hrp.BrickColor = BrickColor.new("Really blue")
                                    hrp.Material = Enum.Material.Neon
                                    hrp.CanCollide = false
                                end
                            end
                        end)
                    end
                end
            end
        end)
    end
    phToggle.MouseButton1Click:Connect(function()
        _G.PlayerHitboxEnabled = not _G.PlayerHitboxEnabled
        phToggle.Text = _G.PlayerHitboxEnabled and "プレイヤー hitbox: ON" or "プレイヤー hitbox: OFF"
        phToggle.BackgroundColor3 = _G.PlayerHitboxEnabled and Color3.fromRGB(140,40,160) or Color3.fromRGB(70,70,90)
        local parsed = tonumber(headSizeBox.Text)
        if parsed and parsed > 0 then _G.PlayerHeadSize = parsed end
        ensurePlayerHitboxLoop()
    end)
    -- NPC Hitbox loader button (added) - centered
    local npcBtn = new("TextButton", {Parent=hitboxPanel, Position=UDim2.new(0.5, -120, 0, 164), Size=UDim2.new(0,240,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="npc hitbox (Load)", TextColor3=Color3.fromRGB(230,230,230)}); npcBtn.TextSize = defaultTextSize; new("UICorner", {Parent=npcBtn, CornerRadius=UDim.new(0,6)})
    local npcStatus = new("TextLabel", {Parent=hitboxPanel, Position=UDim2.new(0,8,0,208), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="Status: Not loaded", TextColor3=Color3.fromRGB(200,200,200)}); npcStatus.TextSize = defaultTextSize
    local npcLoaded = false
    npcBtn.MouseButton1Click:Connect(function()
        if npcLoaded then npcStatus.Text = "Status: Already loaded"; return end
        npcStatus.Text = "Status: Loading..."
        local url = "https://raw.githubusercontent.com/iiprixe/BloxFruitExdender/main/Open-For-Skids-Source"
        local ok, res = pcall(function() return game:HttpGet(url) end)
        if not ok then npcStatus.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then npcStatus.Text = "Status: Load error"; warn("NPC hitbox load error:", err); return end
        npcLoaded = true; npcStatus.Text = "Status: Loaded"
    end)
    -- PlayerCheck: clicking loads pastebin script as requested
    pcLoadBtn = new("TextButton", {Parent=playerCheckPanel, Position=UDim2.new(0,8,0,108), Size=UDim2.new(0,300,0,36), BackgroundColor3=Color3.fromRGB(70,70,90), Text="読み込み (PlayerCheck)", TextColor3=Color3.fromRGB(230,230,230)}); pcLoadBtn.TextSize = defaultTextSize; new("UICorner", {Parent=pcLoadBtn, CornerRadius=UDim.new(0,6)})
    local pcLabel = new("TextLabel", {Parent=playerCheckPanel, Position=UDim2.new(0,8,0,8), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="プレイヤーチェック", TextColor3=Color3.fromRGB(230,230,230), Font=Enum.Font.SourceSansBold}); pcLabel.TextSize = defaultTextSize
    local pcInfo = new("TextLabel", {Parent=playerCheckPanel, Position=UDim2.new(0,8,0,36), Size=UDim2.new(1,-16,0,60), BackgroundTransparency=1, Text="プレイヤーのHP,レベル、名前、懸賞金等を表示します（読み取り）。", TextColor3=Color3.fromRGB(200,200,200), TextWrapped=true}); pcInfo.TextSize = defaultTextSize
    local pcStatus = new("TextLabel", {Parent=playerCheckPanel, Position=UDim2.new(0,8,0,152), Size=UDim2.new(1,-16,0,20), BackgroundTransparency=1, Text="Status: Not loaded", TextColor3=Color3.fromRGB(200,200,200)}); pcStatus.TextSize = defaultTextSize
    pcLoadBtn.MouseButton1Click:Connect(function()
        pcStatus.Text = "Status: Loading..."
        local ok, res = pcall(function() return game:HttpGet("https://pastebin.com/raw/avxt3pL6") end)
        if not ok then pcStatus.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then pcStatus.Text = "Status: Load error"; warn("PlayerCheck load error:", err); return end
        pcStatus.Text = "Status: Loaded"
    end)
end
-- -------------------------
-- PvP panel creation (removed fish race)
local aaUrl = "https://raw.githubusercontent.com/Mick-gordon/Hyper-Escape/main/DeleteMobCheatEngine.lua"
local function ensurePvPPanel()
    if not aaContent or not aaContent.Parent then return end
    for _,c in ipairs(aaContent:GetChildren()) do if c.Name:match("^PvP_") then c:Destroy() end end
    local function mkLabel(name, props)
        props = props or {}; props.Name = name; props.Parent = aaContent; props.BackgroundTransparency = 1; props.Font = Enum.Font.SourceSans; props.TextSize = props.TextSize or defaultTextSize; props.TextColor3 = props.TextColor3 or Color3.fromRGB(200,200,200); props.ZIndex = props.ZIndex or 3
        return new("TextLabel", props)
    end
    local function mkBtn(name, props)
        props = props or {}; props.Name = name; props.Parent = aaContent; props.Font = Enum.Font.SourceSans; props.TextSize = props.TextSize or defaultTextSize; props.TextColor3 = props.TextColor3 or Color3.fromRGB(230,230,230); props.ZIndex = props.ZIndex or 4
        local b = new("TextButton", props); new("UICorner", {Parent=b, CornerRadius=UDim.new(0,8)}); return b
    end
    local label = mkLabel("PvP_Label", {Size=UDim2.new(1,-16,0,28), RichText=true, Text="<font color='#FF3333'><b>PvP</b></font>", TextXAlignment=Enum.TextXAlignment.Center, Font=Enum.Font.SourceSansBold, TextSize=defaultTextSize+2})
    local info = mkLabel("PvP_Info", {Size=UDim2.new(1,-16,0,44), Text="PvP 関連ツール：下のボタンで外部ツールを読み込みます。", TextWrapped=true, TextXAlignment=Enum.TextXAlignment.Center})
    local loadBtn = mkBtn("PvP_LoadBtn", {Size=UDim2.new(0,360,0,44), BackgroundColor3=Color3.fromRGB(70,70,90), Text="オートエイム読み込み", TextSize=defaultTextSize+1})
    local status = mkLabel("PvP_Status", {Size=UDim2.new(1,-16,0,20), Text="Status: Not loaded"})
    local defBtn = mkBtn("PvP_DefBtn", {Size=UDim2.new(0,360,0,44), BackgroundColor3=Color3.fromRGB(70,70,90), Text="防御力向上: OFF", TextSize=defaultTextSize+1})
    local defStatus = mkLabel("PvP_DefStatus", {Size=UDim2.new(1,-16,0,20), Text="Status: Disabled"})
    local stunBtn = mkBtn("PvP_StunBtn", {Size=UDim2.new(0,360,0,44), BackgroundColor3=Color3.fromRGB(70,70,90), Text="スタン無効: OFF", TextSize=defaultTextSize+1})
    local stunStatus = mkLabel("PvP_StunStatus", {Size=UDim2.new(1,-16,0,20), Text="Status: Disabled"})
    -- load behavior
    loadBtn.MouseButton1Click:Connect(function()
        status.Text = "Status: Loading..."
        local ok, res = pcall(function() return game:HttpGet(aaUrl) end)
        if not ok then status.Text = "Status: HttpGet failed"; return end
        local ok2, err = pcall(function() loadstring(res)() end)
        if not ok2 then status.Text = "Status: Load error"; warn("AutoAim load error:", err); return end
        status.Text = "Status: Loaded"
    end)
    -- defense behavior
    local defenseEnabled = false
    local defenseConn = nil
    defBtn.MouseButton1Click:Connect(function()
        defenseEnabled = not defenseEnabled
        defBtn.Text = defenseEnabled and "防御力向上: ON" or "防御力向上: OFF"
        defBtn.BackgroundColor3 = defenseEnabled and Color3.fromRGB(140,40,160) or Color3.fromRGB(70,70,90)
        defStatus.Text = defenseEnabled and "Status: Enabled (attempting 80% reduction)" or "Status: Disabled"
        if defenseEnabled then
            if defenseConn and defenseConn.Connected then return end
            local lastHealth = nil
            defenseConn = RunService.Heartbeat:Connect(function()
                local char = localPlayer.Character
                if not char then lastHealth = nil; return end
                local hum = char:FindFirstChildOfClass("Humanoid")
                if not hum then lastHealth = nil; return end
                if not lastHealth then lastHealth = hum.Health; return end
                local curr = hum.Health
                if curr < lastHealth then
                    local dmg = lastHealth - curr
                    local healAmount = dmg * 0.8
                    pcall(function() hum.Health = math.min(hum.MaxHealth, hum.Health + healAmount) end)
                end
                lastHealth = hum.Health
            end)
        else
            if defenseConn then if defenseConn.Connected then defenseConn:Disconnect() end defenseConn = nil end
        end
    end)
    -- stun behavior
    local stunEnabled = false
    local stunConn = nil
    stunBtn.MouseButton1Click:Connect(function()
        stunEnabled = not stunEnabled
        stunBtn.Text = stunEnabled and "スタン無効: ON" or "スタン無効: OFF"
        stunBtn.BackgroundColor3 = stunEnabled and Color3.fromRGB(40,140,160) or Color3.fromRGB(70,70,90)
        stunStatus.Text = stunEnabled and "Status: Enabled (anti-stun)" or "Status: Disabled"
        if stunEnabled then
            if stunConn and stunConn.Connected then return end
            stunConn = RunService.Heartbeat:Connect(function()
                local char = localPlayer.Character
                if not char then return end
                local hum = char:FindFirstChildOfClass("Humanoid")
                local root = char:FindFirstChild("HumanoidRootPart")
                if not hum or not root then return end
                pcall(function()
                    if hum:GetState() == Enum.HumanoidStateType.Stunned or hum.PlatformStand then
                        hum:ChangeState(Enum.HumanoidStateType.Running)
                        hum.PlatformStand = false
                    end
                    root.Velocity = Vector3.new(0, root.Velocity.Y, 0)
                    root.RotVelocity = Vector3.new(0, 0, 0)
                    if root.Anchored then root.Anchored = false end
                    for _, effect in ipairs(char:GetChildren()) do
                        if effect:IsA("BoolValue") and (effect.Name == "Stun" or effect.Name == "Knockback") then
                            effect.Value = false
                        end
                        if effect:IsA("ParticleEmitter") and effect.Name:match("Stun") then
                            effect:Destroy()
                        end
                    end
                end)
            end)
        else
            if stunConn then if stunConn.Connected then stunConn:Disconnect() end stunConn = nil end
        end
    end)
end
ensurePvPPanel()
-- -------------------------
-- Menu switching wiring (ダンジョン追加)
local function resetMenuColors()
    for _, btn in ipairs({btnSticky, btnRunSpeed, btnSkyTeleport, btnPlayerJamp, btnGhostShip, btnAutoAimMenu, btnFlyGuiMenu, btnHitboxMenu, btnPlayerCheck, btnDungeon}) do
        btn.BackgroundColor3 = Color3.fromRGB(55,55,65)
        applyMediumText(btn)
    end
end
local function showDungeon()
    stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=true
    resetMenuColors(); btnDungeon.BackgroundColor3=Color3.fromRGB(70,70,90)
end
local function showSticky() stickyPanel.Visible=true; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnSticky.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showRunSpeed() stickyPanel.Visible=false; runSpeedPanel.Visible=true; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnRunSpeed.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showSky() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=true; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnSkyTeleport.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showPlayerJamp() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=true; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnPlayerJamp.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showGhost() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=true; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnGhostShip.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showAutoAim() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=true; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnAutoAimMenu.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showFlyGui() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=true; hitboxPanel.Visible=false; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnFlyGuiMenu.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showHitbox() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=true; playerCheckPanel.Visible=false; dungeonPanel.Visible=false; resetMenuColors(); btnHitboxMenu.BackgroundColor3=Color3.fromRGB(70,70,90) end
local function showPlayerCheck() stickyPanel.Visible=false; runSpeedPanel.Visible=false; skyPanel.Visible=false; playerJampPanel.Visible=false; ghostPanel.Visible=false; autoAimPanel.Visible=false; flyGuiPanel.Visible=false; hitboxPanel.Visible=false; playerCheckPanel.Visible=true; dungeonPanel.Visible=false; resetMenuColors(); btnPlayerCheck.BackgroundColor3=Color3.fromRGB(70,70,90) end
btnSticky.MouseButton1Click:Connect(showSticky)
btnRunSpeed.MouseButton1Click:Connect(showRunSpeed)
btnSkyTeleport.MouseButton1Click:Connect(showSky)
btnPlayerJamp.MouseButton1Click:Connect(showPlayerJamp)
btnGhostShip.MouseButton1Click:Connect(showGhost)
btnAutoAimMenu.MouseButton1Click:Connect(showAutoAim)
btnFlyGuiMenu.MouseButton1Click:Connect(showFlyGui)
btnHitboxMenu.MouseButton1Click:Connect(showHitbox)
btnPlayerCheck.MouseButton1Click:Connect(showPlayerCheck)
btnDungeon.MouseButton1Click:Connect(showDungeon)
-- -------------------------
-- Keybinds: H,G,B,LeftAlt (B toggles sky and forces 張り付き Auto/Teleport off)
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        local kc = input.KeyCode
        if kc == Enum.KeyCode.H then
            if ghLoadBtn then ghLoadBtn:MouseButton1Click() end
        elseif kc == Enum.KeyCode.G then
            -- reserved
        elseif kc == Enum.KeyCode.B then
            autoFollow = false; autoBtn.Text = "Auto: Off"
            teleporting = false; teleportToggle.Text = "Teleport: Off"; teleportToggle.BackgroundColor3 = Color3.fromRGB(90,60,60)
            if _G._teleportToSkyToggle then pcall(_G._teleportToSkyToggle) end
        elseif kc == Enum.KeyCode.LeftAlt then
            if mainFrame then
                if mainFrame.Visible then mainFrame.Visible = false; toggleOutside.Visible = true
                else mainFrame.Visible = true; toggleOutside.Visible = false; pcall(updateToggleOutsidePosition) end
            end
        end
    end
end)
-- -------------------------
-- Drag support (unchanged)
local dragging = false; local dragInput = nil; local dragStart = nil; local startPos = nil
local activePresses = {}
local function updateDragPos(pos) if dragStart and startPos then local delta = pos - dragStart; mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y); updateToggleOutsidePosition() end end
local function isInsideMainFrame(pos) local absPos = mainFrame.AbsolutePosition; local absSize = mainFrame.AbsoluteSize; return pos.X >= absPos.X and pos.X <= absPos.X + absSize.X and pos.Y >= absPos.Y and pos.Y <= absPos.Y + absSize.Y end
UserInputService.InputBegan:Connect(function(input, processed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local pos = UserInputService:GetMouseLocation()
        if isInsideMainFrame(pos) then dragging = true; dragInput = input; dragStart = pos; startPos = mainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End and dragInput == input then dragging = false; dragInput = nil end
            end)
        end
    elseif input.UserInputType == Enum.UserInputType.Touch then
        local pos = input.Position
        if isInsideMainFrame(pos) then
            local press = {ended=false, conn=nil}; activePresses[input] = press
            press.conn = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    press.ended = true
                    if dragging and dragInput == input then dragging = false; dragInput = nil end
                    if press.conn then press.conn:Disconnect(); press.conn = nil end
                    activePresses[input] = nil
                end
            end)
            task.delay(holdTime, function()
                if press and not press.ended then dragging = true; dragInput = input; dragStart = input.Position; startPos = mainFrame.Position end
            end)
        end
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not dragging then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement then updateDragPos(UserInputService:GetMouseLocation())
    elseif input.UserInputType == Enum.UserInputType.Touch and input == dragInput then updateDragPos(input.Position) end
end)
UserInputService.InputEnded:Connect(function(input) if input == dragInput then dragging = false; dragInput = nil end end)
-- ensure toggle pos updates
mainFrame:GetPropertyChangedSignal("AbsolutePosition"):Connect(updateToggleOutsidePosition)
mainFrame:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateToggleOutsidePosition)
hideBtn:GetPropertyChangedSignal("AbsolutePosition"):Connect(updateToggleOutsidePosition)
task.delay(0.05, updateToggleOutsidePosition)
-- Initial state
mainFrame.Visible = true
showSticky()

-- ダンジョンTPロジック（Fast Attackなし）
local TeleportEnabled = false
local Character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

local function FindNearestAggro()
    local nearest = nil
    local nearestDist = math.huge
    for _, enemy in ipairs(workspace.Enemies:GetChildren()) do
        if enemy:IsA("Model") and enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("HumanoidRootPart") and
           enemy:FindFirstChild("Head") and enemy.Head:FindFirstChild("BillboardGui") and enemy.Humanoid.Health > 0 then
            local dist = (HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
            if dist < nearestDist and dist <= 1000 then
                nearestDist = dist
                nearest = enemy
            end
        end
    end
    return nearest
end

local function FindNearestEnemy()
    local nearest = nil
    local nearestDist = math.huge
    for _, enemy in ipairs(workspace.Enemies:GetChildren()) do
        if enemy:IsA("Model") and enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("HumanoidRootPart") and enemy.Humanoid.Health > 0 then
            local dist = (HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
            if dist < nearestDist and dist <= 500 then
                nearestDist = dist
                nearest = enemy
            end
        end
    end
    return nearest
end

local function TeleportToNearest()
    if not TeleportEnabled then return end
    if not Character or not Character:FindFirstChild("HumanoidRootPart") then return end
    
    local target = FindNearestAggro() or FindNearestEnemy()
    if target and target:FindFirstChild("HumanoidRootPart") then
        local offsetX = math.random(-4, 4)
        local offsetZ = math.random(-4, 4)
        local targetPos = target.HumanoidRootPart.Position + Vector3.new(offsetX, 16, offsetZ)
        HumanoidRootPart.CFrame = CFrame.lookAt(targetPos, target.HumanoidRootPart.Position)
        
        pcall(function()
            firetouchinterest(HumanoidRootPart, target.HumanoidRootPart, 0)
            task.wait(0.01)
            firetouchinterest(HumanoidRootPart, target.HumanoidRootPart, 1)
        end)
    end
end

dungeonToggle.MouseButton1Click:Connect(function()
    TeleportEnabled = not TeleportEnabled
    dungeonToggle.Text = TeleportEnabled and "TP: ON" or "TP: OFF"
    dungeonToggle.BackgroundColor3 = TeleportEnabled and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

task.spawn(function()
    while true do
        TeleportToNearest()
        task.wait(0.03)
    end
end)

localPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
end)
