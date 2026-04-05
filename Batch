local Rep = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Packager = require(Rep.Packager)

local Batch = {}

local Connections = {}

local DataBatch = {}
local SinglesDataBatch = {}

local BatchUpdate

Batch.Init = function()
	BatchUpdate = Rep.Remotes.Events.BatchUpdate
end

function Batch.AddToBatch(Tag:string, Data)
	if not DataBatch[Tag] then
		DataBatch[Tag] = {}
	end
	table.insert(DataBatch[Tag], Data)
end

function Batch.AddToBatchSingleClient(Tag:string, Data, Player)
	if RunService:IsClient() then return end
	
	if not SinglesDataBatch[Tag] then
		SinglesDataBatch[Tag] = {}
	end
	table.insert(SinglesDataBatch[Tag], {Data = Data, Player = Player})
end

-- This will append data to the batch if it does not find every single key in the search key
-- It must find every single key and it will insert all data into that spot only if every key is found
function Batch.SmartAddToBatchAllKeys(Tag:string, Data, SearchKey) -- SearchKey -- {"Key", "Key2", "Key3"}
	
	local dataNotFound = false
	if not DataBatch[Tag] then
		DataBatch[Tag] = {}
		table.insert(DataBatch[Tag], Data)
		return
	end
	
	local searchedData
	for key,_ in pairs(DataBatch[Tag]) do
		dataNotFound = false
		
		searchedData = DataBatch[Tag][key]
		for i = 1, #SearchKey do
			if searchedData[SearchKey[i]] then
				searchedData = searchedData[SearchKey[i]]
			else
				dataNotFound = true
				break
			end
		end
		if dataNotFound == false then
			break
		end
	end
	
	if dataNotFound then
		table.insert(DataBatch[Tag], Data)
	else
		dataNotFound = false
		local dataToAdd = Data
		for i = 1, #SearchKey do
			if dataToAdd[SearchKey[i]] then
				dataToAdd = dataToAdd[SearchKey[i]]
			else
				dataNotFound = true
				break
			end
		end
		if dataNotFound then
			warn("Search Key is invalid for the Data sent!")
		else
			for key, value in pairs(dataToAdd) do
				searchedData[key] = value
			end
		end
	end
	
	print(DataBatch[Tag])
	
end


-- This will search for as many keys as it can and if it finds any at all itll insert data
-- as far as it found regardless of if it found all of the keys
function Batch.SmartAddToBatchAnyKey(Tag:string, Data, SearchKey) -- SearchKey -- {"Key", "Key2", "Key3"}

	local dataNotFound = false
	local keysFound = 0
	if not DataBatch[Tag] then
		DataBatch[Tag] = {}
		table.insert(DataBatch[Tag], Data)
		return
	end

	local searchedData
	for key,_ in pairs(DataBatch[Tag]) do
		dataNotFound = false

		searchedData = DataBatch[Tag][key]
		for i = 1, #SearchKey do
			if searchedData[SearchKey[i]] then
				searchedData = searchedData[SearchKey[i]]
				keysFound += 1
			else
				dataNotFound = true
				break
			end
		end
		if dataNotFound == false or keysFound > 0 then
			break
		end
	end

	if dataNotFound and keysFound == 0 then
		table.insert(DataBatch[Tag], Data)
	else
		dataNotFound = false
		local dataToAdd = Data
		for i = 1, keysFound do
			if dataToAdd[SearchKey[i]] then
				dataToAdd = dataToAdd[SearchKey[i]]
			else
				dataNotFound = true
				break
			end
		end
		if dataNotFound then
			warn("Search Key is invalid for the Data sent!")
		else
			for key, value in pairs(dataToAdd) do
				searchedData[key] = value
			end
		end
	end
end


local IsServer = RunService:IsServer()

function SendBatch()
	local b = DataBatch
	DataBatch = {}
	local batchIsntEmpty = next(b)
	
	local c = SinglesDataBatch
	SinglesDataBatch = {}
	
	if IsServer then
		if batchIsntEmpty then
			BatchUpdate:FireAllClients(b)
		end
		for Tag, Data in pairs(c) do
			for _, Data in pairs(Data) do
				BatchUpdate:FireClient(Data.Player, {[Tag] = {Data.Data}})
			end
		end
	elseif batchIsntEmpty then
		BatchUpdate:FireServer(b)
	end
end

Batch.Start = function()
	if IsServer then
		BatchUpdate.OnServerEvent:Connect(function(plr, batch)
			if not batch then -- client joined
				return
			end
			
			for Tag, DataTable in pairs(batch) do
				if Connections[Tag] then
					for _, _function in pairs(Connections[Tag]) do
						for _, Data in pairs(DataTable) do
							_function(plr, Data)
						end
					end
				end
			end
		end)
	else
		BatchUpdate.OnClientEvent:Connect(function(batch)
			for Tag, DataTable in pairs(batch) do
				if Connections[Tag] then
					for _, _function in pairs(Connections[Tag]) do
						for _, Data in pairs(DataTable) do
							_function(Data)
						end
					end
				end
			end
		end)
		
		BatchUpdate:FireServer() -- Run whatever you want in this first function for intializing any data as soon as batch is ready
	end

	game:GetService("RunService").Heartbeat:Connect(SendBatch)
end

Batch.ConnectWithTag = function(_function, Tag)
	if not Connections[Tag] then
		Connections[Tag] = {}
	end
  table.insert(Connections[Tag], _function)
end

return Batch
