local Library = loadstring(game:HttpGet('https://raw.githubusercontent.com/LuauCloud/Byte/refs/heads/main/Utils/Library.lua'))()
local Library_Window = Library.Add_Window('Acceptions')


local BladeBall_Tab = Library_Window.Create_Tab({name = 'Blade Ball', icon = 'rbxassetid://6023426975'})
local BladeBall_Section = BladeBall_Tab.Create_Section()

BladeBall_Section.Create_Toggle({
    name = 'Auto Parry',
    flag = 'Auto_Parry',
    callback = function(state)
        auto_parry_enabled = state
        if state then
            task.spawn(AutoParry)
        end
        debugPrint("Auto Parry", state and "Enabled" or "Disabled")
    end
})
