local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Ultra Toilet Fight 2 beta by miko",
   LoadingTitle = "by miko This is just a beta version",
   LoadingSubtitle = "by miko dev",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ProximityPromptService = game:GetService("ProximityPromptService")
local LocalPlayer = Players.LocalPlayer

-- Biến điều khiển Auto Farm
local autoFarm = false
local autoLoot = false
local expandHitbox = false
local hitboxSize = 50
local maxTpDistance = 5000
local farmDistance = 7 -- Khoảng cách farm
local farmPosition = "Phía Trên" -- Phía Trên, Dưới Lòng Đất, Phía Trước, Phía Sau

local currentTarget = nil
local isCollectingLoot = false
local blacklistedTargets = {}

-- Biến phím kỹ năng
local useLMB = false
local useKeyE = false
local useKeyQ = false
local useKeyR = false
local useKeyF = false

-- ===================================================
-- HÀM LƯỢM ĐỒ SIÊU TỐC
-- ===================================================
local function InstantCollectPrompt(prompt)
    if not prompt or not prompt:IsA("ProximityPrompt") or not prompt.Enabled then return end
    
    pcall(function()
        prompt.RequiresLineOfSight = false
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 9999
        
        if fireproximityprompt then
            fireproximityprompt(prompt)
        else
            prompt:InputHoldBegin()
            task.wait()
            prompt:InputHoldEnd()
        end
    end)
end

-- Tự động nhặt đồ ngay khi xuất hiện
ProximityPromptService.PromptShown:Connect(function(prompt)
    if autoLoot or isCollectingLoot then
        InstantCollectPrompt(prompt)
    end
end)

-- KIỂM TRA XEM CÓ ĐỒ RƠI RA QUANH VỊ TRÍ QUÁI CHẾT HAY KHÔNG
local function GetNearbyLootPrompt(pos, maxDist)
    maxDist = maxDist or 25
    for _, v in ipairs(Workspace:GetChildren()) do
        if v:IsA("ProximityPrompt") and v.Parent and v.Parent:IsA("BasePart") then
            if (v.Parent.Position - pos).Magnitude <= maxDist then
                return v
            end
        elseif v:IsA("Model") or v:IsA("Folder") then
            for _, child in ipairs(v:GetChildren()) do
                if child:IsA("ProximityPrompt") and child.Parent and child.Parent:IsA("BasePart") then
                    if (child.Parent.Position - pos).Magnitude <= maxDist then
                        return child
                    end
                end
            end
        end
    end
    return nil
end

-- ===================================================
-- HÀM LỌC QUÁI TỐI ƯU TỐC ĐỘ (CHỐNG LAG)
-- ===================================================
local function IsEnemy(model)
    if not model or not model:IsA("Model") then return false end
    if model == LocalPlayer.Character then return false end
    if Players:GetPlayerFromCharacter(model) then return false end
    if blacklistedTargets[model] then return false end

    local hum = model:FindFirstChildOfClass("Humanoid")
    if not hum or hum.Health <= 0 then return false end
    
    local root = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Head") or model.PrimaryPart
    if not root or root.Anchored then return false end

    local nameLower = model.Name:lower()
    if string.find(nameLower, "shop") 
    or string.find(nameLower, "display") 
    or string.find(nameLower, "showcase") 
    or string.find(nameLower, "lobby") 
    or string.find(nameLower, "store") 
    or string.find(nameLower, "building") 
    or string.find(nameLower, "glass") 
    or string.find(nameLower, "npc") then
        return false
    end

    return true
end

-- TÌM QUÁI GẦN NHẤT
local function GetNearestEnemy()
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end

    local myPos = myChar.HumanoidRootPart.Position
    local nearest = nil
    local shortestDistance = maxTpDistance

    local function CheckModel(v)
        if v:IsA("Model") and IsEnemy(v) then
            local enemyRoot = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Head") or v.PrimaryPart
            if enemyRoot then
                local dist = (myPos - enemyRoot.Position).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    nearest = enemyRoot
                end
            end
        end
    end

    for _, v in ipairs(Workspace:GetChildren()) do
        CheckModel(v)
        if v:IsA("Folder") or (v:IsA("Model") and not IsEnemy(v)) then
            for _, child in ipairs(v:GetChildren()) do
                CheckModel(child)
            end
        end
    end

    return nearest
end

-- ===================================================
-- TAB AUTO FARM & LOOT
-- ===================================================
local CombatTab = Window:CreateTab("Auto Farm & Loot", 4483362458)

