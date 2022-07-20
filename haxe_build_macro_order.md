This sample shows how build types order works and can be upset using build macro https://try.haxe.org/#DD6AA298

```haxe
@:build(Macro.build())
class A {}

@:build(Macro.build())
class B {}

@:build(Macro.build())
class C {
	var a:A;
}

class Test {
	static function main() {
		trace("Haxe is great!");
	}
}
```
The basic order here is C, B, A, but everytime you will access another type (using `.get()`), it will change the order.
Here, if we uncomment #L37 or #L50, the order will be C, A, B

```haxe
import haxe.macro.Context;
import haxe.macro.Expr;

class Macro {
	static function build() {
		var t = Context.getLocalType();
		trace('Building : $t');

		var t2 = Context.getType("A");
		switch t2 {
			case TInst(rt3, _):
				trace(rt3);
				//var t3 = rt3.get();           // Will change build order
			case _:
		}
		var fields = Context.getBuildFields();
		for (field in fields) {
			if (field.name == "a") {
				switch field.kind {
					case FVar(p):
						var t2 = Context.resolveType(p, field.pos);
						//var t2 = haxe.macro.ComplexTypeTools.toType(p);
						switch t2 {
							case TInst(rt3, _):
								trace(rt3);
							  //var t3 = rt3.get();   // Will change build order
							case _:
						}
					case _:
				}
			}
		}
		return fields;
	}
}

```
