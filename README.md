# mac_notes_export_tool
This is a tool helps me to export mac notes to md file.  

1. Create new AppleScript file named `export_notes.scpt`. The code is as follows, I use javascript ;)
```javascript
// set things up 
var app = Application.currentApplication(); 
app.includeStandardAdditions = true; 
var notesApp = Application('Notes'); 
notesApp.includeStandardAdditions = true;
var folders = notesApp.folders
for(var i=0;i<folders.length;i++){
	if(folders[i].name()=='Notes'){
		var targetFolder = folders[i];
		var targetNotes = targetFolder.notes;
		
		for(var j=0;j<targetNotes.length;j++){
		    var idArray = targetNotes[j].id().split("/");
			var id = idArray[idArray.length-1];
			//console.log("id:"+id)
			var filename = '/Users/***/mysoftware/note_backup/data/'+targetNotes[j].name().split(' ')[0].replaceAll('-','')+'_'+id+'.txt';
			var finalContent = targetNotes[j].body().replaceAll("<div><h1>","# ").replaceAll("</h1></div>","\n")
			.replaceAll("<div>","").replaceAll("<br></div>","\n").replaceAll("</div>","\n").replaceAll("<br>","\n")
			var file = app.openForAccess(Path(filename), { writePermission: true });
    		app.setEof(file, { to: 0 });
    		app.write(finalContent, {to: file});
    		app.closeAccess(file);
		}		
	}
}
```

After exporting note to md file the encodeing is gbk, use `iconv` to convert gbk to utf8.

2. Create function in `~/.zshrc`
```bash
nbu()
{
    osascript /Users/***/mysoftware/note_backup/export_notes.scpt
    count=0
    for file in /Users/***/mysoftware/note_backup/data/*
    do
        if [[ $file == *.txt ]]; then
	    echo "convert:$file"
            iconv -f gbk -t utf-8 $file > ${file:0:(${#file}-3)}"md"
            echo "remove:$file"
	    ((count++))
	    rm $file
        fi
    done
    echo "total notes:$count"
    # open /Users/***/mysoftware/note_backup/data
    year=$(date +%Y)
    month=$(date +%m)
    mkdir -p /Users/***/code/wiki/99-日志/"$year/$month"
    # open /Users/***/code/wiki/99-日志/"$year/$month"
    cp /Users/***/mysoftware/note_backup/data/*.md /Users/***/code/wiki/99-日志/"$year/$month"
    find /Users/***/mysoftware/note_backup/data -type f -delete
    cd /Users/***/code/wiki/99-日志/"$year/$month"
}
```

3. use following command to export notes and open target directory.
```bash
nbu
```
