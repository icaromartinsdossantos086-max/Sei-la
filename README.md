local Players = game:GetService("Players")
local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

-- Confirma R15
if humanoid.RigType ~= Enum.HumanoidRigType.R15 then
    warn("Apenas R15")
    return
end

-- IDs de animações
local anims = {
    Idle = "rbxassetid://507766388",  -- Idle creepy
    Walk = "rbxassetid://507777826",  -- Walk normal
    Fall = "rbxassetid://507767968",  -- Fall glitch
}

-- Carrega animações
local tracks = {}
for name, id in pairs(anims) do
    local anim = Instance.new("Animation")
    anim.AnimationId = id

    local track = humanoid:LoadAnimation(anim)
    if name == "Idle" then
        track.Priority = Enum.AnimationPriority.Idle
    elseif name == "Walk" then
        track.Priority = Enum.AnimationPriority.Movement
    else
        track.Priority = Enum.AnimationPriority.Action
    end
    track.Looped = true
    tracks[name] = track
end

-- Começa com Idle
tracks.Idle:Play()

-- Ajusta movimento da entidade
humanoid.WalkSpeed = 6
humanoid.JumpPower = 0

-- Troca Idle / Walk
humanoid.Running:Connect(function(speed)
    if speed > 1 then
        if not tracks.Walk.IsPlaying then
            tracks.Idle:Stop()
            tracks.Walk:Play()
        end
    else
        if not tracks.Idle.IsPlaying then
            tracks.Walk:Stop()
            tracks.Idle:Play()
        end
    end
end)

-- Fall glitch aleatório
task.spawn(function()
    while true do
        task.wait(math.random(5,10))
        tracks.Fall:Play()
        task.wait(0.5)
        tracks.Fall:Stop()
    end
end)

-- Idle glitch de velocidade
task.spawn(function()
    while true do
        task.wait(2)
        if tracks.Idle.IsPlaying then
            tracks.Idle:AdjustSpeed(math.random(6,12)/10)
        end
    end
end)

-- Olhar fixo para o player mais próximo
task.spawn(function()
    while true do
        task.wait(0.1)
        local closestDist = math.huge
        local target = nil
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local dist = (root.Position - p.Character.HumanoidRootPart.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    target = p.Character.HumanoidRootPart
                end
            end
        end
        if target then
            root.CFrame = CFrame.new(root.Position, target.Position) * CFrame.Angles(0, math.rad(math.random(-5,5)), 0)
        end
    end
end)

-- Andar arrastando / glitch (move root levemente pra frente)
task.spawn(function()
    while true do
        task.wait(0.2)
        if humanoid.MoveDirection.Magnitude > 0 then
            root.CFrame = root.CFrame + root.CFrame.LookVector * 0.05
        end
    end
end)

-- Pequeno jumpscare visual (pisca / treme corpo)
task.spawn(function()
    while true do
        task.wait(math.random(8,15))
        local originalCFrame = root.CFrame
        for i = 1,3 do
            root.CFrame = root.CFrame * CFrame.new(math.random(-1,1)/10, 0, math.random(-1,1)/10)
            task.wait(0.05)
        end
        root.CFrame = originalCFrame
    end
end)
