if Player.CharName ~= "Brand" then return end
require("common.log")
module("Bad Paste Brand", package.seeall, log.setup)
clean.module("Bad Paste Brand", clean.seeall, log.setup)

local clock = os.clock
local insert, sort = table.insert, table.sort
local huge, min, max, abs = math.huge, math.min, math.max, math.abs
local _SDK = _G.CoreEx
local Console, ObjManager, EventManager, Geometry, Input, Renderer, Enums, Game = _SDK.Console, _SDK.ObjectManager, _SDK.EventManager, _SDK.Geometry, _SDK.Input, _SDK.Renderer, _SDK.Enums, _SDK.Game
local Menu, Orbwalker, Collision, Prediction, HealthPred = _G.Libs.NewMenu, _G.Libs.Orbwalker, _G.Libs.CollisionLib, _G.Libs.Prediction, _G.Libs.HealthPred
local DmgLib, ImmobileLib, Spell = _G.Libs.DamageLib, _G.Libs.ImmobileLib, _G.Libs.Spell
local TS = Libs.TargetSelector()

---@type TargetSelector
local TS = _G.Libs.TargetSelector()

local Brand = {}
Brand.Logic = {}

--Skills

local Q =
    Spell.Skillshot(
    {
         Slot = Enums.SpellSlots.Q,
		  Range = 1050,
		  Delay = 0.25,
		  Speed = 1200,
		  Radius = 210,
		  Collisions = { Heroes = true, Minions = true, WindWall = true },
		  Type = "Linear",
		  UseHitbox = true,
		  Key = "Q"
    }
)
local W =
    Spell.Skillshot(
    {
         Slot = Enums.SpellSlots.W,
		 Range = 900,
		 Delay = 0.25,
		 Radius = 240.0,
		 Type = "Circular",
		 Key = "W"
    }
)
local E =
    Spell.Targeted(
    {
        Slot = Enums.SpellSlots.E,
        RawSpell = "E",
        Range = 625,
        Delay = 0.25,
        Key = "E"
    }
)
local R =
    Spell.Targeted(
    {
        Slot = Enums.SpellSlots.R,
        RawSpell = "R",
        Range = 750,
        Radius = 0,
        Speed = 1000,
        Delay = 0.25,
        Key = "R"
    }
)

--Functions
local function GameIsAvailable()
    return not (Game.IsChatOpen() or Game.IsMinimized() or Player.IsDead or Player.IsRecalling)
end

function Brand.GetTargets(Spell)
  return TS:GetTargets(Spell.Range,true)
end

function Brand.GetWDmg()
    return 12 + (0.2 + ((W:GetLevel() * 4) / 100) * Player.TotalAD) * 3
end

function Brand.GetEDmg()
    return 90 + E:GetLevel() * 30 + (1 * Player.TotalAD)
end

function Brand.GetQDmg()
    return 25 + Q:GetLevel() * 50 + (0.7 * Player.TotalAD)
end

function Brand.GetRDmg()
    return 100 + R:GetLevel() * 125 + (0.5 * Player.TotalAD)
end

function Brand.GetMinonsAroundTarget(Target)
    local pos = Target.Position
    local minionTable = ObjManager.GetNearby("enemy", "minions")

    local minionAmount = 0
    for _, minion in ipairs(minionTable) do
        local dist = pos:Distance(minion.Position)
        if dist < 550 then
            minionAmount = minionAmount + 1;
        end
    end
    
    if minionAmount >= Menu.Get("Combo.CastR.Minimum") then
    	return true
    else
    	return false
    end
end