CombatTab:CreateSection("1. Cấu Hình Vị Trí Farm & Auto Farm")

CombatTab:CreateToggle({
   Name = "Bật Auto Farm",
   CurrentValue = false,
   Flag = "AutoFarmToggle",
   Callback = function(Value)
      autoFarm = Value
      if not Value then 
          currentTarget = nil 
          isCollectingLoot = false
      end
   end,
})

CombatTab:CreateDropdown({
   Name = "Vị Trí Đứng Farm Quái",
   Options = {"Phía Trên", "Dưới Lòng Đất", "Phía Trước", "Phía Sau"},
   CurrentOption = {"Phía Trên"},
   MultipleOptions = false,
   Flag = "FarmPosDropdown",
   Callback = function(Option)
      if type(Option) == "table" then
          farmPosition = Option[1]
      else
          farmPosition = Option
      end
   end,
})

CombatTab:CreateSlider({
   Name = "Khoảng Cách Farm (Studs)",
   Range = {2, 30},
   Increment = 1,
   Suffix = "Studs",
   CurrentValue = 7,
   Flag = "FarmDistSlider",
   Callback = function(Value) farmDistance = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Lượm Đồ (Chỉ Nhảy Khi Có Đồ Rơi)",
   CurrentValue = false,
   Flag = "AutoLootToggle",
   Callback = function(Value)
      autoLoot = Value
   end,
})

CombatTab:CreateSlider({
   Name = "Tầm Quét Tìm Quái Wave",
   Range = {100, 10000},
   Increment = 100,
   Suffix = "Studs",
   CurrentValue = 5000,
   Flag = "MaxDistSlider",
   Callback = function(Value) maxTpDistance = Value end,
})

CombatTab:CreateButton({
   Name = "⚠️ Bỏ Qua Quái Kẹt Hiện Tại",
   Callback = function()
      if currentTarget and currentTarget.Parent then
          blacklistedTargets[currentTarget.Parent] = true
          currentTarget = nil
          Rayfield:Notify({ Title = "Combat", Content = "Đã bỏ qua quái kẹt!", Duration = 2 })
      end
   end,
})

CombatTab:CreateSection("2. Phím Skill Tự Động Spam")

CombatTab:CreateToggle({
   Name = "Auto Click Chuột Trái (LMB)",
   CurrentValue = false,
   Flag = "LmbToggle",
   Callback = function(Value) useLMB = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Phím [ E ]",
   CurrentValue = false,
   Flag = "KeyEToggle",
   Callback = function(Value) useKeyE = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Phím [ Q ]",
   CurrentValue = false,
   Flag = "KeyQToggle",
   Callback = function(Value) useKeyQ = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Phím [ R ]",
   CurrentValue = false,
   Flag = "KeyRToggle",
   Callback = function(Value) useKeyR = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Phím [ F ]",
   CurrentValue = false,
   Flag = "KeyFToggle",
   Callback = function(Value) useKeyF = Value end,
})

CombatTab:CreateSection("3. Hitbox Phóng To (Max 100)")

CombatTab:CreateToggle({
   Name = "Phóng To Hitbox Quái",
   CurrentValue = false,
   Flag = "HitboxToggle",
   Callback = function(Value) expandHitbox = Value end,
})

CombatTab:CreateSlider({
   Name = "Kích Thước Hitbox",
   Range = {3, 100},
   Increment = 1,
   Suffix = "Size",
   CurrentValue = 50,
   Flag = "HitboxSizeSlider",
   Callback = function(Value) hitboxSize = Value end,
})

-- ===================================================
-- LOGIC BACKEND TỐI ƯU SIÊU MƯỢT
-- ===================================================

