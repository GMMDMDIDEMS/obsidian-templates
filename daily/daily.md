---
tag-links:
  - "[[<% tp.file.creation_date('YYYYMMDD') %>]]"
  - "[[daily]]"
---
<%*
const dailyNoteFolder = "06 - Daily";

// get date from file title (see 'Daily Notes' Core Plugin Setting)
const date = moment(tp.file.title)

// function to get the latest file in a folder 
async function getLatestFile(folderPath) {
	const files = app.vault.getFiles().filter(file => file.path.startsWith(folderPath));
	// rollover only works if there is more than one file
	if (files.length <= 1) return null;
	
	// sort files by their creation date in descending order
	const sortedFiles = files.sort((a, b) => b.stat.ctime - a.stat.ctime);
	return sortedFiles[1]; // Return the latest file
}

// extract rollover elements (tasks and action plan)
let rolloverTasks = "";
let actionPlan = "";

const latestFile = await getLatestFile(`${dailyNoteFolder}/`);
if (latestFile) { 
  const latestFileContent = await app.vault.adapter.read(latestFile.path);
  const lines = latestFileContent.split('\n');
  
  let inActionPlanTomorrow = false;
  
  for (let line of lines) {
    // "Action Plan for Tomorrow" defined?
	if (line.match(/^## Action Plan for Tomorrow/)) {
		inActionPlanTomorrow = true;
	    continue;
	}
	// start of new section/heading?
	if (line.match(/^## /) && inActionPlanTomorrow) {
		inActionPlanTomorrow = false;
	}
	// match 'action plan' lines starting with bullet points ("-" or "- [ ]")
	if (inActionPlanTomorrow && line.match(/- (?:\[ \] |\s*(?!\[ \]))\s*\S/)) {
		if (line.match(/^- \[ \]/)) {
			actionPlan += `${line.trim()}\n`;
		} else {
			// replace "-" (dash) with task bullet point ("- [ ]")
			actionPlan += `${line.replace(/^- /, "- [ ] ")}\n`;
		}
	}
	// match any line defining a task ("- [ ]"), not within the "Action Plan for Tomorrow" section
	if (!inActionPlanTomorrow && line.match(/^\s*[-*] \[ \]\s*\S+/)) {
		rolloverTasks += `${line.trim()}\n`;
	}
  }
} else {
	console.log("Rollover not possible: No other daily file could be found. Please ensure there are previous daily notes available in the expected folder.");
}
%>
# Daily - <%date.format("dddd, Do MMMM YYYY")%>
---

## Action Plan
---
<%* tR += actionPlan %>- [ ] 

## Tasks
---
<%* tR += rolloverTasks %>- [ ] 

## Notes
---
- 

## Action Plan for Tomorrow
___
- 
