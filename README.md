

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local PlayerGui = Player.PlayerGui


local RS = game:GetService("ReplicatedStorage")
local PlayerRequest = RS.PlayerRequest
local Modules_Client = RS.Client
local Modules_RS = RS:WaitForChild "Replicated"
local Service_Visual = require(Modules_Client.Service_Visual)
local Service_Info = require(Modules_RS:WaitForChild("Service_Info"))
local Replicated_Info = Service_Info.Replicated
local Curses_level = Replicated_Info.Curse_level
local Client_Info = Service_Info.Client
local P_Data = Client_Info.Client_Data

local TS = game:GetService("TweenService")
local Tween_UI_Dissapear = TweenInfo.new(2,Enum.EasingStyle.Linear)
local TweenTime = TweenInfo.new(3)
local Time_05 = TweenInfo.new(5)
local SlowMotion = TweenInfo.new(3,Enum.EasingStyle.Quad)

-- COnfiguraciones --
local text_LOADING = "Loading"
local Random_texts = {
	"Have you ever wondered the meaning of the life?",
	"there was a long time before a child who wanted to play some obby games, but what he didn't  notice is that this game would be so hard, now he don't wanna play roblox anymore",
	"Sometimes i believe that there's no sense on going to school, but idk"
}

local Answers = {
	"no i didn't.",
	"okay i don't care",
	"get out!"
}

local function Server_Fire (Action:"Player_Teleport",Info,ExtraInfo)
	PlayerRequest:FireServer(Action,Info,ExtraInfo)
end

