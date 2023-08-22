It overrides a base class after typing is done.
So basically it delays a `@:build` macro process after all is typed

```haxe
public static function delayBuildAfterTyping( cls : String, build : Expr ) {
	haxe.macro.Compiler.addGlobalMetadata( cls, "@:build( ftk.macro.Compiler.collectClassFields() )" );
	Context.onAfterTyping(function( a ) {
		for ( t in a ) {
			switch t {
				case TClassDecl( rcl ):
					if ( rcl.toString() == cls ) {
						var cl = rcl.get();
						cl.exclude();

						var supercl	= cl.superClass;

						var pos 	= Context.currentPos();
						var newT 	= {
							pack	: cl.pack,
							name	: cls +	Std.random( 1000 ), // You have to give a dummy name if not Haxe will complain that "A" already exists (even if it's mark as excluded from compilation...)
							meta	: [
								{ name: ':native', params: [ macro $v{cls} ], pos: pos },
								{ name: ':build', params: [macro $build()], pos: pos },
								{ name: ':keep', pos: pos },
							],
							fields	: collectedClassFields.get( cls ),
							kind	: TDClass(@:privateAccess haxe.macro.TypeTools.toTypePath( supercl.t.get(), supercl.params ), cl.interfaces.map(i->@:privateAccess haxe.macro.TypeTools.toTypePath( i.t.get(), i.params ) ), cl.isInterface, cl.isFinal, cl.isAbstract),
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

static var collectedClassFields : Map<String, Array<Field>> = [];
public static function collectClassFields() {
	collectedClassFields.set( Context.getLocalClass().toString(), Context.getBuildFields() );
	return null;
}
```
