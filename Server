local Players = game:GetService("Players")
local SS = game:GetService("ServerScriptService")
local Services_SS = SS.Services
local Service_DataStore = require(Services_SS.Service_DataStore)
local RS = game:GetService("ReplicatedStorage")
local PlayerRequest = RS.PlayerRequest
local Modules_Replicated = RS.Replicated
local Module_Characters = Modules_Replicated.Service_Characters
local Service_Characters = require(Module_Characters)
local Check_Character = require(Module_Characters.Check_Character)
local Service_Info = require(Modules_Replicated.Service_Info)
local Server_Info = Service_Info.Server
local Replicated_Info = Service_Info.Replicated
local Players_Data = Server_Info.Players
local DEFAULT_DETECTOR_DATA = Server_Info.DEFAULT_DETECTOR_DATA
local Fall_Limit = Replicated_Info.Fall_Limit

local Game_Instances = workspace.Game
local Places_folder = Game_Instances.Places


local Service_Curses = require(script.Curses)
local Service_Workspace =  require(Modules_Replicated.Service_Workspace)

type Userdata = {
	Events: {},
	Curses: {},
	Tasks: {},
	Place: string,
	Subplace: string,
	CanTeleport: boolean,
	Lobby: boolean,
}

type teleport_request = {
	Userstage:number,
	UserMaxstage:number,
	Lobby:number
}

type curse_conditions = {
	Stage : "Si un jugador puede subir de stage con esta maldicion?",
	CursedStages : "Si un jugador puede interactuar con cursed stages?",
	
}

local Tasks = {Curses = {}}



local Places = {
	[{min = 0,max = 20}] = {
		Name = "Tutorial",
		Curses = {
			Destruction = {

			}
		},
	},
	[{min = 21,max = 37}] = {
		Name = "Lobby",
		Curses = {
			
		}
	}
}



local function Stage_get_info (Stage,Request:"Name"|"Curses")
	for Parameters,Info in pairs(Places) do
		if Stage >= Parameters.min and Stage <= Parameters.max then
			return Info[Request]
		end
	end
end



local Stages_Checker = {
	[21] = {
		reset = true
	}
}


local EnumFunction = {
	Place = {
		UserStage = function(Player)
			return Player:GetAttribute("UserStage"),false
		end,
		UserMaxstage = function(Player)
			return Player:GetAttribute("UserMaxstage"),false
		end,
		Lobby = function(Player)
			return Player:GetAttribute("UserStage"),true
		end,		
	},
}



local function get_spawn (Stage)
	local Place_name = Stage_get_info(Stage,"Name")
	return Places_folder[Place_name].Rendering.Spawns[Stage]
end


