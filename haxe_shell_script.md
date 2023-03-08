`ts=$(date +%s);echo "function main(){Sys.print('$ts');sys.FileSystem.deleteFile('Main_$ts.hx');}" > Main_$ts.hx | haxe --run Main_$ts`

OR : 
"hs.sh" file :
```
ts=$(date +%s)
echo "function main(){$1}" > Main_$ts.hx
haxe --run Main_$ts 
rm Main_$ts.hx
```
Usage :
`./hs.sh "Sys.print( 'Hello World' );"`
