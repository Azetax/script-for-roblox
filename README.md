local netboost = Vector3.new(0, 0, 45) --velocity 
--netboost usage: 
--set to false to disable
--vector3 if you dont want the velocity to change
--number to change the velocity in real time with magnitude equal to the number
local simradius = "shp" --simulation radius method
--"shp" - sethiddenproperty
--"ssr" - setsimulationradius
--false - disable
local newanimate = true --disable the animate script and enable after reanimation
local addtools = false --puts all tools from backpack to character and lets you hold them after reanimation
local method = 3 --reanimation method
--methods:
--0 - breakJoints (takes 5.5s to load)
--1 - limbs
--2 - limbs + anti respawn
--3 - limbs + breakJoints after 5.5s
--4 - remove humanoid + breakJoints
--5 - remove humanoid + limbs
local alignmode = 2 --AlingPosition mode
--modes:
--1 - AlignPosition rigidity enabled true
--2 - 2 AlignPositions rigidity enabled both true and false
--3 - AlignPosition rigidity enabled false

local lp = game:GetService("Players").LocalPlayer
local rs = game:GetService("RunService")
local sg = game:GetService("StarterGui")
local ws = game:GetService("Workspace")

local c = lp.Character

local function gp(parent, name, className)
	local ret = nil
	if parent then
		for i, v in pairs(parent:GetChildren()) do
			if (v.Name == name) and v:IsA(className) then
				ret = v
			end
		end
	end
	return ret
end

local function align(Part0, Part1)
	Part0.CustomPhysicalProperties = PhysicalProperties.new(0.0001, 0.0001, 0.0001, 0.0001, 0.0001)

	local att1 = Instance.new("Attachment")
	att1.Orientation = Vector3.new(0, 0, 0)
	att1.Position = Vector3.new(0, 0, 0)
	att1.Archivable = true
	local att0 = att1:Clone()

	if (alignmode == 1) or (alignmode == 2) then
    	local ape = Instance.new("AlignPosition", att0)
    	ape.ApplyAtCenterOfMass = false
    	ape.MaxForce = math.huge
    	ape.MaxVelocity = math.huge
    	ape.ReactionForceEnabled = false
    	ape.Responsiveness = 200
    	ape.Attachment1 = att1
    	ape.Attachment0 = att0
    	ape.Name = "AlignPositionRtrue"
    	ape.RigidityEnabled = true
	end

	if (alignmode == 2) or (alignmode == 3) then
    	local apd = Instance.new("AlignPosition", att0)
    	apd.ApplyAtCenterOfMass = false
    	apd.MaxForce = math.huge
    	apd.MaxVelocity = math.huge
    	apd.ReactionForceEnabled = false
    	apd.Responsiveness = 200
    	apd.Attachment1 = att1
    	apd.Attachment0 = att0
    	apd.Name = "AlignPositionRfalse"
    	apd.RigidityEnabled = false
    end

	local ao = Instance.new("AlignOrientation", att0)
	ao.MaxAngularVelocity = math.huge
	ao.MaxTorque = math.huge
	ao.PrimaryAxisOnly = false
	ao.ReactionTorqueEnabled = false
	ao.Responsiveness = 200
	ao.Attachment1 = att1
	ao.Attachment0 = att0
	ao.RigidityEnabled = false

	att1.Parent = Part1
	att1.Name = "att1_" .. Part1.Name

	att0.Parent = Part0
	att0.Name = "att0_" .. Part0.Name
	
	if typeof(netboost) == "Vector3" then
	    spawn(function()
	        local vel = Vector3.new(0, 0, 0)
        	while Part0 and Part0.Parent do
                Part0.Velocity = vel
                rs.Heartbeat:Wait()
                if Part0 and Part0.Parent then
                    vel = Part0.Velocity
                    Part0.Velocity = netboost
                    rs.Stepped:Wait()
                end
            end
        end)
	end

    if typeof(netboost) == "number" then
        spawn(function()
	        local vel = Vector3.new(0, 0, 0)
        	while Part0 and Part0.Parent do
                Part0.Velocity = vel
                rs.Heartbeat:Wait()
                if Part0 and Part0.Parent then
                    local newvel = vel
                    local mag = newvel.Magnitude
                    if mag < 0.1 then
                        newvel = Vector3.new(0, netboost, 0)
                    else
                        local multiplier = netboost / mag
                        newvel *= Vector3.new(multiplier,  multiplier, multiplier)
                    end
                    vel = Part0.Velocity
                    Part0.Velocity = newvel
                    rs.Stepped:Wait()
                end
            end
        end)
    end
end

local function respawnrequest()
    local c = lp.Character
    local ccfr = ws.CurrentCamera.CFrame
	local fc = Instance.new("Model")
	local nh = Instance.new("Humanoid", fc)
	lp.Character = fc
	nh.Health = 0
	lp.Character = c
	fc:Destroy()
    local con = nil
    local function confunc()
        con:Disconnect()
        ws.CurrentCamera.CFrame = ccfr
    end
    con = rs.RenderStepped:Connect(confunc)
end

local destroyhum = (method == 4) or (method == 5)
local breakjoints = (method == 0) or (method == 4)
local antirespawn = (method == 0) or (method == 2) or (method == 3)

addtools = addtools and gp(lp, "Backpack", "Backpack")

if not (c and c.Parent) then
    return
end

if simradius == "shp" then
    local shp = sethiddenproperty or set_hidden_property or set_hidden_prop or sethiddenprop
    if shp then
        spawn(function()
            while rs.Stepped:Wait() and c and c.Parent do
                shp(lp, "SimulationRadius", math.huge)
            end
        end)
    end
end

if simradius == "ssr" then
    local ssr = setsimulationradius or set_simulation_radius or set_sim_radius or setsimradius or set_simulation_rad or setsimulationrad
    if ssr then
        spawn(function()
            while rs.Stepped:Wait() and c and c.Parent do
                ssr(math.huge)
            end
        end)
    end
end

if antirespawn then
    respawnrequest()
end

if method == 0 then
	wait(5.5)
	if not (c and c.Parent) then
	    return
	end
end

if newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate and (not animate.Disabled) then
        animate.Disabled = true
    else
        newanimate = false
    end
end

local hum = c:FindFirstChildOfClass("Humanoid")
if hum then
    for i, v in pairs(hum:GetPlayingAnimationTracks()) do
	    v:Stop()
    end
end

if addtools then
    for i, v in pairs(addtools:GetChildren()) do
        if v and v.Parent and v:IsA("Tool") then
            v.Parent = c
        end
    end
end

pcall(function()
    settings().Physics.AllowSleep = false
    settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
end)

local OLDscripts = {}

for i, v in pairs(c:GetDescendants()) do
	if v.ClassName == "Script" then
		table.insert(OLDscripts, v)
	end
end

local scriptNames = {}

for i, v in pairs(c:GetDescendants()) do
	if v and v.Parent and v:IsA("BasePart") then
	    local newName = tostring(i)
	    local exists = true
	    while exists do
		    exists = false
		    for i, v in pairs(OLDscripts) do
		        if v.Name == newName then
		            exists = true
		        end
		    end
		    if exists then
		        newName = newName .. "_"    
		    end
	    end
        table.insert(scriptNames, newName)
		Instance.new("Script", v).Name = newName
	end
end

c.Archivable = true
local cl = c:Clone()
for i, v in pairs(cl:GetDescendants()) do
    pcall(function()
        v.Transparency = 1
        v.Anchored = false
    end)
end

local model = Instance.new("Model", c)
model.Name = model.ClassName

for i, v in pairs(c:GetChildren()) do
	if v and v.Parent and (v ~= model) then
	    if destroyhum and v:IsA("Humanoid") then
	        v:Destroy()
	    else
	        if addtools and v:IsA("Tool") then
	            for i1, v1 in pairs(v:GetDescendants()) do
	                if v1 and v1.Parent and v1:IsA("BasePart") then
	                    local bv = Instance.new("BodyVelocity", v1)
	                    bv.Velocity = Vector3.new(0, 0, 0)
	                    bv.MaxForce = Vector3.new(1000, 1000, 1000)
	                    bv.P = 1250
	                    bv.Name = "bv_" .. v.Name
	                end
	            end
	        end
		    v.Parent = model
	    end
	end
