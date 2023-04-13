... piece of ftk.macro.Compiler file : 

```haxe
public static function addGlobalMetadata( pathFilter:String, meta:String, ?recursive:Bool = true, ?ignore : Array<String> ){
	processModule( function(cl)haxe.macro.Compiler.addMetadata( meta, cl ), pathFilter, recursive, ignore );
}

public static function processModule(f : String->Void, pack:String, ?rec = true, ?ignore:Array<String>, ?classPaths:Array<String>, strict = false) {
	var ignoreWildcard:Array<String> = [];
	var ignoreString:Array<String> = [];
	if (ignore != null) {
		for (ignoreRule in ignore) {
			if (StringTools.endsWith(ignoreRule, "*")) {
				ignoreWildcard.push(ignoreRule.substr(0, ignoreRule.length - 1));
			} else {
				ignoreString.push(ignoreRule);
			}
		}
	}
	var skip = if (ignore == null) {
		function(c) return false;
	} else {
		function(c:String) {
			if (Lambda.has(ignoreString, c))
				return true;
			for (ignoreRule in ignoreWildcard)
				if (StringTools.startsWith(c, ignoreRule))
					return true;
			return false;
		}
	}
	var displayValue = Context.definedValue("display");
	if (classPaths == null) {
		classPaths = Context.getClassPath();
		// do not force inclusion when using completion
		switch (displayValue) {
			case null:
			case "usage":
			case _:
				return;
		}
		// normalize class path
		for (i in 0...classPaths.length) {
			var cp = StringTools.replace(classPaths[i], "\\", "/");
			if (StringTools.endsWith(cp, "/"))
				cp = cp.substr(0, -1);
			if (cp == "")
				cp = ".";
			classPaths[i] = cp;
		}
	}
	var prefix = pack == '' ? '' : pack + '.';
	var found = false;
	for (cp in classPaths) {
		var path = pack == '' ? cp : cp + "/" + pack.split(".").join("/");
		if (!sys.FileSystem.exists(path) || !sys.FileSystem.isDirectory(path))
			continue;
		found = true;
		for (file in sys.FileSystem.readDirectory(path)) {
			if (StringTools.endsWith(file, ".hx") && file.substr(0, file.length - 3).indexOf(".") < 0) {
				if( file == "import.hx" ) continue;
				var cl = prefix + file.substr(0, file.length - 3);
				if (skip(cl))
					continue;
				f( cl );
			} else if (rec && sys.FileSystem.isDirectory(path + "/" + file) && !skip(prefix + file))
			processModule(f, prefix + file, true, ignore, classPaths);
		}
	}
	if (strict && !found)
		Context.error('Package "$pack" was not found in any of class paths', Context.currentPos());
}
```
