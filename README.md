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

But getmetatable sometimes grabs problematic metamethods which would end up returning an error. One of these metamethods being __metatable. These metafields are also used by devs to entirely lock tables (there will be an example of this at the end of the tutorial). This is why we use this function instead as it simply skips over the metamethod :
```lua
getrawmetatable(variant<Userdata, string, table> object); --skips over the __metatable metafield
```
The getrawmetatable function is extremely useful for grabbing the entire catalogue of metamethods in the game.
```lua
local mt = getrawmetatable(game); 

--Unfortunately the game table on roblox is locked and set to read-only making it impossible to edit it. Thankfully there's a way to fix this!

setreadonly(mt, false); -- makes the table writable
setreadonly(mt, true); -- makes the table read-only again

--This will be explained more in-depth further in the tutorial.
```

***Metamethods***

So what are metamethods and how can we use them to our advantage? A metamethod is essentially an event that runs whenever the properties of the table change. They are called using a __ prefix. A list of metamethods is as follows :
```diff
> __namecall
```
The __namecall metamethod is used to grab all the methods from a metatable. Methods are built-in functions that are called using colons. Some examples of this are :Kick(), :FireServer() etc... Here is a basic example of how __namecall is used :
```lua
local mt = getrawmetatable(game);
local oldMT = mt.__namecall; -- we're using the __namecall metamethod to backup our current table. We'll need this in the future.

setreadonly(mt, false); -- we're unlocking the table so we can edit it

mt.__namecall = function(self, ...)

```