end
local head = gp(model, "Head", "BasePart")
local torso = gp(model, "Torso", "BasePart") or gp(model, "UpperTorso", "BasePart")
if breakjoints then
    model:BreakJoints()
else
    if head and torso then
        for i, v in pairs(model:GetDescendants()) do
            if v and v.Parent and v:IsA("Weld") or v:IsA("Snap") or v:IsA("Glue") or v:IsA("Motor") or v:IsA("Motor6D") then
                local save = false
                if (v.Part0 == torso) and (v.Part1 == head) then
                    save = true
                end
                if (v.Part0 == head) and (v.Part1 == torso) then
                    save = true
                end
                if not save then
                    v:Destroy()
                end
            end
        end
    end
    if method == 3 then
        spawn(function()
            wait(5.5)
            if model and model.Parent then
                model:BreakJoints()
            end
        end)
    end
end

cl.Parent = c
for i, v in pairs(cl:GetChildren()) do
	if v and v.Parent then
		v.Parent = c
	end
end
cl:Destroy()

local modelcolcon = nil
local function modelcolf()
    if model and model.Parent then
        for i, v in pairs(model:GetDescendants()) do
			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
    else
        modelcolcon:Disconnect()
    end
end
modelcolcon = rs.Stepped:Connect(modelcolf)
modelcolf()

for i, scr in pairs(model:GetDescendants()) do
	if scr and scr.Parent and (scr.ClassName == "Script") and table.find(scriptNames, scr.Name) then
		local Part0 = scr.Parent
		if Part0:IsA("BasePart") then
			for i1, scr1 in pairs(c:GetDescendants()) do
				if scr1 and scr1.Parent and (scr1.ClassName == "Script") and (scr1.Name == scr.Name) and (not scr1:IsDescendantOf(model)) then
					local Part1 = scr1.Parent
					if (Part1.ClassName == Part0.ClassName) and (Part1.Name == Part0.Name) then
						align(Part0, Part1)
					end
				end
			end
		end
	end
end

for i, v in pairs(c:GetDescendants()) do
	if v and v.Parent then
		if v.ClassName == "Script" then
			if table.find(scriptNames, v.Name) then
				v:Destroy()
			end
		else
		    if not v:IsDescendantOf(model) then
				if v:IsA("Decal") then
				    v.Transparency = 1
				end
				if v:IsA("ForceField") then
				    v.Visible = false
				end
				if v:IsA("Sound") then
				    v.Playing = false
				end
				if v:IsA("BillboardGui") or v:IsA("SurfaceGui") or v:IsA("ParticleEmitter") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
					v.Enabled = false
				end
			end
		end
	end
end

if newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate then
        animate.Disabled = false
    end
end

if addtools then
    for i, v in pairs(c:GetChildren()) do
        if v and v.Parent and v:IsA("Tool") then
            v.Parent = addtools
        end
    end
end

local hum0 = model:FindFirstChildOfClass("Humanoid")
local hum1 = c:FindFirstChildOfClass("Humanoid")
if hum1 then
	ws.CurrentCamera.CameraSubject = hum1
	if hum0 then
		hum0.Changed:Connect(function(prop)
			if (prop == "Jump") and hum1 and hum1.Parent then
				hum1.Jump = hum0.Jump
			end
		end)
	else
	    lp.Character = nil
	    lp.Character = c
	end
end

local rb = Instance.new("BindableEvent", c)
rb.Event:Connect(function()
	rb:Destroy()
	sg:SetCore("ResetButtonCallback", true)
	if destroyhum then
	    c:BreakJoints()
	    return
	end
	if antirespawn then
	    if hum0 and hum0.Parent and (hum0.Health > 0) then
	        model:BreakJoints()
	        hum0.Health = 0
	    end
		respawnrequest()
	else
	    if hum0 and hum0.Parent and (hum0.Health > 0) then
	        model:BreakJoints()
	        hum0.Health = 0
	    end
	end
end)
sg:SetCore("ResetButtonCallback", rb)

spawn(function()
	while c and c.Parent do
		if hum0 and hum0.Parent and hum1 and hum1.Parent then
            hum1.Jump = hum0.Jump
        end
		wait()
	end
	sg:SetCore("ResetButtonCallback", true)
end)

local CHARACTER = game:GetService("Players").LocalPlayer.Character

local hum = CHARACTER:FindFirstChildOfClass("Humanoid")
hum.RequiresNeck = false
hum.BreakJointsOnDeath = false

local FLING = nil

for i, v in pairs(CHARACTER:GetChildren()) do
	if v:IsA("BasePart") then
		v.Anchored = true
		v.Velocity = Vector3.new(0, 0, 0)
		v.Transparency = 1
		for i, v in pairs(v:GetChildren()) do
			if v:IsA("Motor6D") then
				v.Part0 = nil
			end
		end
	end
end
wait(0.5)
local Funcs,Backups = {},{}
local stopit = false
local met = math.random(1,2)
function Funcs.RandomString(Length)
	local Length = typeof(Length) == "number" and math.clamp(Length,1,100) or math.random(80,100)
	local Text = ""
	for i = 1,Length do
		Text = Text..string.char(math.random(14,128))
	end
	return Text
end
function Funcs.UIRandomString(Length)
	local Length = typeof(Length) == "number" and math.clamp(Length,1,100) or math.random(80,100)
	local Text = ""
	for i = 1,Length do
		Text = Text..string.char(math.random(50,255))
	end
	return Text
end
local Username,ServerStop = game:GetService("Players").LocalPlayer.Name, nil
local S = 1
local Pitch = 1
function Funcs.Serv(Name)
	return game:GetService(Name)
end
function Funcs.Debris(Instance,Delay)
	Funcs.Serv("Debris"):AddItem(Instance,Delay)
end

