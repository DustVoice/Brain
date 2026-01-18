---
{"publish":true,"aliases":"","created":"2026-01-11T16:12:29.543+01:00","modified":"2026-01-16T11:19:14.964+01:00","tags":["categories"],"cssclasses":""}
---


```datacorejsx
const COLUMNS = [
	{ id: "File", value: (row) => row.$link },
	{
		id: "Tags",
		value: (row) => row.$tags,
		render: (tags) => tags?.join(", ")
	},
	{
		id: "Aliases",
		value: (row) => row.value("aliases"),
		render: (tags) => tags?.join(", ")
	}
];

return function View() {
	const pages = dc.useQuery('@page and linksto([[ELI5]]) and !$name.contains("Template")');
	
	return <dc.Table columns={COLUMNS} rows={pages} />;
}
```
