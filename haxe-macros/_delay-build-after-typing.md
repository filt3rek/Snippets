It overrides a base class after typing is done.
So basically it delays a `@:build` macro process after all is typed

```haxe
public static function delayBuildAfterTyping( cls : String, build : Expr ) {
	haxe.macro.Compiler.addGlobalMetadata( cls, "@:build( ftk.macro.Compiler.collectFields() )" );
	Context.onAfterTyping(function( a ) {
		for ( t in a ) {
			switch t {
				case TClassDecl( rcl ):
					if ( rcl.toString() == cls ) {
						var cl = rcl.get();
						cl.exclude();

						var pos 	= Context.currentPos();
						var newT 	= {
							pack	: cl.pack,
							name	: cls +	Std.random( 1000 ), // You have to give a dummy name if not Haxe will complain that "A" already exists (even if it's mark as excluded from compilation...)
							meta	: cl.meta.get().concat( [
								{ name: ':native', params: [ macro $v{cls} ], pos: pos },
								{ name: ':build', params: [macro $build()], pos: pos },
								{ name: ':keep', pos: pos },
							] ),
							fields	: collectedClassFields.get( cls ),
							kind	: {
								var superClassTP	= cl.superClass != null ? @:privateAccess haxe.macro.TypeTools.toTypePath( cl.superClass.t.get(), cl.superClass.params ) : null;
								TDClass(superClassTP, cl.interfaces.map(i->@:privateAccess haxe.macro.TypeTools.toTypePath( i.t.get(), i.params ) ), cl.isInterface, cl.isFinal, cl.isAbstract);
							},
							pos	: cl.pos
						}
						Context.defineType( newT, cl.module );
					}
				case _:
			}
		}
	});
	return null;
}

static var collectedFields : Map<String, Array<Field>> = [];
public static function collectFields() {
	collectedClassFields.set( Context.getLocalClass().toString(), Context.getBuildFields() );
	return null;
}
```