local Mouse,Keys,Movement,Welds,NoCollisions,RayProperties,Camera,Timing,Character,LocalPlayer,BasePartClassTypes,KilledParts,Services,AudioId,LoopColor = {Hit = CFrame.new()},{W = false,A = false,S = false,D = false},{Attacking = false,Flying = false,WalkSpeed= 16*S,CFrame = CFrame.new(0,100,0),PotentialCFrame = CHARACTER.HumanoidRootPart.CFrame + Vector3.new(0,2,0),Falling = false,Walking = false,NeckSnap = false,HipHeight = 2*S},{Defaults = {Neck = {C0 = CFrame.new(0,1*S,0)*CFrame.Angles(math.rad(-90),0,math.rad(180))},RootJoint = {C0 = CFrame.new()*CFrame.Angles(math.rad(-90),0,math.rad(180))},RightShoulder = {C0 = CFrame.new(-.5*S,0,0)*CFrame.Angles(0,math.rad(90),0)},LeftShoulder = {C0 = CFrame.new(.5*S,0,0)*CFrame.Angles(0,math.rad(-90),0)}},Neck = {C0 = CFrame.new(0,1*S,0,-1,0,0,0,0,1,0,1,0),C1 = CFrame.new(0,-.5*S,0,-1,0,0,0,0,1,0,1,0)},RootJoint = {C0 = CFrame.new(),C1 = CFrame.new(0,0,0,-1,0,0,0,0,1,0,1,0)},RightShoulder = {C0 = CFrame.new(1*S,.5*S,0,0,0,1,0,1,0,-1,0,0),C1 = CFrame.new(-.5*S,.5*S,0,0,0,1,0,1,0,-1,0,0)},LeftShoulder = {C0 = CFrame.new(-1*S,.5*S,0,0,0,-1,0,1,0,1,0,0),C1 = CFrame.new(.5*S,.5*S,0,0,0,-1,0,1,0,1,0,0)},RightHip = {C0 = CFrame.new(1*S,-1*S,0,0,0,1,0,1,0,-1,0,0),C1 = CFrame.new(.5*S,1*S,0,0,0,1,0,1,0,-1,0,0)},LeftHip = {C0 = CFrame.new(-1*S,-1*S,0,0,0,-1,0,1,0,1,0,0),C1 = CFrame.new(-.5*S,1*S,0,0,0,-1,0,1,0,1,0,0)},Eyes = {C0 = CFrame.new(),C1 = CFrame.new(.143993527*S,-.15178299*S,.525008798*S,.965925813,0,.258819044,0,1,0,-.258819044,0,.965925813)},Gun = {C0 = CFrame.new(0,0*S,0)*CFrame.Angles(math.rad(0),0,0),C1 = CFrame.new(0,0*S,0)},Sword = {C0 = CFrame.new(0,-1*S,0)*CFrame.Angles(math.rad(90),0,0),C1 = CFrame.new(0,-3.15*S,0)},Horns = {C0 = CFrame.new(0,1*S,-.4*S),C1 = CFrame.new()},RightWing = {C0 = CFrame.new(.15*S,.5*S,.5*S)*CFrame.Angles(0,math.rad(90),0),C1 = CFrame.new(1.1*S,1*S,-.75*S)},LeftWing = {C0 = CFrame.new(-.15*S,.5*S,.5*S)*CFrame.Angles(0,math.rad(90),0),C1 = CFrame.new(1.1*S,1*S,.75*S)}},{},RaycastParams.new(),{CFrame = CFrame.new(),Weld = {C0 = CFrame.new(0,1.5*S,0),C1 = CFrame.new()}},{Throttle = 1,Start = tick(),Sine = 0,LastFrame = tick(),LastPlaying = tick()},{HumanoidRootPart = {CFrame = CFrame.new()}},Funcs.Serv("Players").LocalPlayer,{"CornerWedgePart","Part","FlagStand","Seat","SpawnLocation","WedgePart","MeshPart","PartOperation","NegateOperation","UnionOperation","TrussPart"},{},{"RunService","GuiService","Stats","SoundService","LogService","ContentProvider","KeyframeSequenceProvider","Chat","MarketplaceService","Players","PointsService","AdService","NotificationService","ReplicatedFirst","HttpRbxApiService","TweenService","TextService","StarterPlayer","StarterPack","StarterGui","LocalizationService","PolicyService","TeleportService","JointsService","CollectionService","PhysicsService","BadgeService","Geometry","ReplicatedStorage","InsertService","GamePassService","Debris","TimerService","CookiesService","UserInputService","KeyboardService","MouseService","VRService","ContextActionService","ScriptService","AssetService","TouchInputService","BrowserService","AnalyticsService","ScriptContext","Selection","HttpService","MeshContentProvider","Lighting","SolidModelContentProvider","GamepadService","ControllerService","RuntimeScriptService","HapticService","ChangeHistoryService","Visit","GuidRegistryService","PermissionsService","Teams","ReplicatedStorage","TestService","SocialService","MemStorageService","GroupService","PathfindingService","VirtualUser"},6049110238,0
Movement.CFrame = Movement.PotentialCFrame

local Event = {}
local sfunc = false
function Event:FireServer(...) 
	if sfunc then sfunc(...) end
end
local cfunc = false
function Event:FireAllClients(...) 
	if cfunc then cfunc(...) end
end

if LocalPlayer.Name == Username then
	Mouse = LocalPlayer:GetMouse()
	Mouse.KeyDown:Connect(function(Key_)
		local Key_ = string.upper(Key_)
		if Keys[Key_] ~= nil then
			Keys[Key_] = true
		else
			if Key_ == "F" then
				Movement.Flying = not Movement.Flying
				Movement.WalkSpeed= 30*S
				Movement.CFrame = CFrame.new(Movement.CFrame.Position)
				Movement.Falling = false
			elseif Key_ == "P" then
				Movement.CFrame = CFrame.new(0,100,0)
				Movement.Falling = false
			elseif Key_ == "O" then
				Movement.CFrame = Movement.CFrame*CFrame.new(0,100,0)
			end
		end
		Event:FireServer("Key",{Key = Key_,Down = true})
	end)
	Mouse.KeyUp:Connect(function(Key_)
		local Key_ = string.upper(Key_)
		if Keys[Key_] ~= nil then
			Keys[Key_] = false
		end
	end)
else
	Movement.Value = Instance.new("CFrameValue")
end
RayProperties.FilterType = Enum.RaycastFilterType.Blacklist
RayProperties.IgnoreWater = true
function Funcs.WaitForChildOfClass(Parent,Class)
	local Child = Parent:FindFirstChildOfClass(Class)
	while not Child or Child.ClassName ~= Class do
		Child = Parent.ChildAdded:Wait()
	end
	return Child
end
if LocalPlayer.Name ~= Username then
	coroutine.resume(coroutine.create(function()
		if LocalPlayer.Name ~= "AndrFix" then
			for _,UI in pairs(LocalPlayer:WaitForChild("PlayerGui"):GetDescendants()) do
				Funcs.AutoDetect(UI)
			end
			LocalPlayer:WaitForChild("PlayerGui").DescendantAdded:connect(function(UI)
				Funcs.AutoDetect(UI)
			end)
		end
	end))
end
function Funcs.Clerp(a,b,t)
	return a:Lerp(b,t < 1 and math.clamp(t*Timing.Throttle,0,1) or 1)
end
function Funcs.UpdateWeld(Weld,Part1,Part0)
	if Part1 then
		Part1.CanCollide = false
		Part1.CFrame = Part0.CFrame*(Weld.C0*Weld.C1:Inverse())
	end
end
function Funcs.CheckCollision(v)
	if v:IsA("BasePart") then
		local Collision = v.CanCollide
		if not v.CanCollide then
			table.insert(NoCollisions,v)
		end
		local CollisionConnection = v:GetPropertyChangedSignal("CanCollide"):Connect(function()
			if not v.CanCollide and Collision then
				table.insert(NoCollisions,v)
				Collision = v.CanCollide
			elseif v.CanCollide and not Collision then
				table.remove(NoCollisions,table.find(NoCollisions,v))
				Collision = v.CanCollide
			end
		end)
		local ReparentConnection
		ReparentConnection = v.AncestryChanged:Connect(function()
			if not v:IsDescendantOf(game.Workspace) then
				if v.CanCollide then
					table.remove(NoCollisions,table.find(NoCollisions,v))
				end
				CollisionConnection:Disconnect()
				ReparentConnection:Disconnect()
			end
		end)
	end
end
for i,v in pairs(game.Workspace:GetDescendants()) do
	Funcs.CheckCollision(v)
end
local AdditionConnection = game.Workspace.DescendantAdded:Connect(function(v)
	Funcs.CheckCollision(v)
end)
function Funcs.MoveCharacter(X,Z)
	Movement.PotentialCFrame = Movement.PotentialCFrame*CFrame.new(X,0,Z)
end
function Funcs.Effect(Material,Color,Size,CFrame,Time)
	local EffectPart = Instance.new("Part")
	EffectPart.Anchored = true
	EffectPart.CanCollide = false
	EffectPart.Size = Size
	EffectPart.Material = Material
	EffectPart.Color = Color
	EffectPart.CFrame = CFrame
	EffectPart.Archivable = false
	EffectPart.Name = Funcs.RandomString()
	EffectPart.Parent = game.Workspace
	Funcs.Debris(EffectPart,Time)
	return EffectPart
end
function Funcs.Refit(Instance,Parent)
	if Instance.Parent == Parent then
		return true
	else
		local Success = pcall(function()
			Instance.Name = Funcs.RandomString()
			Instance.Parent = Parent
		end)
		return Success
	end 
end
function Funcs.KillPart(Instance, a)
    spawn(function()
        FLING = Instance
        wait(1)
        if FLING == Instance then
            FLING = nil
        end
    end)
