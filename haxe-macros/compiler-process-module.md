This piece of code applies `f()` to all **modules** in given *paths* exluding these in *ignore*

## Example of usage :
```haxe
public static function addGlobalMetadata(meta:String, paths:Array<String>, ?ignore:Array<String>) {
  processModule(function(module, _) haxe.macro.Compiler.addGlobalMetadata(module, meta, false), reducePaths(paths),
    ignore != null ? reducePaths(ignore) : null);
}
```
## Compiler macro function
```haxe
/*
 *	paths	: `my.not_recurive.package` or `my.recursive.package.`
 *	ignore	: `my.exact.Class` or `my.not_recurive.package` or `my.recursive.package.`
 */
public static function processModule(f:(module:String, file:String) -> Void, paths:Array<String>, ?ignore:Array<String>) {
  switch Context.definedValue("display") {
    case null, "usage":
    case _:
      return;
  }
  ignore ??= [];
  var classPaths = Context.getClassPath();
  for (i in 0...classPaths.length) {
    var cp = classPaths[i].split("\\").join("/");
    if (cp.endsWith("/"))
      cp = cp.substr(0, -1);
    if (cp == "")
      cp = ".";
    classPaths[i] = cp;
  }
  var processed = [];
  function checkDir(path:String, pack:String, recursive:Bool) {
    if (!sys.FileSystem.exists(path)) {
      return;
    }
    for (file in sys.FileSystem.readDirectory(path)) {
      if (sys.FileSystem.isDirectory(path + "/" + file) && recursive) {
        checkDir(path + "/" + file, pack == "" ? file : pack + "." + file, recursive);
      } else {
        if (file == "import.hx" || !file.endsWith(".hx") || file.substr(0, file.length - 3).indexOf(".") > -1)
          continue;
        var module = (pack == "" ? "" : pack + ".") + file.substr(0, -3);
        if (paths.indexOf(module) > -1) {
          if (processed.indexOf(path + "/" + file) == -1) {
            processed.push(path + "/" + file);
            f(module, path + "/" + file);
          }
        } else {
          for (gpath in paths) {
            if (module.indexOf(gpath) > -1 && ignore.indexOf(module) == -1) {
              var skip = false;
              for (gignore in ignore) {
                if (module.indexOf(gignore) > -1) {
                  if (gignore.endsWith(".")) {
                    skip = true;
                  } else {
                    var apack = module.split(".");
                    apack.pop();
                    if (gignore == apack.join(".")) {
                      skip = true;
                    }
                  }
                }
              }
              if (!skip) {
                if (processed.indexOf(path + "/" + file) == -1) {
                  processed.push(path + "/" + file);
                  f(module, path + "/" + file);
                }
              }
            }
          }
        }
      }
    }
  }

  for (cp in classPaths) {
    for (path in paths) {
      var recursive = false;
      if (path.endsWith(".")) {
        path = path.substr(0, -1);
        recursive = true;
      }
      var p = path == '' ? cp : cp + "/" + path.split(".").join("/");
      checkDir(p, path, recursive);
    }
  }
}
```
## Helper
This helper function *reduces* given paths. For example if you give `[ "my.pack.MyClass", "my.pack" ]` it reduces it to `[ "my.pack" ]` which already includes `my.pack.MyClass`...
```haxe
static function reducePaths(paths:Array<String>) {
  if (paths.indexOf(".") > -1)
    return ["."];

  paths = paths.reduce();
  if (paths.length == 1)
    return paths;

  haxe.ds.ArraySort.sort(paths, (s1, s2) -> s1 < s2 ? -1 : s1 > s2 ? 1 : 0);

  function partPath(s:String) {
    var a = s.split(".");
    var tmp = a.pop();
    var fc = tmp.charCodeAt(0);
    var isType = fc != null && fc > 64 && fc < 91;
    var o = {
      pack: a.join("."),
      isType: isType,
      isRec: tmp == ""
    }
    return o;
  }

  var _paths = [];
  for (path in paths) {
    if (_paths.length == 0) {
      _paths.push(path);
      continue;
    }
    var ppath = partPath(path);

    var i = 0;
    var doPush = true;
    while (i < _paths.length) {
      var _path = _paths[i];
      var _ppath = partPath(_path);

      if (ppath.pack.indexOf(_ppath.pack) == 0) {
        if (ppath.isType) {
          doPush = false;
          i++;
          continue;
        } else if (_ppath.isRec) {
          doPush = false;
          i++;
          continue;
        }
      }
      i++;
    }
    if (doPush) {
      _paths.push(path);
    }
  }
  return _paths;
}
```
