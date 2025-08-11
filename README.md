--高级ESP脚本v2.3-优化版(监测范围1000单位)
本地玩家=游戏：GetService(“玩家”)
本地RunService=game:GetService("RunService")
本地LocalPlayer=players.LocalPlayer

-- ===== 配置区域 =====
本地设置={
-- 敌人设置
敌人_TAGS={"耙"，"怪物"，"敌人"}，
f25Demy_COLOR=Color3.fromRGB(255，50，50)，
    
    -- 玩家设置
    player_COLOR=Color3.fromRGB(50，150，255)，
    team_COLOR=Color3.来自RGB(50，255，100)，
    
    -- 显示设置
    show_HEALTH=true，
    show_DISTANCE=true，
    show_TRACER=true，
    tracer_COLOR=Color3.fromRGB(0，150，255)，
    从RGB(255，50，50)，
    tracer_THICNESS=0.1，
    
    -- 高级设置
    Max_DISTANCE=1000，--已修改为1000单位
    health_TEXT_SIZE=16，
    distance_TEXT_SIZE=14，
    name_TEXT_SIZE=18，
    update_RATE=0.1，
    
    -- 新增设置
    scan_INTERVAL=5，
    tracer_FOR_ENEMIES=true
}

-- ===== 核心功能 =====
本地espCache={}
本地lastScanTime=0

