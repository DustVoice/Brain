---
{"publish":true,"aliases":"","created":"2025-09-10T09:52:52.000+02:00","modified":"2026-01-16T10:57:33.127+01:00","tags":["categories"],"cssclasses":""}
---


```datacorejsx
const COLUMNS = [
	{ id: "File", value: (row) => row.$link },
	{ id: "Author", value: (row) => row.value("author") },
	{ id: "Cuisine", value: (row) => row.value("cuisine") },
	{ id: "Type", value: (row) => row.value("type") },
	{ id: "Rating", value: (row) => row.value("rating") }
];

return function View() {
	const pages = dc.useQuery('@page and linksto([[Recipes]]) and !$name.contains("Template")');
	
	return <dc.Table columns={COLUMNS} rows={pages} />;
}
```
