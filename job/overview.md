---
tag-links:
- "[[jobtracking]]"
---

## Open Tasks
---
```tasks
not done
path includes 08 - Jobs
```


## Job Portals
---

```dataviewjs
// offset in milliseconds
var tzoffset = (new Date()).getTimezoneOffset() * 60000;
var localISOTime = (new Date(Date.now() - tzoffset)).toISOString().slice(0, -5).replace("T", " ");
console.log(localISOTime);

const now = moment()
console.log(now.toISOString())

dv.table(["Company", "Last Checked", "Update"], dv.pages('"08 - Jobs"')
	.where(p => dv.func.contains(p["tag-links"], dv.func.link("jobportal")))
	.map(p => [
		p.file.link,
		moment(p.file.mtime.toISO()).from(now),
		// localISOTime - p.file.mtime.toISO().slice(0, -10).replace("T", " "),
		dv.el("span", "Update", {
		    cls: "edit-button",
		    onclick: async () => {
				console.log(p.file.mtime.toISO().slice(0, -10).replace("T", " "))
				const file = await app.vault.getAbstractFileByPath(p.file.path); 
				if (file) {
					const content = await app.vault.read(file);
					const updatedContent = content.replace(/modified: .*/, `modified: ${localISOTime}`);
					await app.vault.modify(file, updatedContent);
				}
			}
		})
	])
);
```


## Open Applications
---

```dataview
TABLE WITHOUT ID
rows[0].company AS Company,
date[0] AS "Days since applying",
rows[0].rows.job-title AS Job
FROM "08 - Jobs"
WHERE company
GROUP BY company
GROUP BY map(rows, (r) => floor((date(now) - r.file.ctime).days)) AS date
```


*Job listing*:

```dataview
TABLE floor((date(now) - file.ctime).days) AS "Days since applying"
FROM "08 - Jobs"
WHERE application-status = [[applied]]
SORT file.ctime DESC
```
