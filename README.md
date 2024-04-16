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
var finalNotes=new Map();
for(var i=0;i<folders.length;i++){
	if(folders[i].name()=='Notes'){
		var targetFolder = folders[i];
		var targetNotes = targetFolder.notes;
		
		for(var j=0;j<targetNotes.length;j++){
			var filename = '/Users/***/mysoftware/note_backup/data/'+targetNotes[j].name().split(' ')[0].replaceAll('-','')+'.txt';
			if(finalNotes.has(filename)){
				finalNotes.set(filename,finalNotes.get(filename)+"<br>"+ targetNotes[j].body());
			}else{
				finalNotes.set(filename, targetNotes[j].body())
			}
		}		
	}
}
finalNotes.forEach((content,filename)=>{
	var finalContent = content.replaceAll("<div><h1>","# ").replaceAll("</h1></div>","\n")
	.replaceAll("<div>","").replaceAll("<br></div>","\n").replaceAll("</div>","\n").replaceAll("<br>","\n")
	var file = app.openForAccess(Path(filename), { writePermission: true });
    app.setEof(file, { to: 0 });
    app.write(finalContent, {to: file});
    app.closeAccess(file);
});
```

After exporting note to md file the encodeing is gbk, use `iconv` to convert gbk to utf8.

2. Create function in `~/.zshrc`
```bash
nbu()
{
    osascript /Users/***/mysoftware/note_backup/export_notes.scpt
    for file in /Users/***/mysoftware/note_backup/data/*
    do
        if [[ $file == *.txt ]]; then
            iconv -f gbk -t utf-8 $file > ${file:0:(${#file}-3)}"md"
            rm $file
        fi
    done
    open /Users/***/mysoftware/note_backup/data
}
```

3. use following command to export notes and open target directory.
```bash
nbu
```