-- 1. LOGIC AUTO FARM & CHUYỂN MỤC TIÊU LƯỢM ĐỒ CÓ KIỂM TRA VẬT PHẨM
task.spawn(function()
    while task.wait(0.15) do
        if autoFarm and not isCollectingLoot then
            pcall(function()
                local isTargetAlive = currentTarget 
                    and currentTarget.Parent 
                    and currentTarget.Parent:FindFirstChildOfClass("Humanoid") 
                    and currentTarget.Parent:FindFirstChildOfClass("Humanoid").Health > 0

                if not isTargetAlive then
                    if currentTarget then
                        local deathPos = currentTarget.Position
                        currentTarget = nil

                        -- CHỈ LƯỢM KHI CÓ VẬT PHẨM/PROXIMITY PROMPT THỰC SỰ RƠI RA
                        if autoLoot then
                            local dropPrompt = GetNearbyLootPrompt(deathPos, 25)
                            if dropPrompt then
                                isCollectingLoot = true
                                local lootPart = dropPrompt.Parent
                                local startTime = tick()
                                
                                while tick() - startTime < 0.35 and dropPrompt and dropPrompt.Parent and autoFarm do
                                    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                                    if root and lootPart then
                                        root.CFrame = lootPart.CFrame * CFrame.new(0, 1.5, 0)
                                        root.AssemblyLinearVelocity = Vector3.zero
                                        InstantCollectPrompt(dropPrompt)
                                    else
                                        break
                                    end
                                    task.wait(0.05)
                                end
                                isCollectingLoot = false
                            end
                        end
                    end

                    -- Chuyển ngay mục tiêu sang quái mới mà không dừng lại
                    currentTarget = GetNearestEnemy()
                end
            end)
        end
    end
end)

-- 2. TÍNH TOÁN CFRAME THEO VỊ TRÍ ĐÃ CHỌN
local function GetTargetCFrame(targetRoot)
    local targetCF = targetRoot.CFrame
    if farmPosition == "Phía Trên" then
        local pos = targetCF * CFrame.new(0, farmDistance, 0)
        return CFrame.new(pos.Position, targetRoot.Position)
    elseif farmPosition == "Dưới Lòng Đất" then
        local pos = targetCF * CFrame.new(0, -farmDistance, 0)
        return CFrame.new(pos.Position, targetRoot.Position)
    elseif farmPosition == "Phía Trước" then
        local pos = targetCF * CFrame.new(0, 0, -farmDistance)
        return CFrame.new(pos.Position, targetRoot.Position)
    elseif farmPosition == "Phía Sau" then
        local pos = targetCF * CFrame.new(0, 0, farmDistance)
        return CFrame.new(pos.Position, targetRoot.Position)
    end
    return targetCF * CFrame.new(0, farmDistance, 0)
end

-- 3. TELEPORT ĐẾN VỊ TRÍ QUÁI (RenderStepped)
RunService.RenderStepped:Connect(function()
    if autoFarm and not isCollectingLoot and currentTarget and currentTarget.Parent then
        local hum = currentTarget.Parent:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 then
            pcall(function()
                local myChar = LocalPlayer.Character
                if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                    local myRoot = myChar.HumanoidRootPart
                    myRoot.CFrame = GetTargetCFrame(currentTarget)
                    myRoot.AssemblyLinearVelocity = Vector3.zero
                end
            end)
        end
    end
end)

-- 4. SPAM PHÍM SKILL (DẠNG NHẤP NHẢ)
task.spawn(function()
    while task.wait(0.1) do
        if autoFarm and not isCollectingLoot then
            pcall(function()
                if useLMB then
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                    task.wait(0.01)
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                end
                if useKeyE then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
                end
                if useKeyQ then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Q, false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Q, false, game)
                end
                if useKeyR then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.R, false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.R, false, game)
                end
                if useKeyF then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
                end
            end)
        end
    end
end)

-- 5. HITBOX TỐI ƯU TỐC ĐỘ (CHỐNG LAG)
task.spawn(function()
    while task.wait(1) do
        if expandHitbox then
            pcall(function()
                local function ApplyHitbox(v)
                    if v:IsA("Model") and IsEnemy(v) then
                        local enemyRoot = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Head")
                        if enemyRoot then
                            if enemyRoot.Size ~= Vector3.new(hitboxSize, hitboxSize, hitboxSize) then
                                enemyRoot.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
                            end
                            if not enemyRoot:GetAttribute("HitboxResized") then
                                enemyRoot:SetAttribute("HitboxResized", true)
                                enemyRoot.Transparency = 0.7
                                enemyRoot.BrickColor = BrickColor.new("Really blue")
                                enemyRoot.Material = Enum.Material.Neon
                                enemyRoot.CanCollide = false
                                enemyRoot.CanTouch = false
                            end
                        end
                    end
                end

                for _, v in ipairs(Workspace:GetChildren()) do
                    ApplyHitbox(v)
                    if v:IsA("Folder") then
                        for _, child in ipairs(v:GetChildren()) do
                            ApplyHitbox(child)
                        end
                    end
                end
            end)
        end
    end
end)

Rayfield:Notify({
   Title = "Ultra Toilet Fight 2",
   Content = "beta!",
   Duration = 4,
   Image = 4483362458,
})
