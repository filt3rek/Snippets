Delaying a building macro in case of cyclical redundancy
```haxe
@:build(Macro.build())
class A {
	var b:B;
}

@:build(Macro.build())
class B {
	var a:A;
}
```

https://try.haxe.org/#c3aBc327
```haxe
import haxe.macro.Context;
import haxe.macro.Expr;
import haxe.macro.Type;

class Macro {
	static var delayed = [];
	static var isATSet = false;
	static var it = 0;

	static function build() {
		var rcl = Context.getLocalClass();
		trace("Building " + rcl);
		var fields = Context.getBuildFields();
		for (field in fields) {
			switch field.kind {
				case FVar(TPath({name: n, sub: sub, pack: p})), FProp(_, _, TPath({name: n, sub: sub, pack: p})):
          // Be careful TPath "module declaration" : name is module, sub is type !
					if (sub != null) {
						p.push(n);
						p.push(sub);
					} else {
						p.push(n);
					}

					var t = Context.getType(p.join("."));
					switch t {
						case TInst(_t, _):
							var __t = _t.get();
							var _fields = __t.fields.get(); // Here fields are empty because of cyclical redundancy
							if (_fields.length == 0) {
								delayed.push(rcl);
							}
							trace('Class : $rcl', 'Field : ${field.name}', 'Field type : ${_t.toString()}', 'Type\'s fields : ${_fields.map(o -> o.name)}');
						case _:
					}
				case _:
			}
		}
		if (!isATSet) {
			Context.onAfterTyping(function(_) {
				trace("onAfterTyping pass " + it++);
				var nts = [];
				var rcl = null;
				while ((rcl = delayed.pop()) != null) {
					var cl = rcl.get();
					cl.exclude(); // Exclude does'nt really exclude, but mark class as "extern"
					var ncl = classToTypeDefinition(cl, cl.name + "_Delayed_" + it);
					Context.defineType(ncl);
				}
			});
			isATSet = true;
		}
		return null;
	}

	static function classToTypeDefinition(cl:ClassType, newName:String):TypeDefinition {
		var params = [];
		for (i in cl.params) {
			params.push({name: i.name, constraints: null, params: null});
		}
		var lsc = cl.superClass;
		var scDef = null;
		if (lsc != null) {
			var sc = lsc.t.get();
			scDef = {pack: sc.pack, name: sc.name};
		}
		var interfaces = [];
		for (i in cl.interfaces) {
			var ii = i.t.get();
			interfaces.push({
				pack: ii.pack,
				name: ii.name,
				sub: null,
				params: null
			});
		}

		var fields = [];
		for (field in cl.fields.get()) {
			fields.push(classFieldToField(field, false));
		}
		for (field in cl.statics.get()) {
			fields.push(classFieldToField(field, true));
		}
		var metas = cl.meta.get();
		// metas.push({name: "@:keep", pos: Context.currentPos()});
		var o = {
			pack: cl.pack,
			name: newName,
			doc: cl.doc,
			pos: cl.pos,
			meta: metas,
			params: params,
			isExtern: cl.isExtern,
			kind: TDClass(scDef, interfaces, cl.isInterface),
			fields: fields
		}
		// trace(o);
		return o;
	}

	static function classFieldToField(cf:ClassField, isStatic = false):Field {
		var field = @:privateAccess haxe.macro.TypeTools.toField(cf);
		if (isStatic)
			field.access.push(AStatic);
		// trace( cf );
		// trace( field );
		return field;
	}
}

```