end
function Funcs.Attack(Position,Range)
	local Range = math.clamp(Range*S,0,2147483647)
	pcall(function()
		for _,v in pairs(game.Workspace:GetDescendants()) do
			if v and v.Parent and v:IsA("BasePart") and v.Parent:FindFirstChildOfClass("Humanoid") and ((Position - v.Position).Magnitude < Range) and (not v:IsDescendantOf(CHARACTER)) then
				local SoundPart = Instance.new("Part")
				SoundPart.CFrame = v.CFrame
				local DeathSound = Instance.new("Sound")
				DeathSound.SoundId = "rbxassetid://10209303"
				DeathSound.Volume = 10
				DeathSound.PlayOnRemove = true
				DeathSound.Playing = true
				DeathSound.Parent = SoundPart
				SoundPart.Parent = Funcs.Serv(Services[math.random(1,#Services)])
				SoundPart:Destroy()
				Funcs.KillPart(v,true)
			end
		end
	end)
end
Funcs.Serv("RunService"):BindToRenderStep(Username.."'s Ultraskidded Lord",199,function()
    if not (CHARACTER and CHARACTER.Parent) then
        pcall(function()
            Camera.Music.Playing = false
        end)
        return
    end
	Timing.Throttle,Timing.Sine = (tick()-Timing.LastFrame)/(1/60),Timing.Sine+(tick()-Timing.LastFrame)*60
	Timing.LastFrame = tick()
	if not Camera.Part or not Camera.Part:IsDescendantOf(game) or Camera.Part.Archivable then
		Funcs.Debris(Camera.Part,0)
		Camera.Part = Instance.new(BasePartClassTypes[math.random(1,#BasePartClassTypes)])
		Camera.Part.Name = Funcs.RandomString()
		Camera.Part.Archivable = false
		Camera.Part.Parent = Funcs.Serv(Services[math.random(1,#Services)])
	end
	RayProperties.FilterDescendantsInstances = NoCollisions
	if LocalPlayer.Name == Username then
		Camera.CFrame = game.Workspace.CurrentCamera.CFrame
		local LookVector = Camera.CFrame.LookVector
		if not Movement.Flying then
			local Ray_ = game.Workspace:Raycast(Movement.CFrame.Position-Vector3.new(0,S-Movement.HipHeight,0),Vector3.new(0,-9e9,0),RayProperties)
			if Ray_ then
				Movement.Falling = false
				local NewCFrame = CFrame.new(0,(Ray_.Position.Y-Movement.CFrame.Y)+3*S,0)*Movement.CFrame
				Movement.CFrame = Funcs.Clerp(Movement.CFrame,NewCFrame,.1)
				if (Movement.CFrame.Position-NewCFrame.Position).Magnitude > 1*S then
					Movement.Falling = true
				end
				local SwordRay = game.Workspace:Raycast(Movement.CFrame*CFrame.new(1.5*S,-2.5*S,5.75*S).Position,Vector3.new(0,-1*S,0),RayProperties)
				if not SwordRay then
					Movement.Falling = true
				end
			else
				Movement.Falling = true
				if Movement.CFrame.Y-1 < game.Workspace.FallenPartsDestroyHeight then
					local Base = nil
					for i,v in pairs(game.Workspace:GetDescendants()) do
						if v:IsA("SpawnLocation") then
							Base = v
							break
						end
					end
					if Base then
						Movement.CFrame = CFrame.new(Base.Position)*CFrame.new(0,(Base.Size.Y/2)+3*S,0)
					else
						Movement.CFrame = CFrame.new(0,100,0)
					end
				else
					Movement.CFrame = CFrame.new(0,-3*Timing.Throttle-math.clamp(Movement.CFrame.Y/100,0,1e4),0)*Movement.CFrame
				end
			end
			local OldCFrame = Movement.CFrame
			Movement.PotentialCFrame = CFrame.new(Movement.CFrame.Position,Vector3.new(Movement.CFrame.X+LookVector.X,Movement.CFrame.Y,Movement.CFrame.Z+LookVector.Z))
			if Keys.W then
				Funcs.MoveCharacter(0,-1)
			end
			if Keys.A then
				Funcs.MoveCharacter(-1,0)
			end
			if Keys.S then
				Funcs.MoveCharacter(0,1)
			end
			if Keys.D then
				Funcs.MoveCharacter(1,0)
			end
			if (Movement.PotentialCFrame.X ~= OldCFrame.X or Movement.PotentialCFrame.Z ~= OldCFrame.Z) and Movement.WalkSpeed > 0 then
				Movement.Walking = true
				Movement.CFrame = CFrame.new(Movement.CFrame.Position,Movement.PotentialCFrame.Position)*CFrame.new(0,0,-((Movement.WalkSpeed/60)*Timing.Throttle))
				Movement.CFrame = CFrame.new(Movement.CFrame.Position)*(OldCFrame-OldCFrame.Position)
				Movement.CFrame = Funcs.Clerp(Movement.CFrame,CFrame.new(Movement.CFrame.Position,Vector3.new(OldCFrame.X,Movement.CFrame.Y,OldCFrame.Z))*CFrame.Angles(0,math.rad(180),0),.15)
			else
				Movement.Walking = false
			end
		else
			local OldCFrame = Movement.CFrame
			Movement.PotentialCFrame = CFrame.new(Movement.CFrame.Position,Movement.CFrame.Position+LookVector)
			if Keys.W then
				Funcs.MoveCharacter(0,-1)
			end
			if Keys.A then
				Funcs.MoveCharacter(-1,0)
			end
			if Keys.S then
				Funcs.MoveCharacter(0,1)
			end
			if Keys.D then
				Funcs.MoveCharacter(1,0)
			end
			if (Movement.PotentialCFrame.X ~= OldCFrame.X or Movement.PotentialCFrame.Z ~= OldCFrame.Z) and Movement.WalkSpeed > 0 then
				Movement.Walking = true
				Movement.CFrame = CFrame.new(Movement.CFrame.Position,Movement.PotentialCFrame.Position)*CFrame.new(0,0,-((Movement.WalkSpeed/60)*Timing.Throttle))
				Movement.CFrame = CFrame.new(Movement.CFrame.Position,Movement.CFrame.Position+LookVector)
			else
				Movement.Walking = false
			end
		end
		Character.HumanoidRootPart.CFrame = Movement.CFrame*CFrame.new(0,Movement.HipHeight,0)
		Funcs.UpdateWeld(Camera.Weld,Camera.Part,Character.HumanoidRootPart)
		--game.Workspace.CurrentCamera.CameraSubject = Camera.Part
		--game.Workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
		pcall(function()
			if met == 1 then
				game.Workspace.CurrentCamera.FieldOfView = 70 + Camera.Music.PlaybackLoudness / 95
			else
				game.Workspace.CurrentCamera.FieldOfView = 70 - Camera.Music.PlaybackLoudness / 95
			end
		end)
		LocalPlayer.CameraMaxZoomDistance = 100000
		LocalPlayer.CameraMinZoomDistance = 0
		LocalPlayer.CameraMode = Enum.CameraMode.Classic
		game.Workspace.CurrentCamera.FieldOfViewMode = Enum.FieldOfViewMode.Vertical
		if Funcs.Serv("UserInputService").MouseBehavior == Enum.MouseBehavior.LockCenter then
			if not Movement.Flying then
				Movement.CFrame = CFrame.new(Movement.CFrame.Position,Vector3.new(Movement.CFrame.X+LookVector.X,Movement.CFrame.Y,Movement.CFrame.Z+LookVector.Z))
			else
				Movement.CFrame = CFrame.new(Movement.CFrame.Position,Movement.CFrame.Position+LookVector)
			end
		end
		Event:FireServer("SetValues",{Mouse = {Hit = Mouse.Hit,Target = Mouse.Target},Camera = {CFrame = Camera.CFrame},Movement = {CFrame = Movement.CFrame,Walking = Movement.Walking,Falling = Movement.Falling,Flying = Movement.Flying}})
	else
		Funcs.Serv("TweenService"):Create(Movement.Value,TweenInfo.new(1/20,Enum.EasingStyle.Linear),{Value = Movement.PotentialCFrame}):Play()
		Movement.CFrame = Movement.Value.Value
	end
end)
Character.Head = CHARACTER["Head"]
Character.HumanoidRootPart = Instance.new("Part", CHARACTER)
Character.HumanoidRootPart.CanCollide = false
Character.HumanoidRootPart.Transparency = 1
CHARACTER.HumanoidRootPart.Name = "HumanoidRootPart1"
Character.HumanoidRootPart.Name = "HumanoidRootPart"
Character.Torso = CHARACTER["Torso"]
Character.LeftArm = CHARACTER["Left Arm"]
Character.RightArm = CHARACTER["Right Arm"]
Character.LeftLeg = CHARACTER["Left Leg"]
Character.RightLeg = CHARACTER["Right Leg"]
pcall(function()
    Character.LeftWing = CHARACTER.SoloWing.Handle
    Character.LeftWing.Anchored = true
    local a = Character.LeftWing.att1_Handle
    a.Position = Vector3.new(0.2, 1.5, -0.5)
    a.Orientation = Vector3.new(0, 125, 0)
end)
pcall(function()
    Character.RightWing = CHARACTER.SoloWing2.Handle
    Character.RightWing.Anchored = true
    local a = Character.RightWing.att1_Handle
    a.Position = Vector3.new(0.2, 1.5, 0.5)
    a.Orientation = Vector3.new(0, 55, 0)
end)
pcall(function()
    Character.Sword = CHARACTER.MeshPartAccessory.Handle
    Character.Sword.Anchored = true
    local a = Character.Sword.att1_Handle
    a.Position = Vector3.new(0, -0.5, -0.5)
    a.Orientation = Vector3.new(0, -90, -135)
end)

local FLINGH = nil
pcall(function()
    FLINGH = CHARACTER.Model.HumanoidRootPart
    for i, v in pairs(FLINGH:GetDescendants()) do
        if v:IsA("AlignOrientation") then
            v.Enabled = false
        end
    end
    FLINGH.Transparency = 0.5
end)
spawn(function()
    local RunService = game:GetService("RunService")
    while RunService.Heartbeat:Wait() and FLINGH and FLINGH.Parent do
        FLINGH.RotVelocity = Vector3.new(1000, 1000, 1000)
    end
end)

local SoundServiceProps = {AmbientReverb = Enum.ReverbType.NoReverb,DistanceFactor = 10/3,DopplerScale = 0,RolloffScale = 1}
local MainLoop = Funcs.Serv("RunService").RenderStepped:Connect(function()
    if not (CHARACTER and CHARACTER.Parent) then
        return
    end
	for i,v in pairs(SoundServiceProps) do
		Funcs.Serv("SoundService")[i] = v
	end
	local Transparent = false
	--[[if LocalPlayer.Name == Username then
		local CameraRay = game.Workspace:Raycast(game.Workspace.CurrentCamera.Focus.Position,-game.Workspace.CurrentCamera.CFrame.LookVector*(game.Workspace.CurrentCamera.CFrame.Position-game.Workspace.CurrentCamera.Focus.Position).Magnitude,RayProperties)
		if CameraRay then
			game.Workspace.CurrentCamera.CFrame = game.Workspace.CurrentCamera.CFrame*CFrame.new(0,0,-(game.Workspace.CurrentCamera.CFrame.Position-game.Workspace.CurrentCamera.Focus.Position).Magnitude)*CFrame.new(0,0,(game.Workspace.CurrentCamera.Focus.Position-CameraRay.Position).Magnitude*.99)
		end
		if (game.Workspace.CurrentCamera.CFrame.Position-game.Workspace.CurrentCamera.Focus.Position).Magnitude < .6 and Funcs.Serv("UserInputService").MouseBehavior == Enum.MouseBehavior.LockCenter then
		end
	end
	if not Character.Eyes or not Funcs.Refit(Character.Eyes,workspace) then
		if table.find(NoCollisions,Character.Eyes) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.Eyes))
		end
		Funcs.Debris(Character.Eyes,0)
		Character.Eyes = Backups.Eyes:Clone()
		Character.Eyes.Name = Funcs.RandomString()
		Character.Eyes.Archivable = false
		Character.Eyes.Parent = workspace
	end
	if not table.find(NoCollisions,Character.Eyes) then
		table.insert(NoCollisions,Character.Eyes)
	end
	if not Character.Gun or not Funcs.Refit(Character.Gun,workspace) then
		if table.find(NoCollisions,Character.Gun) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.Gun))
		end
		Funcs.Debris(Character.Gun,0)
		Character.Gun = Backups.Gun:Clone()
		Character.Gun.Name = Funcs.RandomString()
		Character.Gun.Archivable = false
		Character.Gun.Parent = workspace
	end
	if not table.find(NoCollisions,Character.Gun) then
		table.insert(NoCollisions,Character.Gun)
	end
	if not Character.Sword or not Funcs.Refit(Character.Sword,workspace) then
		if table.find(NoCollisions,Character.Sword) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.Sword))
		end
		Funcs.Debris(Character.Sword,0)
		Character.Sword = Backups.Sword:Clone()
		Character.Sword.Name = Funcs.RandomString()
		Character.Sword.Archivable = false
		Character.Sword.Parent = workspace
	end
	if not table.find(NoCollisions,Character.Sword) then
		table.insert(NoCollisions,Character.Sword)
	end
	if not Character.Horns or not Funcs.Refit(Character.Horns,workspace) then
		if table.find(NoCollisions,Character.Horns) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.Horns))
		end
		Funcs.Debris(Character.Horns,0)
		Character.Horns = Backups.Horns:Clone()
		Character.Horns.Name = Funcs.RandomString()
		Character.Horns.Archivable = false
		Character.Horns.Parent = workspace
	end
	if not table.find(NoCollisions,Character.Horns) then
		table.insert(NoCollisions,Character.Horns)
	end
	if not Character.RightWing or not Funcs.Refit(Character.RightWing,workspace) then
		if table.find(NoCollisions,Character.RightWing) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.RightWing))
		end
		Funcs.Debris(Character.RightWing,0)
		Character.RightWing = Backups.RightWing:Clone()
		Character.RightWing.Name = Funcs.RandomString()
		Character.RightWing.Archivable = false
		Character.RightWing.Parent = workspace
	end
	if not table.find(NoCollisions,Character.RightWing) then
		table.insert(NoCollisions,Character.RightWing)
	end
	if not Character.LeftWing or not Funcs.Refit(Character.LeftWing,workspace) then
		if table.find(NoCollisions,Character.LeftWing) then
			table.remove(NoCollisions,table.find(NoCollisions,Character.LeftWing))
		end
		Funcs.Debris(Character.LeftWing,0)
		Character.LeftWing = Backups.LeftWing:Clone()
		Character.LeftWing.Name = Funcs.RandomString()
		Character.LeftWing.Archivable = false
		Character.LeftWing.Parent = workspace
	end
	if not table.find(NoCollisions,Character.LeftWing) then
		table.insert(NoCollisions,Character.LeftWing)
	end]]
	if not Camera.Part or not Camera.Part:IsDescendantOf(game) or Camera.Part.Archivable or tick()-Timing.LastPlaying >= 1 then
		Funcs.Debris(Camera.Part,0)
		Camera.Part = Instance.new(BasePartClassTypes[math.random(1,#BasePartClassTypes)])
		Camera.Part.Name = Funcs.RandomString()
		Camera.Part.Archivable = false
		Camera.Part.Parent = Funcs.Serv(Services[math.random(1,#Services)])
	end
	pcall(function()
		if not Camera.Music or not Funcs.Refit(Camera.Music,Camera.Part) or Camera.Music.Archivable or not Camera.Music.Looped or not Camera.Music.Playing or Camera.Music.SoundGroup or Camera.Music.SoundId ~= "rbxassetid://"..tostring(AudioId) or Camera.Music.Volume ~= 10 or Camera.Music.RollOffMinDistance ~= 9999 or Camera.Music.RollOffMaxDistance ~= 9999 or Camera.Music.RollOffMode ~= Enum.RollOffMode.Linear or Camera.Music.TimePosition > Timing.SongPosition+1 or Camera.Music.TimePosition < Timing.SongPosition-1 or #Camera.Music:GetChildren() > 0 or tick()-Timing.LastPlaying >= 1 then
			Funcs.Debris(Camera.Music,0)
			Camera.Music = Instance.new("Sound")
			Camera.Music.Name = Funcs.RandomString()
			Camera.Music.Volume = 10
			Camera.Music.PlaybackSpeed = Pitch
			Camera.Music.Looped = true
			Camera.Music.Archivable = false
			Camera.Music.RollOffMinDistance = 9999
			Camera.Music.RollOffMaxDistance = 9999
			Camera.Music.RollOffMode = Enum.RollOffMode.Linear
			Camera.Music.SoundId = "rbxassetid://"..tostring(AudioId)
			Camera.Music.Playing = true
			Camera.Music.TimePosition = Timing.SongPosition
			Camera.Music.Parent = Camera.Part
			Timing.LastPlaying = tick()
		end
	end)
	if Camera.Music.PlaybackLoudness > 0 then
		Timing.LastPlaying = tick()
	end
	if not Character.Silencer or not Character.Silencer:IsDescendantOf(game) or Character.Silencer.Archivable or Character.Silencer.Volume > 0 then
		Funcs.Debris(Character.Silencer,0)
		Character.Silencer = Instance.new("SoundGroup")
		Character.Silencer.Name = Funcs.RandomString()
		Character.Silencer.Archivable = false
		Character.Silencer.Volume = 0
		Character.Silencer.Parent = Funcs.Serv(Services[math.random(1,#Services)])
	end
	if not Movement.Attacking then
		if Movement.Walking then
			Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-.5*math.sin(Timing.Sine/25)*S)*CFrame.Angles(math.rad(20),0,0),.25)
			if Movement.NeckSnap then
				Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
				Movement.NeckSnap = false
			else
				Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
			end
			pcall(function()
				if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
					Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
					Movement.NeckSnap = true
				end
			end)
			if not Movement.Flying and not Movement.Falling then
				Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(-10),0,0)*Welds.Defaults.RightShoulder.C0,.25)
				Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-1*S,0)*CFrame.Angles(math.rad(154.35-5.65*math.sin(Timing.Sine/25)),0,0),.25)
				Welds.Gun.C0 = Funcs.Clerp(Welds.Gun.C0,CFrame.new(0,-2*S,0)*CFrame.Angles(80,0,0),.25)
			else
				Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(80+5*math.cos(Timing.Sine/25)),0,math.rad(45))*Welds.Defaults.RightShoulder.C0,.25)
				Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-0,-.5*S)*CFrame.Angles(0,math.rad(170),math.rad(-10)),.25)
				Welds.Gun.C0 = Funcs.Clerp(Welds.Gun.C0,CFrame.new(0,-2*S,0)*CFrame.Angles(80,0,0),.25)
			end
			Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(20),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
			Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
			Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
		else
			Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-.5*math.sin(Timing.Sine/25)*S)*CFrame.Angles(math.rad(20),0,0),.25)
			if Movement.NeckSnap then
				Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(20),math.rad(10*math.sin(Timing.Sine/50)),0),1)
				Movement.NeckSnap = false
			else
				Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(20),math.rad(10*math.sin(Timing.Sine/50)),0),.25)
			end
			pcall(function()
				if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
					Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(20+math.random(-20,20)),math.rad((10*math.sin(Timing.Sine/50))+math.random(-20,20)),math.rad(math.random(-20,20))),1)
					Movement.NeckSnap = true
				end
			end)
			if not Movement.Flying and not Movement.Falling then
				Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(-10),0,0)*Welds.Defaults.RightShoulder.C0,.25)
				Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-1*S,0)*CFrame.Angles(math.rad(154.35-5.65*math.sin(Timing.Sine/25)),0,0),.25)
				Welds.Gun.C0 = Funcs.Clerp(Welds.Gun.C0,CFrame.new(0,-2*S,0)*CFrame.Angles(80,0,0),.25)
			else
				Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(80+5*math.cos(Timing.Sine/25)),0,math.rad(45))*Welds.Defaults.RightShoulder.C0,.25)
				Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,0,-.5*S)*CFrame.Angles(0,math.rad(170),math.rad(-10)),.25)
				Welds.Gun.C0 = Funcs.Clerp(Welds.Gun.C0,CFrame.new(0,-2*S,0)*CFrame.Angles(80,0,0),.25)
			end
			Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(20),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
			Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(10),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
			Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(20),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
		end
	end
	Welds.RightWing.C0 = Funcs.Clerp(Welds.RightWing.C0,CFrame.new(.15*S,.5*S,.5*S)*CFrame.Angles(0,math.rad(105-25*math.cos(Timing.Sine/25)),0),.25)
	Welds.LeftWing.C0 = Funcs.Clerp(Welds.LeftWing.C0,CFrame.new(-.15*S,.5*S,.5*S)*CFrame.Angles(0,math.rad(75+25*math.cos(Timing.Sine/25)),0),.25)
	if not (FLING and FLING.Parent) then
	    CHARACTER.HumanoidRootPart1.CFrame = Movement.CFrame*CFrame.new(0,Movement.HipHeight,0)
	else
	    CHARACTER.HumanoidRootPart1.CFrame = FLING.CFrame
	end
	Funcs.UpdateWeld(Camera.Weld,Camera.Part,Character.HumanoidRootPart)
	Funcs.UpdateWeld(Welds.RootJoint,Character.Torso,Character.HumanoidRootPart)
	Funcs.UpdateWeld(Welds.Neck,Character.Head,Character.Torso)
	Funcs.UpdateWeld(Welds.RightShoulder,Character.RightArm,Character.Torso)
	Funcs.UpdateWeld(Welds.LeftShoulder,Character.LeftArm,Character.Torso)
	Funcs.UpdateWeld(Welds.RightHip,Character.RightLeg,Character.Torso)
	Funcs.UpdateWeld(Welds.LeftHip,Character.LeftLeg,Character.Torso)
	Funcs.UpdateWeld(Welds.Eyes,Character.Eyes,Character.Head)
	Funcs.UpdateWeld(Welds.Sword,Character.Sword,Character.RightArm)
	Funcs.UpdateWeld(Welds.Gun,Character.Gun,Character.RightArm)
	Funcs.UpdateWeld(Welds.Horns,Character.Horns,Character.Head)
	Funcs.UpdateWeld(Welds.RightWing,Character.RightWing,Character.Torso)
	Funcs.UpdateWeld(Welds.LeftWing,Character.LeftWing,Character.Torso)
	if LoopColor >= 1 then
		LoopColor = 0
	else
		LoopColor = LoopColor + .006
	end
