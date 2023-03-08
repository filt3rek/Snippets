ts=$(date +%s);echo "function main(){Sys.print('$ts');sys.FileSystem.deleteFile('Main_$ts.hx');}" > Main_$ts.hx | haxe --run Main_$ts
