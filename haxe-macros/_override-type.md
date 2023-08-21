https://try.haxe.org/#bCc02c3f :

```haxe
class A {
	public static var foo:Int = 5;
	public static var extra:Array<String>;
}
```

```haxe
import haxe.macro.Context;
import haxe.macro.Type;
import haxe.macro.Expr;

class Macro {
	public static function build() {
		trace(Context.getLocalClass());
		return null;
	}
	public static function init() {
		Context.onAfterTyping(function(a) {
			// This is your extra field that is populated while building types
			// It can be stored whenever you want and just taken at the end, at this point i.e.
			var expr = macro ["name_one, name_two"];
			var texpr = Context.typeExpr(expr);
			//
			for (t in a) {
				switch t {
					case TClassDecl(cls):
						if (cls.toString() == "A") {
							var cl = cls.get();
							cl.exclude(); // We exclude the original class from compilation

							// And build the new one (the same as the old but we can now "complete" the "extra" field expression)
							var pos = Context.currentPos();
							var newt = {
								pack: [],
								name: "A_", // You have to give a dummy name if not Haxe will complain that "A" already exists (even if it's mark as excluded from compilation...)
								meta: [{name: ':native', params: [macro "A"], pos: pos}, {name: ':keep', pos: pos},],
								// Or meta: [{name: ':build', params: [macro Macro.build()], pos: pos}, {name: ':native', params: [macro "A"], pos: pos}, {name: ':keep', pos: pos},],
								fields: [
									for (field in cl.fields.get())
										classFieldToField(field, true, true)
								].concat([
									for (field in cl.statics.get()) {
										var cf = classFieldToField(field, true, true);
										if (cf.name == "extra") { // Our field to "complete"
											cf.kind = FVar(Context.toComplexType(texpr.t), expr);
										}
										cf;
									}
									]),
								kind: TDClass(),
								pos: cl.pos
							}
							Context.defineType(newt);
						}
					case _:
				}
			}
		});
		return null;
	}

	// Helper function
	static function classFieldToField(cf:ClassField, isStatic = false, withExpr = false) {
		var field = @:privateAccess haxe.macro.TypeTools.toField(cf);
		if (withExpr) {
			field.kind = switch cf.kind {
				case FVar(_, _):
					var te = cf.expr();
					var e = te != null ? Context.getTypedExpr(te) : null;
					FVar(null, e);
				case _: throw "Not implemented";
			}
		}
		if (isStatic) {
			field.access.push(AStatic);
		}
		return field;
	}
}
```
