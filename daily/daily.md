---
tag-links:
  - "[[<% tp.file.creation_date('YYYYMMDD') %>]]"
  - "[[daily]]"
---
<%*
// get date from file title (see 'Daily Notes' Core Plugin Setting)
const date = moment(tp.file.title)

// manipulate yesterday date to build correct file path
const yesterday = tp.date.yesterday('YYYY-MM-DD');
const yesterdayStr = moment(yesterday);
const yesterdayYear = yesterdayStr.format('YYYY');
const yesterdayMonth = yesterdayStr.format('MMMM');

// file path of yesterday's daily
const dailyNoteFolder = "06 - Daily";
const yesterdayPath = `${dailyNoteFolder}/${yesterdayYear}/${yesterdayMonth}/${yesterday}.md`;

// extract rollover elements (tasks and action plan)
let rolloverTasks = "";
let actionPlan = "";
if (await app.vault.adapter.exists(yesterdayPath)) {
  const yesterdayFile = await app.vault.adapter.read(yesterdayPath);
  const lines = yesterdayFile.split('\n');
  
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
	if (inActionPlanTomorrow && line.match(/^\s*- (\[ \]|)/)) {
		if (line.match(/^\s*- \[ \]/)) {
			actionPlan += `${line.trim()}\n`;
		} else {
			// replace "-" (dash) with task bullet point ("- [ ]")
			actionPlan += `${line.trim().replace(/^\s*- /, "- [ ] ")}\n`;
		}
	}
	// match any line defining a task ("- [ ]"), not within the "Action Plan for Tomorrow" section
	if (!inActionPlanTomorrow && line.match(/^\s*[-*] \[ \]/)) {
		rolloverTasks += `${line.trim()}\n`;
	}
  }
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