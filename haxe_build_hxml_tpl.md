build.hxml :
  ```haxe
--macro Build.run()
```

build.tpl :
```haxe
-js %LOGNAME%.js
-D %LANG%
```

Build.hx : 
```haxe
class Build{
    static var ENV    = Sys.environment();
    static var r    = ~/(%([a-z_]+)%)/gmi;

    static function run(){
        var content    = sys.io.File.getContent( "build.tpl" );
        Sys.command( "haxe", r.map( content, (e)->ENV[ e.matched( 2 ) ] ).split( "\r\n" ) );
    }
}
```
