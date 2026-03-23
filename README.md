-- Grok Fly Hub | Học code + GUI sang chảnh 🐧 (Pastebin load)
local Library = loadstring(game:HttpGet("https://pastebin.com/raw/vff1bQ9F"))()  -- Pastebin mirror ổn định, nếu lỗi thì thử cái dưới

-- Nếu vẫn lỗi load (hiếm), thay bằng GitHub gốc:
-- local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()

local Window = Library.CreateLib("Grok Fly Hub | Học Code 🐧", "DarkTheme")  -- Đổi theme: BloodTheme, Ocean, Midnight cho đẹp

local Tab = Window:NewTab("Controls")
local Section = Tab:NewSection("Fly - Noclip - ESP (Toggle nút dễ dùng trên đt)")

-- Biến toàn cục
local player = game.Players.LocalPlayer
local flying = false
local noclipping = false
local espEnabled = false
local flySpeed = 50

local bodyVelocity, bodyGyro

-- Hàm toggle Fly
local function toggleFly(state)
    flying = state
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    if flying then
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bodyGyro.P = 9e4
        bodyGyro.Parent = root
        
        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.Parent = root
        
        Section:NewLabel("Fly: BẬT | Speed: " .. flySpeed)
    else
        if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
        if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
        Section:NewLabel("Fly: TẮT")
    end
end

-- Toggle Noclip
local function toggleNoclip(state)
    noclipping = state
    Section:NewLabel("Noclip: " .. (noclipping and "BẬT" or "TẮT"))
end

-- Toggle ESP (Highlight + tên + HP)
local function toggleESP(state)
    espEnabled = state
    Section:NewLabel("ESP: " .. (espEnabled and "BẬT" or "TẮT"))
    
    if espEnabled then
        for _, plr in pairs(game.Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local hl = Instance.new("Highlight")
                hl.FillColor = Color3.fromRGB(255, 50, 50)  -- Đỏ
                hl.OutlineColor = Color3.fromRGB(255, 255, 0)  -- Vàng
                hl.FillTransparency = 0.4
                hl.OutlineTransparency = 0
                hl.Parent = plr.Character
                
                local bb = Instance.new("BillboardGui")
                bb.Adornee = plr.Character:FindFirstChild("Head") or plr.Character
                bb.Size = UDim2.new(0, 120, 0, 40)
                bb.StudsOffset = Vector3.new(0, 3, 0)
                bb.AlwaysOnTop = true
                bb.Parent = plr.Character
                
                local txt = Instance.new("TextLabel", bb)
                txt.Size = UDim2.new(1,0,1,0)
                txt.BackgroundTransparency = 1
                txt.TextScaled = true
                txt.Text = plr.Name .. " | HP: " .. math.floor(plr.Character.Humanoid.Health or 100)
                txt.TextColor3 = Color3.new(1,1,1)
            end
        end
    else
        for _, plr in pairs(game.Players:GetPlayers()) do
            if plr.Character then
                if plr.Character:FindFirstChildOfClass("Highlight") then plr.Character:FindFirstChildOfClass("Highlight"):Destroy() end
                if plr.Character:FindFirstChild("BillboardGui", true) then plr.Character:FindFirstChild("BillboardGui", true):Destroy() end
            end
        end
    end
end

-- Loop chạy liên tục (RenderStepped)
game:GetService("RunService").RenderStepped:Connect(function()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart
    
    -- Noclip: tắt va chạm
    if noclipping then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
    
    -- Fly control (hướng camera + input)
    if flying and bodyVelocity and bodyGyro then
        bodyGyro.CFrame = workspace.CurrentCamera.CFrame
        
        local move = Vector3.new()
        local look = workspace.CurrentCamera.CFrame.LookVector
        local right = look:Cross(Vector3.new(0,1,0))
        
        local uis = game:GetService("UserInputService")
        if uis:IsKeyDown(Enum.KeyCode.W) then move = move + look end
        if uis:IsKeyDown(Enum.KeyCode.S) then move = move - look end
        if uis:IsKeyDown(Enum.KeyCode.A) then move = move - right end
        if uis:IsKeyDown(Enum.KeyCode.D) then move = move + right end
        if uis:IsKeyDown(Enum.KeyCode.Space) then move = move + Vector3.new(0,1,0) end
        if uis:IsKeyDown(Enum.KeyCode.LeftControl) then move = move - Vector3.new(0,1,0) end
        
        if move.Magnitude > 0 then
            bodyVelocity.Velocity = move.Unit * flySpeed
        else
            bodyVelocity.Velocity = Vector3.new()
        end
    end
end)

-- GUI nút (dễ học: NewToggle, NewSlider, NewButton)
Section:NewToggle("Fly (Bay)", "Bật/tắt bay", toggleFly)
Section:NewToggle("Noclip (Xuyên tường)", "Bật/tắt xuyên", toggleNoclip)
Section:NewToggle("ESP (Thấy người qua tường)", "Bật/tắt ESP", toggleESP)

Section:NewSlider("Tốc độ Bay", "Chỉnh từ 10 đến 200", 200, 10, function(val)
    flySpeed = val
    Section:NewLabel("Tốc độ hiện tại: " .. val)
end, flySpeed)

Section:NewButton("Refresh ESP", "Cập nhật cho người mới join", function()
    if espEnabled then 
        toggleESP(false) 
        wait(0.1) 
        toggleESP(true) 
    end
end)

print("Grok Fly Hub loaded từ Pastebin! Học code vui nhé, thử chỉnh tốc độ hoặc thêm nút mới đi 🐧")