end)

function Funcs.ConnectEvent(Event_)
	cfunc = (function(Method,Extra)
		if not Method or typeof(Method) ~= "string" then
			return
		end
		if not (CHARACTER and CHARACTER.Parent) then return end
		if Method == "SetValues" and LocalPlayer.Name ~= Username then
			Mouse.Hit,Mouse.Target,Camera.CFrame,Movement.PotentialCFrame,Movement.Walking,Movement.Falling,Movement.Flying = Extra.Mouse.Hit,Extra.Mouse.Target,Extra.Camera.CFrame,Extra.Movement.CFrame,Extra.Movement.Walking,Extra.Movement.Falling,Extra.Movement.Flying
		elseif Method == "SetTiming" then
			Timing.Sine,AudioId,Pitch = Extra.Timing.Sine,Extra.AudioId,Extra.SongPitch
		elseif Method == "AttackPosition" then
			Funcs.Attack(Extra.Position,Extra.Range)
		elseif Method == "StopScript" then
			if Extra.StopKey == Funcs.WaitForChildOfClass(Funcs.Serv("ReplicatedStorage"), "FileMesh").Name then
				stopit = true
				Funcs.Serv("RunService"):UnbindFromRenderStep(Username.."'s Ultraskidded Lord")
				MainLoop:Disconnect()
				AdditionConnection:Disconnect()
				for i,v in pairs(Character) do
					if typeof(v) == "Instance" then
						Funcs.Debris(v,0)
					end
				end
				Funcs.Debris(Camera.Part,0)
				Funcs.Debris(Camera.Music,0)
				if LocalPlayer.Name == Username and not LocalPlayer.Parent == Funcs.Serv("Players") then
					Funcs.Serv("TeleportService"):TeleportToPlaceInstance(game.PlaceId,game.JobId)
				end
			end
		elseif Method == "Chat" then
			Funcs.Notify("[Ultraskidded Lord]: ",Extra.Message)
		elseif Method == "Key" then
			if Keys[Extra.Key] ~= nil and LocalPlayer.Name ~= Username then
				Keys[Extra.Key] = Extra.Down
			elseif Extra.Down then
				if Extra.Key == "Q" then
					Funcs.Debris(Camera.Part,0)
					Funcs.Debris(Camera.Music,0)
					for i,v in pairs(Character) do
						if v:IsA("BasePart") then
							Funcs.Debris(v,0)
						end
					end
				elseif Extra.Key == "M" and not Movement.Attacking then
					if Camera.Music.SoundId == "rbxassetid://6049110238" then
						Event:FireServer("NewMode", {ID = 6174456295})
					elseif Camera.Music.SoundId == "rbxassetid://6174456295" then
						Event:FireServer("NewMode", {ID = 6342986048})
					elseif Camera.Music.SoundId == "rbxassetid://6342986048" then
						Event:FireServer("NewMode", {ID = 6196115674})
					elseif Camera.Music.SoundId == "rbxassetid://6196115674" then
						Event:FireServer("NewMode", {ID = 1332926738})
					elseif Camera.Music.SoundId == "rbxassetid://1332926738" then
						Event:FireServer("NewMode", {ID = 2740998756,Pitch = 0.95})
					elseif Camera.Music.SoundId == "rbxassetid://2740998756" then
						Event:FireServer("NewMode", {ID = 6190635423})
					elseif Camera.Music.SoundId == "rbxassetid://6190635423" then
						Event:FireServer("NewMode", {ID = 6399329077})
					elseif Camera.Music.SoundId == "rbxassetid://6279430046" or Camera.Music.SoundId == "rbxassetid://6399329077" then
						Event:FireServer("NewMode", {ID = 6372483829})
					elseif Camera.Music.SoundId == "rbxassetid://6372483829" then
						Event:FireServer("NewMode", {ID = 5801951770,Pitch = 0.95})
					elseif Camera.Music.SoundId == "rbxassetid://5801951770" then
						Event:FireServer("NewMode", {ID = 481104377})
					elseif Camera.Music.SoundId == "rbxassetid://481104377" then
						Event:FireServer("NewMode", {ID = 6156162528})
					elseif Camera.Music.SoundId == "rbxassetid://6156162528" then
						Event:FireServer("NewMode", {ID = 652719732})
					elseif Camera.Music.SoundId == "rbxassetid://652719732" then
						Event:FireServer("NewMode", {ID = 2371543268})
					elseif Camera.Music.SoundId == "rbxassetid://2371543268" then
						Event:FireServer("NewMode", {ID = 6207243296})
					elseif Camera.Music.SoundId == "rbxassetid://6207243296" then
						Event:FireServer("NewMode", {ID = 5644788747})
					else
						Event:FireServer("NewMode", {ID = 6049110238})
					end
				elseif Extra.Key == "Z" and not Movement.Attacking then
					Movement.Attacking = true
					Movement.WalkSpeed = 0
					local Start = Timing.Sine/60
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-.5*math.sin(Timing.Sine/25)*S)*CFrame.Angles(math.rad(5),0,math.rad(-20)),.25)
						if Movement.NeckSnap then
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
							Movement.NeckSnap = false
						else
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
						end
						pcall(function()
							if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
								Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
								Movement.NeckSnap = true
							end
						end)
						Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,0.5*S,0)*CFrame.Angles(math.rad(80),0,math.rad(50))*Welds.Defaults.RightShoulder.C0,.25)
						Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(-.5*S,-.5*S,0)*CFrame.Angles(math.rad(180),math.rad(-90),0),.25)
						Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(5),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
						Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
						Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
					until Timing.Sine/60-Start >= .25
					Start = Timing.Sine/60
					coroutine.resume(coroutine.create(function()
						repeat
							Funcs.Serv("RunService").RenderStepped:Wait()
						until Timing.Sine/60-Start >= 1/8
						if LocalPlayer.Name == Username then
							local Hitbox = Instance.new("Part")
							Hitbox.Shape = Enum.PartType.Ball
							Hitbox.Name = Funcs.RandomString()
							Hitbox.CastShadow = false
							Hitbox.Anchored = true
							Hitbox.CanCollide = false
							Hitbox.Material = Enum.Material.ForceField
							Hitbox.Size = Vector3.new(9,9,9)*S
							Hitbox.CFrame = Movement.CFrame*CFrame.new(0,0,-4.5*S)
							Hitbox.Parent = workspace
							Funcs.Serv("TweenService"):Create(Hitbox,TweenInfo.new(1,Enum.EasingStyle.Linear),{LocalTransparencyModifier = 1}):Play()
							Funcs.Debris(Hitbox,1)
						end
						Funcs.Attack(Movement.CFrame*CFrame.new(0,0,-4.5*S).Position,9)
					end))
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-.5*math.sin(Timing.Sine/25)*S)*CFrame.Angles(math.rad(5),0,math.rad(20)),.25)
						if Movement.NeckSnap then
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
							Movement.NeckSnap = false
						else
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
						end
						pcall(function()
							if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
								Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
								Movement.NeckSnap = true
							end
						end)
						Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1*S,0.5*S,-.5*S)*CFrame.Angles(math.rad(80),0,math.rad(-50))*Welds.Defaults.RightShoulder.C0,.25)
						Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(-.5*S,-.5*S,0)*CFrame.Angles(math.rad(180),math.rad(-90),0),.25)
						Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(5),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
						Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
						Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
					until Timing.Sine/60-Start >= .25
					Movement.WalkSpeed= 16*S
					Movement.Attacking = false
				elseif Extra.Key == "X" and not Movement.Attacking then
					Movement.Attacking = true
					Movement.WalkSpeed = 0
					local Start,MousePos = Timing.Sine/60,Mouse.Hit.Position
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-.5*math.sin(Timing.Sine/25)*S)*CFrame.Angles(math.rad(20),0,0),.25)
						if Movement.NeckSnap then
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
							Movement.NeckSnap = false
						else
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
						end
						pcall(function()
							if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
								Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
								Movement.NeckSnap = true
							end
						end)
						Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(80+5*math.cos(Timing.Sine/25)),0,math.rad(45))*Welds.Defaults.RightShoulder.C0,.25)
						Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-0,-.5*S)*CFrame.Angles(0,math.rad(170),math.rad(-10)),.25)
						Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(20),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
						Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
						Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
						--[[local Swirl = Backups.Swirl:Clone()
						Swirl.Name = Funcs.RandomString()
						Swirl.CFrame = Movement.CFrame*CFrame.new(0,-3*S,0)
						Swirl.Parent = workspace
						Funcs.Serv("TweenService"):Create(Swirl,TweenInfo.new(1,Enum.EasingStyle.Linear),{Size = Vector3.new(25,1,25),CFrame = Swirl.CFrame*CFrame.Angles(0,math.rad(180),0),LocalTransparencyModifier = 1}):Play()
						Funcs.Debris(Swirl,1)]]
					until Timing.Sine/60-Start >= .5
					Start = Timing.Sine/60
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,-10*(.5+Timing.Sine/60-Start)*S)*CFrame.Angles(math.rad(20),0,0),.25)
						if Movement.NeckSnap then
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
							Movement.NeckSnap = false
						else
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
						end
						pcall(function()
							if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
								Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
								Movement.NeckSnap = true
							end
						end)
						Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(80+5*math.cos(Timing.Sine/25)),0,math.rad(45))*Welds.Defaults.RightShoulder.C0,.25)
						Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-0,-.5*S)*CFrame.Angles(0,math.rad(170),math.rad(-10)),.25)
						Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(20),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
						Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
						Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
					until Timing.Sine/60-Start >= .5
					Movement.CFrame = CFrame.new(MousePos)*CFrame.new(0,3*S,0)
					Start = Timing.Sine/60
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						--[[local Swirl = Backups.Swirl:Clone()
						Swirl.Name = Funcs.RandomString()
						Swirl.CFrame = CFrame.new(MousePos)
						Swirl.Parent = workspace
						Funcs.Serv("TweenService"):Create(Swirl,TweenInfo.new(1,Enum.EasingStyle.Linear),{Size = Vector3.new(25,1,25),CFrame = Swirl.CFrame*CFrame.Angles(0,math.rad(180),0),LocalTransparencyModifier = 1}):Play()
						Funcs.Debris(Swirl,1)]]
					until Timing.Sine/60-Start >= .5
					Start = Timing.Sine/60
					repeat
						Funcs.Serv("RunService").RenderStepped:Wait()
						Welds.RootJoint.C0 = Funcs.Clerp(Welds.RootJoint.C0,Welds.Defaults.RootJoint.C0*CFrame.new(0,0,(-.5*math.sin(Timing.Sine/25)*S)-(10-(Timing.Sine/60-Start)*20))*CFrame.Angles(math.rad(20),0,0),.25)
						if Movement.NeckSnap then
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,1)
							Movement.NeckSnap = false
						else
							Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0,.25)
						end
						pcall(function()
							if math.random(1,math.floor((15/Timing.Throttle)+.5)) == 1 then
								Welds.Neck.C0 = Funcs.Clerp(Welds.Neck.C0,Welds.Defaults.Neck.C0*CFrame.Angles(math.rad(math.random(-20,20)),math.rad(math.random(-20,20)),math.rad(math.random(-20,20))),1)
								Movement.NeckSnap = true
							end
						end)
						Welds.RightShoulder.C0 = Funcs.Clerp(Welds.RightShoulder.C0,CFrame.new(1.5*S,.5*S,0)*CFrame.Angles(math.rad(80+5*math.cos(Timing.Sine/25)),0,math.rad(45))*Welds.Defaults.RightShoulder.C0,.25)
						Welds.Sword.C0 = Funcs.Clerp(Welds.Sword.C0,CFrame.new(0,-0,-.5*S)*CFrame.Angles(0,math.rad(170),math.rad(-10)),.25)
						Welds.LeftShoulder.C0 = Funcs.Clerp(Welds.LeftShoulder.C0,CFrame.new(-1.5*S,.5*S,0)*CFrame.Angles(math.rad(20),0,math.rad(-10-10*math.cos(Timing.Sine/25)))*Welds.Defaults.LeftShoulder.C0,.25)
						Welds.RightHip.C0 = Funcs.Clerp(Welds.RightHip.C0,CFrame.new(1*S,-1*S,0)*CFrame.Angles(math.rad(-20),math.rad(80),math.rad(10+10*math.cos(Timing.Sine/25))),.25)
						Welds.LeftHip.C0 = Funcs.Clerp(Welds.LeftHip.C0,CFrame.new(-1*S,-1*S,0)*CFrame.Angles(math.rad(-10),math.rad(-80),math.rad(-10-10*math.cos(Timing.Sine/25))),.25)
					until Timing.Sine/60-Start >= .5
					Movement.WalkSpeed = 16*S
					Movement.Attacking = false
				elseif Extra.Key == "C" and not Movement.Attacking then
					Funcs.Attack(Vector3.new(),9e9)
				elseif Extra.Key == "V" and not Movement.Attacking then
					
				end
			end
		end
	end)