function Brand.Logic.Combo()
	local MenuValueQ = Menu.Get("Combo.CastQ")
	local MenuValueE = Menu.Get("Combo.CastE")
  	local MenuValueW = Menu.Get("Combo.CastW")
  	local MenuValueR = Menu.Get("Combo.CastR")

  	for k, enemy in pairs(Brand.GetTargets(Q)) do
  		if E:IsReady() and MenuValueE then
  			if E:IsInRange(enemy) then 
  				E:Cast(enemy)
  			end
  		end

  		if Q:IsReady() and MenuValueQ then
  			local predQ = Q:GetPrediction(enemy)
  			if predQ ~= nil and predQ.HitChanceEnum >= Enums.HitChance.High and Q:IsInRange(enemy) then
  				if Q:Cast(predQ.CastPosition) then return true end
  			end
  		end

  		if W:IsReady() and MenuValueW then
  			local predW = W:GetPrediction(enemy)
  			if predW ~= nil and predW.HitChanceEnum >= Enums.HitChance.High and W:IsInRange(enemy) then
  				if W:Cast(predW.CastPosition) then return true end
  			end
  		end

  		if R:IsReady() and MenuValueR then
  			if R:IsInRange(enemy) then 
  				if (100*enemy.Health/enemy.MaxHealth) < Menu.Get("Combo.CastR.healthLimit") then 
  					if Brand.GetMinonsAroundTarget(enemy) then 
  						R:Cast(enemy)
  					end
  				end
  			end
  		end
  	end
end

function Brand.Logic.Waveclear()
	--INFO("Wave Clear")

	local MenuValueQ = Menu.Get("Waveclear.UseQ")
	local MenuValueW = Menu.Get("Waveclear.UseW")
	local Cannons = {}
	local otherMinions = {}

	for k, v in pairs(ObjManager.GetNearby("enemy", "minions")) do

		local minion = v.AsMinion
	    local pos = minion:FastPrediction(Game.GetLatency()+ W.Delay)

	    if minion.IsTargetable and (minion.IsSiegeMinion or minion.IsSuperMinion) and W:IsInRange(minion) and Q:IsInRange(minion) then
			table.insert(Cannons, pos)
		end

		if minion.IsTargetable and minion.IsLaneMinion and W:IsInRange(minion) and Q:IsInRange(minion) then
			table.insert(otherMinions, pos)
		end

		-- W
		if W:IsReady() and MenuValueW then
		    local cannonsPos, hitCount1 = W:GetBestCircularCastPos(Cannons, W.Radius)
      		local laneMinionsPos, hitCount2 = W:GetBestCircularCastPos(otherMinions, W.Radius)

      		
      		if cannonsPos ~= nil and laneMinionsPos ~= nil and Menu.Get("Waveclear.MinManaW") <= Player.ManaPercent * 100 then
      			if hitCount1 >= 1 then
		        	if W:Cast(cannonsPos) then return true end
			    end
      		end

      		if laneMinionsPos ~= nil and Menu.Get("Waveclear.MinManaW") <= Player.ManaPercent * 100 then
      			if hitCount2 >= 3 then
			    	if W:Cast(laneMinionsPos) then return true end
			    end
      		end	
      	end
      	-- End W

      	-- Q
      	if Q:IsReady() and MenuValueQ then
			if Q:IsInRange(minion) and minion.IsTargetable then
				local predQ = Prediction.GetPredictedPosition(minion, Q, Player.Position)
				if predQ ~= nil and Q:IsReady() then
			        if Q:Cast(predQ.CastPosition) then return true end
			    end
			end
      	end
      	-- EndQ

    end
end

function Brand.Logic.Harass()
	if Menu.Get("Harass.ManaSlider") >= Player.ManaPercent * 100 then return false end
	local MenuValueQ = Menu.Get("Harass.CastQ")
	local MenuValueW = Menu.Get("Harass.CastW")
	for k, enemy in pairs(Brand.GetTargets(Q)) do
		if Q:IsReady() and Q:IsInRange(enemy) and MenuValueQ then
			local predQ = Prediction.GetPredictedPosition(enemy, Q, Player.Position)
			if predQ ~= nil and Q:IsReady() then
			    if Q:Cast(predQ.CastPosition) then return true end
			end
		end

		if W:IsReady() and MenuValueW and W:IsInRange(enemy) then
  			local predW = W:GetPrediction(enemy)
  			if predW ~= nil and predW.HitChanceEnum >= Enums.HitChance.High and W:IsInRange(enemy) then
  				if W:Cast(predW.CastPosition) then return true end
  			end
  		end

	end
end

function Brand.TotalDmg(Target, countSS)
    local Damage = DmgLib.CalculatePhysicalDamage(Player, Target, Player.TotalAD)
    if Q:IsReady() then
        Damage = Damage + DmgLib.CalculatePhysicalDamage(Player, Target, Brand.GetQDmg())
    end
    if W:IsReady() then
        Damage = Damage + DmgLib.CalculatePhysicalDamage(Player, Target, Brand.GetWDmg())
    end
    if E:IsReady() then
        Damage = Damage + DmgLib.CalculatePhysicalDamage(Player, Target, Brand.GetEDmg())
    end
    if R:IsReady() then
        Damage = Damage + DmgLib.CalculatePhysicalDamage(Player, Target, Brand.GetRDmg())
    end
    return Damage
