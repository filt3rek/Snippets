https://try.haxe.org/#feF925fc

```haxe
class Test {
	static function main() {
#if js
		MyLogger.haxeTrace = haxe.Log.trace;
		haxe.Log.trace = MyLogger.jsTrace;
#end
		trace("Test class", Test);
	}
}
```

```haxe
#if js
class MyLogger {
	public static dynamic function haxeTrace(v:Dynamic, ?infos:haxe.PosInfos) {}
	public static function jsTrace(v:Dynamic, ?infos:haxe.PosInfos) {
		var args = [v].concat(infos?.customParams ?? []);
		for (o in args) {
			switch Type.typeof(o) {
				case Type.ValueType.TObject:
					js.Browser.console.dir(o);
				case _:
					haxeTrace(o);
			}
		}
	}
}
#end
```