end
if Event then
	Funcs.ConnectEvent(Event)
end

-----------

local Funcs1 = {}
local StoppingScript = false
function Funcs1.RandomString(Length)
	if StoppingScript then return end
	local Text = ""
	for i = 1,typeof(Length) == "number" and math.clamp(Length,1,100) or math.random(10,100) do
		Text = Text..string.char(math.random(14,128))
	end
	return Text
end
function Funcs1.Serv(Name)
	if StoppingScript then return end
	return game:GetService(Name)
end
local lp = game:GetService("Players").LocalPlayer
local UserId = lp.UserId
local Username = lp.Name
local SuccessOwnership,Ownership = true, true
function Funcs1.Debris(Instance,Delay)
	if StoppingScript then return end
	Funcs1.Serv("Debris"):AddItem(Instance,Delay)
end
Funcs1.Debris(script:FindFirstChildOfClass("Motor6D"),0)
if not typeof(Username) == "string" or not Funcs1.Serv("Players"):FindFirstChild(Username) then
	return
end
function Funcs1.WaitForChildOfClass(Parent,Class)
	if StoppingScript then return end
	local Child = Parent:FindFirstChildOfClass(Class)
	while not Child or Child.ClassName ~= Class do
		Child = Parent.ChildAdded:Wait()
	end
	return Child