end


-- MENU
function Brand.LoadMenu()
	Menu.RegisterMenu("BadPasteBrand", "Bad Paste Brand", function()
		Menu.NewTree("Combo", "Combo Options", function()
            Menu.Checkbox("Combo.CastQ",   "Use [Q]", true)
            Menu.Checkbox("Combo.CastW",   "Use [W]", true)
            Menu.Checkbox("Combo.CastE",   "Use [E]", true)
            Menu.Checkbox("Combo.CastR",   "Use [R]", true)
            Menu.Slider("Combo.CastR.Minimum", "Minimum Enemies: R", 15, 0, 100, 1)
            Menu.Slider("Combo.CastR.healthLimit", "% health Limit", 15, 0, 100, 1)
        end)

		Menu.NewTree("Harass", "Harass Options", function()
            Menu.Checkbox("Harass.CastQ",   "Use [Q]", true)
            Menu.Checkbox("Harass.CastW",    "Use [W]", false)
            Menu.Slider("Harass.ManaSlider","",50,0,100)
        end)

		Menu.NewTree("WaveClear", "WaveClear Options", function()
            Menu.NewTree("Lane", "Lane", function()
                Menu.Checkbox("Waveclear.UseQ", "Use Q", true)
            	Menu.Slider("Waveclear.MinManaQ", "% Mana Limit", 15, 0, 100, 1)
            	Menu.Checkbox("Waveclear.UseW", "Use W", true)
            	Menu.Slider("Waveclear.MinManaW", "% Mana Limit", 15, 0, 100, 1)
            	Menu.Slider("Waveclear.TargetsW", "Minimum Minions", 1, 1, 5, 1)
            end)
        end)

        Menu.NewTree("Draw", "Drawing Options", function()
        	Menu.Checkbox("Drawing.Damage", "Draw Possible DMG", false)
            Menu.Checkbox("Drawing.Q.Enabled",   "Draw [Q] Range",true)
            Menu.ColorPicker("Drawing.Q.Color", "Draw [Q] Color", 0x118AB2FF)
            Menu.Checkbox("Drawing.W.Enabled",   "Draw [W] Range",true)
            Menu.ColorPicker("Drawing.W.Color", "Draw [W] Color", 0x118AB2FF)
            Menu.Checkbox("Drawing.E.Enabled",   "Draw [E] Range",false)
            Menu.ColorPicker("Drawing.E.Color", "Draw [E] Color", 0x118AB2FF)
            Menu.Checkbox("Drawing.R.Enabled",   "Draw [R] Range",true)
            Menu.ColorPicker("Drawing.R.Color", "Draw [R] Color", 0x118AB2FF)
        end)
	end)
end

function Brand.OnDrawDamage(target, dmgList)
    if Menu.Get("Drawing.Damage") then
        table.insert(dmgList, Brand.TotalDmg(target, true))
    end
end

function Brand.OnDraw()
	if Player.IsVisible and Player.IsOnScreen and not Player.IsDead then
    local Pos = Player.Position
    local Spells = {Q,W,E,R}
    for k, v in pairs(Spells) do
      if Menu.Get("Drawing."..v.Key..".Enabled", true) then
        if Renderer.DrawCircle3D(Pos, v.Range, 30, 3, Menu.Get("Drawing."..v.Key..".Color")) then return true end
      end
    end
  end
  return false
end

-- LOAD
function Brand.OnUpdate()
  if not GameIsAvailable() then return false end
  local OrbwalkerMode = Orbwalker.GetMode()

  local OrbwalkerLogic = Brand.Logic[OrbwalkerMode]

  if OrbwalkerLogic then
    if OrbwalkerLogic() then return true end
  end
  return false
end

function OnLoad()
    Brand.LoadMenu()
    for eventName, eventId in pairs(Enums.Events) do
        if Brand[eventName] then
            EventManager.RegisterCallback(eventId, Brand[eventName])
        end
    end    
    return true
end
