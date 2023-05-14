# Destruct a structure using Haxe macros
This expression macro creates local variables according to the given structure fields. It's a kind of "Destruct" mecanism.

Full working example here : https://try.haxe.org/#Fab5481b
## Example 1
When you want to assign structure fields to custom variables (you can ommit some fields using `_`) :
```haxe
using Macro;

class Test {
	public static var arr : Array<Dynamic> = [];

	static function main() {
		foo().destruct( a, b, _, arr[ 1 ] );
		trace( a ); 	// 1
		trace( b ); 	// bar
		trace( arr ); 	// [ null, test ]
	}

	static function foo() {
		return { x: 1, y: "bar", z: true, r: "test" }
	}
}
```
## Example 2
When you want to have local variable with names as the structure fields :
```haxe
using Macro;

class Test {
	static function main() {
		foo().destruct();
		trace( x ); 	// 1
		trace( y ); 	// bar
    		trace( z ); 	// true
		trace( r ); 	// [ null, test ]
	}

	static function foo() {
		return { x: 1, y: "bar", z: true, r: "test" }
	}
}

```
## Expression macro
```haxe
#if macro
import haxe.macro.Expr;
import haxe.macro.Context;

using haxe.macro.Tools;
#end

class Macro {
	macro public static function destruct(...rest:Expr) {
		var a = rest.toArray();
		var estruct = a.shift();
		var fields = [];
		// Context.follow permits to use typedef instead on anonymous structure
		switch Context.follow(Context.typeExpr(estruct).t) {
			case TAnonymous(o):
				for (field in o.get().fields) {
					fields.push({name: field.name, pos: field.pos.getInfos().min});
				}
			case _:
				Context.fatalError("Should be a structure", estruct.pos);
		}
		if (a.length > 0 && a.length != fields.length) {
			Context.fatalError("Should have zero or same number of fields as structure", Context.currentPos());
		}
		// Here we reorder fields according to the structure fields order (if not, Haxe gives it as alphabetical order)
		fields.sort(function(o1, o2) {
			if (o1.pos < o2.pos) return -1;
			if (o1.pos > o2.pos) return 1;
			return 0;
		});
		var tmp = "_" + Std.random(1000);
		var bl = [macro var $tmp = $estruct];
		for (i in 0...fields.length) {
			var field = fields[i].name;
			var e = a[i];
			if (e == null) {
				e = macro $i{field};
			}
			var sv = e.toString();
			if (sv == "_")
				continue;

			// Here we check if the variable name already exists or should be created as local variable
			try {
				Context.typeExpr(e);
				bl.push(macro $e = $i{tmp}.$field);
			} catch (_) {
				bl.push(macro var $sv = $i{tmp}.$field);
			}
		}
		return macro @:mergeBlock $b{bl};
	}
}
```