local function player_teleport (Player,Stage,Data)
	local Lobby = Data.Lobby
	local Character = Check_Character(Player.Character)
	if Character then 
		local Spawn_pos = get_spawn(Stage).Position
		for _,Part in pairs(Character:GetChildren()) do
			if Part:IsA("BasePart") then
				Part.Anchored = Lobby
				Part.CastShadow = not Lobby
				Part.Transparency = Lobby and 1 or 0
			end
		end
		Character.PrimaryPart.Transparency = 1

		local Place = Players_Data[Player.Name].Place
		local Camera
		if Lobby then
			local Cameras = Places_folder[Place].Cameras:GetChildren()
			if #Cameras > 0 then
				Camera = Cameras[math.random(1,#Cameras)].CFrame
			else
				local X,Y,Z = math.random(-100,100),math.random(50,100),math.random(-100,100)
				Camera = CFrame.new(Spawn_pos + Vector3.new(X,Y,Z).Unit*30)
			end
			Character:PivotTo(Camera)
		else
			if Service_Characters.Teleport_Character(Character,CFrame.new(Spawn_pos)) then
				PlayerRequest:FireClient(Player,"Debug","Player_teleported_succesfully")
			end
		end
		PlayerRequest:FireClient(Player,"Camera",{
			CameraType = Camera and "Lobby" or "Player",
			Camera = Camera
		})
	end
	return true
end

local function player_place_update (Player,Instant_load)
	local PD:Userdata = Players_Data[Player.Name]
	local Stage = Player:GetAttribute("UserStage")
	local Lobby = PD.Lobby
	local Place = Stage_get_info(Stage,"Name")
	PD.Place = Place
	PlayerRequest:FireClient(Player,"Place_Update",{Place = Place,Lobby = Lobby,Instant_load = Instant_load})
end

local function player_stage_update (Player,Stage,Data:{})
	local PD = Players_Data[Player.Name]
	local Lobby = Data.Lobby
	if Lobby then
		PD.Lobby = Lobby
	else
		if Lobby ~= nil then
			PD.Lobby = Lobby
		end
	end
	local stage_info = Stages_Checker[Stage]
	if stage_info then
		if stage_info.reset then
			local Character = Check_Character(Player.Character)
			for _,curse in pairs(PD.Curses) do
				if Service_Curses.unregister(Player,curse) then
					player_place_update(Player,true)
				end
			end
		end
	end
	local Stage_name = Stage_get_info(Stage,"Name")
	local NewStage = Player:GetAttribute("UserMaxstage") < Stage 
	if NewStage then
		Player:SetAttribute("UserMaxstage",Stage)
	end
	Player:SetAttribute("UserStage",Stage)

	Player.leaderstats.Stage.Value = Stage
	PD.Place = Stage_name

	PlayerRequest:FireClient(Player,"Stage_Update",{Stage = Stage,Place = Stage_name,First_time = NewStage and true})
	return true
end


local Enum_request = {
	Curse = {
		register = function(Player,Data) 

		end,
		unregister = function(Character,Data)

		end,
	},

	Stage = {
			
	}
}

local Player_Keys = {
	LeftShift = function(Player,Pressing)
		local Character = Check_Character(Player.Character)
		if Character then
			Players_Data[Player.Name].Running = Pressing
			Character.Humanoid.WalkSpeed = Pressing and 32 or 16
		end	
	end,
}

local Stage_Info = {
	[20] = {
		Answers = {
			No = function()

			end,
			Yes = function(Player)
				if player_stage_update(Player,21,{Client = false}) then
					if player_teleport(Player,21,{}) then
						player_place_update(Player,true)
					end
				end
			end,
		}
	}
}


local Interact = {
	Touch = {
		KillParts = function(Character,Part)
			if Check_Character(Character) then
				local Player = Players:GetPlayerFromCharacter(Character)
				if Player then
					if Service_Curses.register(Player,Part.Name) then
						player_place_update(Player)
					end
				end
			end
		end,
		Spawns = function(Character,Part)
			local Player = Players:GetPlayerFromCharacter(Character)
			if Player then 
				local Stage = tonumber(Part.Name)
				local UserMaxstage_update,UserStage_update = false,false
				local P_Data = Players_Data[Player.Name]
				local UserStage = Player:GetAttribute("UserStage")
				local UserMaxStage = Player:GetAttribute("UserMaxstage")
				local Stage_info = Stages_Checker[Stage]
				if Stage ~= UserStage then
					if Stage <= UserMaxStage then
						UserStage = true
					else
						if UserMaxStage + 5 > Stage then
							local reachable,cursedstages = Service_Curses.stage_check(Player,Stage)
							if not reachable then
								if Stage_info then
									if not cursedstages then
										return 
									end
								else
									return
								end
							end
							UserMaxStage = true
							UserStage = true
						end
					end	
					if UserStage or UserMaxStage then
						player_stage_update(Player,Stage,{})
					end
				end
			end
		end,
	}
}

return {
	Client = setmetatable({
		Player_Teleport = function(Player:Player,Where:string):"Success"
			local Stage,Lobby = EnumFunction.Place[Where](Player)
			if player_stage_update(Player,Stage,{Client = true,Lobby = Lobby}) then
				if player_teleport(Player,Stage,{Lobby = Lobby,}) then
					player_place_update(Player)	
				end
			end
		end,
		Player_Input = function(Player,Input,Data)
			local Input_Function = Player_Keys[Input]
			if Input_Function then 
				Input_Function(Player,Data.Pressing)
			end
		end,
		Answer = function(Player,Data)
			local Answer = Data.Answer
			local Stage = Player:GetAttribute("UserStage")
			Stage_Info[Stage].Answers[Answer](Player)
		end,
	},
	{
		__index = function(table,index)
			return function()
				print("no existe la llamada :",index)
			end
		end,
	}),

	player_register = function (Player)
		local First_time = true
		local Player_Events,Player_Data:Userdata,Player_Tasks = {},nil,nil
		Player_Events.CharacterAdded = Player.CharacterAdded:Connect(function(Character)
			Service_Workspace.character.register(Character,"Characters")
			local first_run = First_time and true
			if First_time then
				First_time = false
				local P_stage = Service_DataStore.new(Player)
				Player_Data = Players_Data[Player.Name]
				Player_Data.Events = Player_Events
				if not P_stage then
					Player_Data.Server_info.Kick_reason = "denied"
					return false
				end
				Player_Tasks = Player_Data.Tasks
			end
			local UserStage = Player:GetAttribute("UserStage")
			local Lobby = Player_Data.Lobby
			if player_stage_update(Player,UserStage,{Lobby = Lobby}) then
				if player_teleport(Player,UserStage,{Lobby = Player_Data.Lobby}) then
					player_place_update(Player,Lobby)
				end
			end
			
			local H:Humanoid = Character:WaitForChild("Humanoid")
			Player_Events.Touched = Character.PrimaryPart.Touched:Connect(function(Part)
				local Interact_function = Interact.Touch[Part.Parent.Name]
				if Interact_function then
					local EnumClass,Index,Info = Interact_function(Character,Part)
					if EnumClass then
						Enum_request[EnumClass][Index](Player,Info)
					end
				end

			end)
			Character.Humanoid.Died:Once(function()
				for _,Curse in pairs(Player_Data.Curses) do
					Service_Curses.unregister(Player,Curse)
				end
				Player_Events.Touched:Disconnect()
			end)
		end)
		return true
	end,

	player_unregister = function (Player:Player)
		Service_DataStore.Save(Player,"Stage",Player:GetAttribute("UserMaxstage"),true)
		local Player_Data = Players_Data[Player.Name]
		local P_Server_info = Player_Data.Server_info
		if P_Server_info.Kick_reason == "denied" then
			return
		end
		for _, Event in pairs(Player_Data.Events) do
			Event:Disconnect()
		end
		for _, Curse in pairs(Player_Data.Curses) do
			Enum_request.Curse.unregister(Player,{Curse = Curse})
		end
		Players_Data[Player.Name] = nil
	end
}