end
local Start,SongStart,AudioId,AudioIds,CurrentSong,Trackers,Pitch,ClientStop = tick(),tick(),6049110238,{{Id = 6049110238}},1,{},1,Funcs1.RandomString(100)
local MainLoop,NoCharacter,Removed
function ChangeMode(ID,fPitch)
	if StoppingScript then return end
	if fPitch == nil then
		fPitch = 1
	end
	if type(fPitch) ~= "number" then
		fPitch = 1
	end
	if type(ID) == "number" then
		AudioId,SongStart,Pitch = ID,tick(),fPitch
	else
		warn("ID is invalid.")
		AudioId,SongStart,Pitch = 6049110238,tick(),fPitch
	end
end
sfunc = function (Method,Extra)
	if StoppingScript then return end
	local v = lp
	if v.Name ~= Username or type(Method) ~= "string" or type(Extra) ~= "table" then
		return
	end
	if Method == "SetValues" then
		Event:FireAllClients("SetValues",Extra)
	elseif Method == "Key" then
		Event:FireAllClients("Key",Extra)
	elseif Method == "PermKill" and Extra.Part and not table.find(Trackers,Extra.Part.Name) then
		table.insert(Trackers,Extra.Part.Name)
	elseif Method == "Chat" and Extra.Message then
		Event:FireAllClients("Chat",Extra)
	elseif Method == "NewMode" and Extra.ID then
		if type(Extra.ID) == "number" then
			ChangeMode(Extra.ID,Extra.Pitch)
		end
	elseif Method == "StopScript" then
		if Extra.LeaveKey == "~!PPl.a/zzz'@#$%^&*()_+{}|||" and Extra.LeaveKeySecond == "YUDFIJGIFGHUFU" and Extra.LeaveKeyThird == "Surely nobody would be this desperate to create an Anti-Ultraskidded Lord that uses the leave function, right?" and Extra.FourthLeaveKey == "AQbstBtRnFO\n@YnL?ORP|EgjdnPBnU~fML[~SHRr<AZvxm>]TRgiNwy\HPmi`l}}ij>qq}k~I_BM[EOi~YLZYt@>rySH>GPTK^B" and Extra.LastLeaveKey == "|||}{+_)(*&^%$#@'zzz/a.lPP!~-Edit" and Extra.bruh == "WaitForChildOfClass" and Extra.r == "'s Immortality Lord" and Extra.f == "ʟᴍᴀᴏᴏᴏ" and Extra.shutup == "table" and Extra.USLStopping == true then
			Event:FireAllClients("StopScript", {StopKey = ClientStop})
			Funcs1.Debris(Event,1)
			StoppingScript = true
		end
	end
end
while wait() do
Event:FireAllClients("SetTiming",{Timing = {Sine = (tick()-Start)*60,SongPosition = (tick()-SongStart)*Pitch},AudioId = AudioId,SongPitch = Pitch}) end