local function GUI_Tween_update_visibility (GUI:GuiObject,boolean:true|false)
	local Transparency = boolean and 0 or 1
	local AnimObjects = {}
	for _,Children in pairs(GUI:GetDescendants()) do
		if Children:IsA("GuiObject") then
			AnimObjects[#AnimObjects+1] = Children
		end
	end
	for Index,UI in ipairs(AnimObjects) do
		local Animation = TS:Create(UI,Tween_UI_Dissapear,{
			BackgroundTransparency = Transparency
		})
		Animation:Play()
		if #AnimObjects == Index then
			Animation.Completed:Wait()
		end
	end
	return true
end

local function GUI_Visibility_Update (GUI:GuiBase,Visible:boolean,Enabled:boolean)
	GUI.Enabled = Enabled
	local Transparency = Visible and 0 or 1
	for _,UI in pairs(GUI:GetDescendants()) do
		if UI:IsA("GuiObject") then
			UI.Transparency = Transparency
		end
	end
end

local function GUI_Set_Main (...:string)
	for _,GUI in pairs(PlayerGui:GetChildren()) do
		if GUI:IsA("ScreenGui") then
			GUI.Enabled = false
		end
	end
	for _,GUI in pairs({...}) do
		PlayerGui:WaitForChild(GUI).Enabled = true
	end
end

local function text_create(text_type:"Classic"|"Player",text)
	local UI = PlayerGui:WaitForChild("Text")
	UI.Enabled = true
	for _,descendants in pairs(UI:GetChildren()) do
		descendants.Visible = false
	end
	local Text_Frame = UI:WaitForChild(text_type)
	if UI:GetAttribute("Using") then
		return warn("Animation is running!")
	end
	UI:SetAttribute("Using",true)
	Text_Frame.Visible = true
	local Text_font = Text_Frame.Font
	Text_font.Text = ""
	Text_font.Visible = true
	local Text_size = #text
	if Text_size > 0 then
		for i = 1,Text_size do
			Text_font.Text = string.sub(text,1,i)
			task.wait(0.05)
		end
	end
	UI:SetAttribute("Using")
	task.wait(5)
	if not UI:GetAttribute("Using") then
		Text_Frame.Visible = false
	end
end

local function screen_set_loading ()
	local UI_Loading = PlayerGui:WaitForChild("UI_Loading")
	GUI_Visibility_Update(UI_Loading,false,true)
	GUI_Tween_update_visibility(UI_Loading,true)
end

local Stages_info = {
	[1] = function(Lb,ft)
		if not Lb and ft then
			text_create("Classic",[[?:]].."Hello!, don't touch the grey parts")
		end
	end,
	[2] = function(Lb,ft)
		local cursed = #P_Data.Curses > 0
		if not Lb and ft then
			local Text = cursed and "Avoid the entity that is following you!" or
			"By touching the grey parts you will get a curse!..."
			text_create("Classic",[[?:]]..Text)
		end
	end,
	[4] = function (Lb,ft)
		if not Lb and ft then
			local cursed = #P_Data.Curses > 0
			local Text = cursed and "You can see the curse angry by checking his bar down this text!" or
				"You are impresive."
			text_create("Classic",[[?:]]..Text)	
		end
	end,
	[10] = function(Lb,ft)
		if not Lb and ft then
			text_create("Classic",[[?:This is your imagination.]])
		end
	end,
	[17] = function(Lb,ft)
		if not Lb and ft then
			text_create("Classic",[[?:That's why you can die the amount of times you want.]])
		end
	end,
	[20] = function(Lb,ft)
		if not Lb and ft then
			text_create("Classic",[[?:"This is the end of your meet, now is time to face the real world, good luck!]])
		end
	end,
	[21] = function(Lb,ft)
		if not Lb and ft then
			text_create("Player","You: ...")
		end
	end,
}

local function Frame_transparency (Frame,boolean)
	local Descendants = Frame:GetDescendants()
	Descendants[#Descendants+1] = Frame
	for _,UI:GuiObject in pairs(Descendants) do
		if UI:IsA("GuiObject") then
			UI.Transparency = boolean and 0 or 1
		end
	end
end


local RBXScriptConnections,AllTask,UI,Buttons = {},{},{},setmetatable({
	Close_Parent = function(Button:GuiObject)
		Frame_transparency(Button.Parent,false)
	end,
	Play = function(Button:GuiButton)
		if not Button:GetAttribute("Pressing") then
			Button:SetAttribute("Pressing",true)
		else
			return
		end
		screen_set_loading()
		Server_Fire("Player_Teleport","UserStage")
		Button:SetAttribute("Pressing")
	end,
},{__index = function(table,index)
	warn("no se encontro la indexacion : ",index)
	return function()
		
	end
end,})




-- Datos internos --
local Table_Index_Loading = {
	UI_Dead = {
		Load = function(GUI)
			for _,Descendant in pairs(GUI:GetDescendants()) do
				Descendant.Visible = false
			end
			GUI.Enabled = true
		end,
	},
	UI_Loading = {
		Load = function(GUI:GuiMain)
		GUI.Enabled = false
		end,
		Enable = function(GUI)
			
			local TextLabel = GUI.Space.PointText
			TextLabel.Text = text_LOADING
			AllTask[GUI.Name] = AllTask[GUI.Name] or {}
			local CurrentTable = AllTask[GUI.Name]
			CurrentTable[#CurrentTable+1] = task.spawn(function()
				local Number = 0
				while GUI.Parent do
					Number += 1
					TextLabel.Text = TextLabel.Text.."."
					task.wait(0.5)
					if Number == 3 then
						Number = 0
						TextLabel.Text = text_LOADING
					end
				end
			end)
		end,
		Disable = function (GUI)
			for _,Atask in pairs(AllTask[GUI.Name]) do
				task.cancel(Atask)
			end
		end,
	},
	UI_Starter = {
		Load = function(GUI)
		GUI.Enabled = true
		local Space = GUI:WaitForChild("Space")
		local Frame_random_text = Space:WaitForChild("Random_Text")
		local TextLabel_1 = Frame_random_text:WaitForChild("Text")
		local TextLabel_2 = Frame_random_text:WaitForChild("Close_Parent")
		local Index = math.random(1,#Random_texts)
		TextLabel_1.Text = Random_texts[Index]
		TextLabel_2.Text = Answers[Index]
		end,
	},
	UI_Game = {
		Load = function(GUI)
			GUI.Enabled = false
			local Space = GUI:WaitForChild("Space")
			local Curse_Frame = Space:WaitForChild("Curse")
			Curse_Frame.Visible = false
		end,
	},
	Text = {
		Load = function(GUI)
			GUI.Enabled = true
			for _,Descendant in pairs(GUI:GetDescendants()) do
				Descendant.Visible = false
			end
		end,
	}
}

local Curses_info = {
	Destruction = {
		Curse_function = function (Frame)
			Frame.Visible = true
			task.wait(1)
			Frame.Visible = false
		end,
	}
}



return {
	screen = {
		set = {
			loading = {
				screen_set_loading = screen_set_loading
			},
		}	
	},
	load_GUI = function(GUI)
		GUI.ResetOnSpawn = false
		local GUI_Name = GUI.Name
		for _,Event in pairs(RBXScriptConnections[GUI_Name] or {}) do
			Event:Disconnect()
		end
		RBXScriptConnections[GUI_Name] = {}
		local GUI_Functions = Table_Index_Loading[GUI_Name]
		if GUI_Functions then
			GUI_Functions.Load(GUI)
		end
		local Current_Table = RBXScriptConnections[GUI_Name]
		for _,Child in pairs(GUI:GetDescendants()) do
			if Child:IsA("GuiButton") then
				Current_Table[#Current_Table+1] = Child.MouseButton1Click:Connect(function()
					Buttons[Child.Name](Child)
				end)
			end
		end
	end,
	Debug = {
		Player_teleported_succesfully = function()
			GUI_Set_Main("UI_Game","UI_Loading")
			GUI_Tween_update_visibility(PlayerGui:WaitForChild("UI_Loading"),false)
		end,
	},
	Stage_Update = function(Stage,First_time)
		PlayerGui:WaitForChild("UI_Game").Space.Stage.Text.Text = "Stage : "..tostring(Stage)
		local Stage_function = Stages_info[Stage]
		local Lobby = P_Data.Lobby
		if Stage_function then
			Stage_function(Lobby,First_time)
		end
	end,
	Curse_Update = function(Data)
		local Anger = Data.Anger
		local Game_Space = PlayerGui:WaitForChild("UI_Game"):WaitForChild("Space")
		local Frame_Curse = Game_Space:WaitForChild("Curse")	
		local P_Curses = P_Data.Curses
		if #P_Curses > 0 then
			local Anger_Frame = Frame_Curse:WaitForChild("Anger")
			if Anger then
				Anger_Frame.Text.Text = "Anger : "..tostring(Anger)
			end
			if Anger == 1 then
				Anger_Frame.T = 1
			Frame_Curse.Visible = true
				local Tween = TS:Create(Anger_Frame,Time_05,{Transparency = 0})
			Tween:Play()
			end
		else
			Frame_Curse.Visible = false
		end
	end,
	Kill = function()
		local GUI = PlayerGui:WaitForChild "UI_Dead"
		local P_Curses = P_Data.Curses
		local Greatest_Curse = {Curse = nil,level = -1}
		for _,Curse in ipairs(P_Curses) do
			local Curse_level = Curses_level[Curse]
			if Curse_level > Greatest_Curse.level then
				Greatest_Curse.level = Curse_level
				Greatest_Curse.Curse = Curse
			end
		end
		local Curse = Greatest_Curse.Curse
		local Frame = GUI:WaitForChild(Curse)
		GUI.Enabled = true
		local Curse_function = Curses_info[Curse].Curse_function
		local Succes = Curse_function and Curse_function(Frame)
	end,
}
