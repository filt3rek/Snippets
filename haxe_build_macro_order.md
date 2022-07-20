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
				//var t3 = rt3.get();           // (1) Will change build order
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
							  //var t3 = rt3.get();   // (2) Will change build order
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
**Notes :**
1. The basic order here is C, B, A, but everytime you will access another type (using `.get()`), it will change the order.
Here, if we uncomment (1) or (2), the order will be C, A, B
2. The build order isn't changed when (1) and (2) are commented even if C has a field of type A, I suppose the build macro doesn't type the fields
automatically as I understand here :
> Build Macros: These are defined for classes, enums and abstracts through the @:build or @:autoBuild metadata. They are executed per type, after the type has been set up (including its relation to other types, such as inheritance for classes) but before its fields are typed (see Type Building).

**Pending questions :**
1. I don't really know the difference between `haxe.macro.ComplexTypeTools.toType(p)` and `Context.resolveType(p, field.pos)`
