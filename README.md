# Metatables & Metamethods - A basic guide to understanding how they integrate into the ROBLOX LUA environment (and how they can be exploited).

***Metatables***

So what exactly are metatables? To put it into simple terms, it's a table inside of a table. Although they can store data like normal tables, they can also contain what are called metamethods. Metamethods can not be accessed like regular data. This is where a custom exploit function comes into play : 
```lua
getmetatable(variant<Userdata, string, table> object);
```
```lua
local testTable = {};
local mt = getmetatable(testTable);
print(type(mt));

--Output : Table
```

But getmetatable sometimes grabs problematic metamethods which would end up returning an error. One of these metamethods being `__metatable`. These metafields are also used by devs to entirely lock tables (there will be an example of this at the end of the tutorial). This is why we use this function instead as it simply skips over the metamethod :
```lua
getrawmetatable(variant<Userdata, string, table> object); --skips over the __metatable metafield
```
The getrawmetatable function is extremely useful for grabbing the entire catalogue of metamethods in the game.
```lua
local mt = getrawmetatable(game); 

--Unfortunately the game table on roblox is locked and set to read-only, normally making it impossible to edit it. Thankfully there's a way to fix this!

setreadonly(mt, false); -- makes the table writable
setreadonly(mt, true); -- makes the table read-only again

--This will be explained more in-depth further in the tutorial.
```

***Metamethods***

So what are metamethods and how can we use them to our advantage? A metamethod is essentially an event that runs whenever the properties of the table change. They are called using a __ prefix. A list of metamethods is as follows :
```diff
> __namecall
```
The `__namecall` metamethod is used to grab all the methods from a metatable. Methods are built-in functions that are called using colons. Some examples of this are `:Kick()`, `:FireServer()` etc... Here is a basic example of how `__namecall` is used :
```lua
local mt = getrawmetatable(game);
local oldMT = mt.__namecall; -- we're using the __namecall metamethod to backup our current table. We'll need this in the future.

setreadonly(mt, false); -- we're unlocking the table so we can edit it

mt.__namecall = function(self, ...)
local args = {...};
local getMethod = getnamecallmethod();
    if getMethod == 'Kick' then
        print(self:GetFullName());
        warn('Uh oh the game tried to kick you');
        return nil;
    end

    return oldMT(self, ...);
end

setreadonly(mt, true)
```
Lets break this down one by one. At the start we're defining a variable, mt to contain the metatable of 'game'. We'll need to back it up using `__namecall` as we'll need this to prevent the game from breaking. Next we use `setreadonly(mt, false)` to make the game metatable writable. Later we define a function with the arguments `self` & `...`. `self` is the object that's being fired by the method. `...` is the compilation of all the arguments and methods that `mt.__namecall` returns. Next we'll assign `...` to a table using the variable args. What this function will currently do is go through all of the methods being fired in the game metatable and return the method with its assigned arguments in the args table. But we're not done yet! We need to somehow differentiate between methods and their arguments! Before the LuaU update it was possible to grab the method by taking the last index out of the args table, however, as of writing this guide the only way to get the method is by using this option : 
```lua
getnamecallmethod();
```
This function will return the current method being used. Next we'll check if `getMethod` is the specific method that we're looking for by using an if statement. In this case we'll be looking for the `Kick()` method. If it so happens that it's detected then the game will print the path of the object thats being fired by the `Kick()` method, using `GetFullName()`. In this case it will only return the path of your character regardless as metamethods can only be used to bypass clientside methods! Methods fired on the serverside are impossible to avoid. Next we'll have it return nil as the result,essentially rendering `Kick()` useless by giving it `nil` as the argument instead of the player. Next, at the end of the function we'll return the cloned and unedited metatable with the current arguments the function is using to avoid breaking the game. At the end of the script we'll lock the table using `setreadonly()` as to avoid accidentally editing the table and breaking the game! Depending on the script, you might sometimes want to leave the metatable unlocked. Congrats, you've succesfully made a functioning clientside anti-kick!

Also here's a small explanation of how the arguments in the `mt.__namecall` function return their values. Lets assume that the method being fired in question is :  
```lua 
game.Players.LocalPlayer:FireServer('Sit', game.Workspace.BasePlate)
```
Lets take a look at a simplified version of the script above : 
```lua
local mt = getrawmetatable(game);
local oldMT = mt.__namecall;

setreadonly(mt, false);

mt.__namecall = function(...)
    local args = {...};
    local getMethod = getnamecallmethod();

    if getMethod == "FireServer" then
        for i, v in pairs(args) do
            warn(i.." is assigned to : ", v);
        end
    end

    return oldMT(...);
end

setreadonly(mt, true)
```
The output will looks something like this :
```lua
> 1 is assigned to : LocalPlayer 
> 2 is assigned to : Sit
> 3 is assigned to : BasePlate
```
In the anti-kick script above I used `self` as an argument simply for the sake of simplicity & also to indicate that userdata is returned as an object, and that to get the full path we can use `GetFullName()`. Now that we know how the arguments are indexed we can entirely avoid using self. Assuming that we're still using the args table, we can just grab the `self` argument by indexing it as `args[1]`.

The next metamethod we'll be reviewing is :
```diff
> __index
```




