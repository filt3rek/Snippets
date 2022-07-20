Documentation to see : 
- https://haxe.org/manual/type-system-modules-and-paths.html

```haxe
@:build(Macro.build())
private class A {}

class B {}

class Test {
	static function main() {
		trace("Haxe is great!");
	}
}

class _Test {}	// Class named _Test it's ok, but module can't be named starting with an _
```

```haxe
import haxe.macro.Context;

class Macro {
	static function build() {
		var a = ["A", "B"];
		for (o in a) {
			var t1 = Context.getType(o);
      var t2 = Context.getType('Test.$o');
			var ct1 = haxe.macro.TypeTools.toComplexType(t1);
			var ct2 = haxe.macro.TypeTools.toComplexType(t2);
			trace(t1);
			trace(t2);
			trace(ct1);
			trace(ct2);
		}

		return null;
	}
}
```
Output : 
```
Macro.hx:11: TInst(_Test.A,[])
Macro.hx:12: TInst(_Test.A,[])
Macro.hx:13: TPath({name: A, params: [], pos: #pos((unknown)), pack: []})
Macro.hx:14: TPath({name: A, params: [], pos: #pos((unknown)), pack: []})
Macro.hx:11: TInst(B,[])
Macro.hx:12: TInst(B,[])
Macro.hx:13: TPath({name: Test, params: [], sub: B, pos: #pos((unknown)), pack: []})
Macro.hx:14: TPath({name: Test, params: [], sub: B, pos: #pos((unknown)), pack: []})
```

Notes :
- Private types have modules named starting with an _
- If sub != null => name = Module, sub = Type. Otherwise name = Type
