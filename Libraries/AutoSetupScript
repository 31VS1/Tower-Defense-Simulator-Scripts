local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RemoteFunction = ReplicatedStorage:WaitForChild("RemoteFunction")
local RemoteEvent = ReplicatedStorage:WaitForChild("RemoteEvent")

local function printLog(message)
    local logFile = "TDS_AutoStrat/LastPrintLog.txt"
    appendfile(logFile, tostring(message) .. "\n")
    print(tostring(message))
end

local function equipTroop(troop)
    local troops = {}
    for index, _ in next, RemoteFunction:InvokeServer("Session", "Search", "Inventory.Troops") do
        table.insert(troops, index)
    end

    if not troop or troop == "Nil" then
        troop = "nil"
    end

    if tostring(troop) ~= "nil" and not table.find(troops, tostring(troop)) then
        local errorMessage = table.concat({"\n\n---------- AUTO STRAT ----------\n\nError 2:\nYou don't own ", tostring(troop), " troop.\n\n---------- AUTO STRAT ----------\n"})
        game.Players.LocalPlayer:Kick(errorMessage)
    end

    RemoteEvent:FireServer("Inventory", "Equip", "tower", troop)

    if not getgenv().GoldenPerks then
        getgenv().GoldenPerks = {}
    end

    if table.find(getgenv().GoldenPerks, troop) then
        RemoteFunction:InvokeServer("Inventory", "Equip", "Golden", troop)
    else
        RemoteFunction:InvokeServer("Inventory", "Unequip", "Golden", troop)
    end

    getgenv().status = "Equipped "..troop
end

local function unEquip()
    for towerName, tower in next, RemoteFunction:InvokeServer("Session", "Search", "Inventory.Troops") do
        if tower.Equipped then
            RemoteFunction:InvokeServer("Inventory", "Unequip", "Tower", towerName)
        end
    end
end

local function joinElevator(elevator)
    local mp = elevator.State.Map.Title
    local plrs = elevator.State.Players
    local rq = require(elevator.Settings).Type

    if getgenv().Maps[mp.Value] and rq == getgenv().MultiStratType then
        if plrs.Value <= 0 then
            printLog("Joining elevator...")
            getgenv().status = "Joining..."
            RemoteFunction:InvokeServer("Elevators", "Enter", elevator)
            unEquip()

            for _, troop in next, getgenv().Maps[mp.Value] do
                equipTroop(troop)
            end

            printLog("Joined elevator...")
            getgenv().status = "Joined"

            while true do
                getgenv().status = "Joined ("..elevator.State.Timer.Value.."s)"

                if elevator.State.Timer.Value == 0 then
                    local someoneJoined = false

                    for _=1,100 do
                        if plrs.Value > 1 then
                            printLog("Someone joined, leaving elevator...")
                            getgenv().status = "Someone joined..."
                            RemoteFunction:InvokeServer("Elevators", "Leave")
                            return
                        end
                        task.wait(0.01)
                    end

                    if elevator.State.Timer.Value == 0 andplrs.Value <= 1 and not someoneJoined then
                        printLog("No one joined, leaving elevator...")
                        getgenv().status = "No one joined..."
                        RemoteFunction:InvokeServer("Elevators", "Leave")
                        return
                    end
                end

                task.wait(0.01)
            end
        end
    end
end

local function onElevatorAdded(elevator)
    if elevator.State.Players.Value == 0 then
        printLog("Elevator added, joining...")
        getgenv().status = "Elevator added..."
        joinElevator(elevator)
    end
end

local function onElevatorChanged(elevator, property)
    if property == "Players" and elevator.State.Players.Value == 0 then
        printLog("Elevator changed, joining...")
        getgenv().status = "Elevator changed..."
        joinElevator(elevator)
    end
end

game.ReplicatedStorage:WaitForChild("Elevators"):GetPropertyChangedSignal("ElevatorAdded"):Connect(onElevatorAdded)
game.ReplicatedStorage:WaitForChild("Elevators"):GetPropertyChangedSignal("ElevatorChanged"):Connect(onElevatorChanged)

printLog("Auto Strat Script Loaded...")
getgenv().status = "Script Loaded"