本地函数createESP(目标)
如果不是目标或不是目标：FindFirstChild("HumanoidRootPart")，则返回end
    
    如果espCache[target]，则返回end
    
    本地isEnemy=false
    本地isTeamate=false
    局部espColor=SETTINGS.PAYER_COLOR
    
    对于_，在ipairs中的标记(SETTINGS.ENEMY_TAGS)
    如果target.name：查找(标记)或目标：FindFirstChild(标记)，则
    isEnemy=true
    espColor=SETTINGS.ENEMY_COLOR
    打破
    结束
    结束
    
    如果不是敌人和目标：FindFirstChild(“团队”)，则
    如果是LocalPlayer。团队和目标。团队==LocalPlayer。然后组队
    isTeamate=true
    espColor=SETTINGS.TEAM_COLOR
        end
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "AdvancedESP_Highlight"
    highlight.FillColor = espColor
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.3
    highlight.OutlineTransparency = 0
    highlight.Parent = target
    
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "AdvancedESP_Billboard"
    billboard.Adornee = target.HumanoidRootPart
    billboard.Size = UDim2.new(0, 150, 0, 60)
    billboard.StudsOffset = Vector3.new(0, 3.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = target
    
    local distanceText = Instance.new("TextLabel")
    distanceText.Name = "ESP_Distance"
    distanceText.Size = UDim2.new(1, 0, 0.3, 0)
    distanceText.Position = UDim2.new(0, 0, 0, 0)
    distanceText.BackgroundTransparency = 1
    distanceText.TextColor3 = Color3.fromRGB(200, 200, 255)
    distanceText.TextStrokeTransparency = 0.5
    distanceText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    distanceText.TextSize = SETTINGS.DISTANCE_TEXT_SIZE
    distanceText.Font = Enum.Font.SourceSansBold
    distanceText.TextXAlignment = Enum.TextXAlignment.Center
    distanceText.Parent = billboard
    
    local healthText = Instance.new("TextLabel")
    healthText.Name = "ESP_Health"
    healthText.Size = UDim2.new(1, 0, 0.4, 0)
    healthText.Position = UDim2.new(0, 0, 0.3, 0)
    healthText.BackgroundTransparency = 1
    healthText.TextColor3 = Color3.fromRGB(255, 255, 255)
    healthText.TextStrokeTransparency = 0.5
    healthText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    healthText.TextSize = SETTINGS.HEALTH_TEXT_SIZE
    healthText.Font = Enum.Font.SourceSansBold
    healthText.TextXAlignment = Enum.TextXAlignment.Center
    healthText.Parent = billboard
    
    local nameText = Instance.new("TextLabel")
    nameText.Name = "ESP_Name"
    nameText.Size = UDim2.new(1, 0, 0.3, 0)
    nameText.Position = UDim2.new(0, 0, 0.7, 0)
    nameText.BackgroundTransparency = 1
    nameText.TextColor3 = espColor
    nameText.TextStrokeTransparency = 0.5
    nameText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameText.TextSize = SETTINGS.NAME_TEXT_SIZE
    nameText.Font = Enum.Font.SourceSansBold
    nameText.TextXAlignment = Enum.TextXAlignment.Center
    nameText.Parent = billboard
    
    local tracer
    if SETTINGS.SHOW_TRACER and (not isEnemy or SETTINGS.TRACER_FOR_ENEMIES) then
        local startAttachment = Instance.new("Attachment")
        startAttachment.Name = "TracerStart"
        startAttachment.Parent = workspace.CurrentCamera
        
        local endAttachment = Instance.new("Attachment")
        endAttachment.Name = "TracerEnd"
        endAttachment.Parent = target.HumanoidRootPart
        
        tracer = Instance.new("Beam")
        tracer.Name = "ESP_Tracer"
        tracer.Attachment0 = startAttachment
        tracer.Attachment1 = endAttachment
        tracer.Color = ColorSequence.new(isEnemy and SETTINGS.ENEMY_TRACER_COLOR or SETTINGS.TRACER_COLOR)
        tracer.Width0 = SETTINGS.TRACER_THICKNESS
        tracer.Width1 = SETTINGS.TRACER_THICKNESS * 0.8
        tracer.LightEmission = 0.8
        tracer.Transparency = NumberSequence.new(0.5)
        tracer.FaceCamera = true
        tracer.Parent = target.HumanoidRootPart
    end
    
    espCache[target] = {
        highlight = highlight,
        billboard = billboard,
        tracer = tracer,
        humanoid = target:FindFirstChildOfClass("Humanoid"),
        rootPart = target.HumanoidRootPart,
        startAttachment = tracer and tracer.Attachment0,
        endAttachment = tracer and tracer.Attachment1,
        isEnemy = isEnemy
    }
end

local function fullScan()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            createESP(player.Character)
        end
    end
    
    for _, descendant in ipairs(workspace:GetDescendants()) do
        if descendant:IsA("Model") and descendant:FindFirstChild("HumanoidRootPart") then
            for _, tag in ipairs(SETTINGS.ENEMY_TAGS) do
                if descendant.Name:find(tag) or descendant:FindFirstChild(tag) then
                    createESP(descendant)
                    break
                end
            end
        end
    end
end

-- ===== 更新循环 =====
local lastUpdate = 0
RunService.Heartbeat:Connect(function(deltaTime)
    lastUpdate = lastUpdate + deltaTime
    lastScanTime = lastScanTime + deltaTime
    
    if lastScanTime >= SETTINGS.SCAN_INTERVAL then
        lastScanTime = 0
        fullScan()
    end
    
    if lastUpdate < SETTINGS.UPDATE_RATE then return end
    lastUpdate = 0
    
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    local localRoot = LocalPlayer.Character.HumanoidRootPart
    
    for target, data in pairs(espCache) do
        if not target.Parent then
            if data.highlight then data.highlight:Destroy() end
            if data.billboard then data.billboard:Destroy() end
            if data.tracer then data.tracer:Destroy() end
            espCache[target] = nil
        else
            local distance = (data.rootPart.Position - localRoot.Position).Magnitude
            local shouldShow = distance <= SETTINGS.MAX_DISTANCE
            
            data.highlight.Enabled = shouldShow
            data.billboard.Enabled = shouldShow
            
            if data.tracer then
                data.tracer.Enabled = shouldShow
                if data.startAttachment then
                    data.startAttachment.WorldPosition = localRoot.Position - Vector3.new(0, 1.5, 0)
                end
            end
            
            if shouldShow then
                if SETTINGS.SHOW_DISTANCE then
                    data.billboard.ESP_Distance.Text = string.format("%.1fm", distance/3.571)
                else
                    data.billboard.ESP_Distance.Text = ""
                end
                
                if SETTINGS.SHOW_HEALTH and data.humanoid then
                    data.billboard.ESP_Health.Text = string.format("HP: %d/%d", 
                        math.floor(data.humanoid.Health), 
    math.floor(数据.humanoid.MaxHealth)
    其他
    data.billboard.ESP_Health.Text=""
    结束
                
    data.billboard.ESP_Name.Text=target.Name
    结束
    结束
    结束
结束)

-- ===== 初始化和事件监听 =====
fullScan()

players.PlayerAdded：连接(函数(播放器)
运动员。CharacterAdded:Connect(函数(字符)
createESP(字符)
结束)
结束)

工作区。后代添加：连接(函数(后代)
如果子代：Isa(“模型”)和子代：FindFirstChild(“HumanoidRootPart”)，则
对于_，在ipairs中的标记(SETTINGS.ENEMY_TAGS)
如果descendant.Name:find(标记)或descendant:FindFirstChild(标记)，则
createESP(子代)
打破
结束
结束
结束
结束)

LocalPlayer.Character Removing：连接(功能()
对于_，成对数据(espCache)
如果data.highlight，则data.highlight:Destroy()结束
如果data.billboard，则data.billboard:Destroy()结束
如果data.tracer，则data.tracer:Destroy()结束
结束
espCache={}
结束)
