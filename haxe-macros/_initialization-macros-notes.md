Documentation to see :
- https://haxe.org/manual/macro-initialization.html

**Some notes about init macros :**

- Init macros are executed in order they appear in the .hxml haxe build file
- You can add [`?pos : haxe.PosInfos`](https://api.haxe.org/haxe/PosInfos.html) as argument at the end of a init macro function in order to get the line in the .hxml haxe build file which executes this function.
