Here is how we can get a kind of partial typedef with dynamic properties even parametrized : [https://try.haxe.org/#050dF777](https://try.haxe.org/#050dF777)

```haxe
import haxe.macro.Expr;
import haxe.macro.Context;

using haxe.macro.Tools;

abstract ADyn<T>(T) from T to T {
	@:op(A.B) function get(prop:String) {
		if (Reflect.hasField(this, prop))
			return Reflect.field(this, prop);
		var data = Reflect.field(this, "_data_");
		return Reflect.field(data, prop);
	}

	@:op(A.B) macro function set(obj:haxe.macro.Expr, prop:haxe.macro.Expr, val:haxe.macro.Expr) {
		var sprop = switch prop.expr {
			case EConst(CString(s)): s;
			case _: null;
		}
		var fieldNames = [];
		var fieldSTypes = [];
		var tobj = Context.typeExpr(obj);
		var stparam = null;
		switch Context.followWithAbstracts(tobj.t) {
			case TInst(t, p):
				stparam = p[0].toString();
				var fields = t.get().fields.get();
				for (field in fields) {
					fieldNames.push(field.name);
					fieldSTypes.push(field.type.toString());
				}
			case _:
		}
		var valType = Context.typeExpr(val).t;
		var ind = fieldNames.indexOf(sprop);
		if (ind > -1) {
			if (valType.toString() != fieldSTypes[ind]) {
				Context.fatalError(fieldSTypes[ind] + " expected", val.pos);
			}
			return macro Reflect.setField($obj, $prop, $val);
		}
		if (stparam != "Dynamic" && valType.toString() != stparam) {
			Context.fatalError(stparam + " expected", val.pos);
		}
		return macro Reflect.setField($obj._data_, $prop, $val);
	}
}

@:structInit
class Dyn<T> {
	public var foo:Int;

	var _data_:Dynamic;

	public function new(foo:Int) {
		this._data_ = {}
		this.foo = foo;
	}
}

class Test {
	static function main() {
		var mydynobj:ADyn<Dyn<Dynamic>> = {foo: 1}
		mydynobj.bar = "barry";
		mydynobj.foo = 3;
		trace(mydynobj.foo, mydynobj.bar);

		var mydynobj2:ADyn<Dyn<Float>> = {foo: 1}
		// mydynobj2.bar = "barry"; // Float expected
		// mydynobj2.foo = "barry"; // Int expected
		trace(mydynobj2.foo, mydynobj2.bar);
	}
}
```
