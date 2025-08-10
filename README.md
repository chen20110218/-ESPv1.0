#-ESPv1.0-- 高级ESP脚本 v2.2 - 优化显示布局和追踪线
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- ===== 配置区域 =====
local SETTINGS = {
    -- 敌人设置
    ENEMY_TAGS = {"Rake", "Monster", "Enemy"},
    ENEMY_COLOR = Color3.fromRGB(255, 50, 50),
    
    -- 玩家设置
    PLAYER_COLOR = Color3.fromRGB(50, 150, 255),
    TEAM_COLOR = Color3.fromRGB(50, 255, 100),
    
    -- 显示设置
    SHOW_HEALTH = true,
    SHOW_DISTANCE = true,
    SHOW_TRACER = true,
    TRACER_COLOR = Color3.fromRGB(0, 150, 255),
    TRACER_THICKNESS = 0.1,
    
    -- 高级设置
    MAX_DISTANCE = 500,
    HEALTH_TEXT_SIZE = 16,
    DISTANCE_TEXT_SIZE = 14,
    NAME_TEXT_SIZE = 18,
    UPDATE_RATE = 0.1
}

-- ===== 核心功能 =====
local espCache = {}

local function createESP(target)
    if not target:FindFirstChild("HumanoidRootPart") then return end
    
    -- 确定目标类型和颜色
    local isEnemy = false
    local isTeammate = false
    local espColor = SETTINGS.PLAYER_COLOR
    
    for _, tag in ipairs(SETTINGS.ENEMY_TAGS) do
        if target.Name:find(tag) or target:FindFirstChild(tag) then
            isEnemy = true
            espColor = SETTINGS.ENEMY_COLOR
            break
        end
    end
    
    if not isEnemy and target:FindFirstChild("Team") then
        if LocalPlayer.Team and target.Team == LocalPlayer.Team then
            isTeammate = true
            espColor = SETTINGS.TEAM_COLOR
        end
    end
    
    -- 创建高亮效果
    local highlight = Instance.new("Highlight")
    highlight.Name = "AdvancedESP_Highlight"
    highlight.FillColor = espColor
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.3
    highlight.OutlineTransparency = 0
    highlight.Parent = target
    
    -- 创建信息板 (调整为垂直布局)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "AdvancedESP_Billboard"
    billboard.Adornee = target.HumanoidRootPart
    billboard.Size = UDim2.new(0, 150, 0, 60)  -- 调整为更高以适应新布局
    billboard.StudsOffset = Vector3.new(0, 3.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = target
    
    -- 创建距离显示 (顶部)
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
    
    -- 创建血量显示 (中部)
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
    
    -- 创建名字显示 (底部)
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
    
    -- 创建优化的追踪线 (从玩家脚部指向目标)
    local tracer
    if SETTINGS.SHOW_TRACER and (not isEnemy) then
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
        tracer.Color = ColorSequence.new(SETTINGS.TRACER_COLOR)
        tracer.Width0 = SETTINGS.TRACER_THICKNESS
        tracer.Width1 = SETTINGS.TRACER_THICKNESS * 0.8  -- 末端稍细
        tracer.LightEmission = 0.8
        tracer.Transparency = NumberSequence.new(0.5)
        tracer.FaceCamera = true
        tracer.Parent = target.HumanoidRootPart
        
        -- 设置初始位置
        startAttachment.WorldPosition = LocalPlayer.Character and 
                                      LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and 
                                      (LocalPlayer.Character.HumanoidRootPart.Position - Vector3.new(0, 1.5, 0)) or 
                                      Vector3.new(0, 0, 0)
    end
    
    espCache[target] = {
        highlight = highlight,
        billboard = billboard,
        tracer = tracer,
        humanoid = target:FindFirstChildOfClass("Humanoid"),
        rootPart = target.HumanoidRootPart,
        startAttachment = tracer and tracer.Attachment0,
        endAttachment = tracer and tracer.Attachment1
    }
end

-- ===== 更新循环 =====
local lastUpdate = 0
RunService.Heartbeat:Connect(function(deltaTime)
    lastUpdate = lastUpdate + deltaTime
    if lastUpdate < SETTINGS.UPDATE_RATE then return end
    lastUpdate = 0
    
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    local localRoot = LocalPlayer.Character.HumanoidRootPart
    
    for target, data in pairs(espCache) do
        if not target.Parent then
            -- 清理无效目标
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
                -- 更新追踪线起点位置(玩家脚部)
                if data.startAttachment then
                    data.startAttachment.WorldPosition = localRoot.Position - Vector3.new(0, 1.5, 0)
                end
            end
            
            if shouldShow then
                -- 更新距离显示
                if SETTINGS.SHOW_DISTANCE then
                    data.billboard.ESP_Distance.Text = string.format("%.1fm", distance/3.571)  -- 转换为米
                else
                    data.billboard.ESP_Distance.Text = ""
                end
                
                -- 更新血量
                if SETTINGS.SHOW_HEALTH and data.humanoid then
                    data.billboard.ESP_Health.Text = string.format("HP: %d/%d", 
                        math.floor(data.humanoid.Health), 
                        math.floor(data.humanoid.MaxHealth))
                else
                    data.billboard.ESP_Health.Text = ""
                end
                
                -- 更新名字
                data.billboard.ESP_Name.Text = target.Name
            end
        end
    end
end)

-- ===== 初始化和事件监听 =====
-- 扫描现有玩家
for _, player in ipairs(Players:GetPlayers()) do
    if player.Character then
        createESP(player.Character)
    end
    player.CharacterAdded:Connect(function(character)
        createESP(character)
    end)
end

-- 扫描NPC/怪物
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

-- 监听新NPC/怪物
workspace.DescendantAdded:Connect(function(descendant)
    if descendant:IsA("Model") and descendant:FindFirstChild("HumanoidRootPart") then
        for _, tag in ipairs(SETTINGS.ENEMY_TAGS) do
            if descendant.Name:find(tag) or descendant:FindFirstChild(tag) then
                createESP(descendant)
                break
            end
        end
    end
end)

-- 清理脚本
LocalPlayer.CharacterRemoving:Connect(function()
    for _, data in pairs(espCache) do
        if data.highlight then data.highlight:Destroy() end
        if data.billboard then data.billboard:Destroy() end
        if data.tracer then data.tracer:Destroy() end
    end
    espCache = {}
end)
