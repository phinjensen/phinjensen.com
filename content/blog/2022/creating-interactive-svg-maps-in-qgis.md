---
title: "Creating interactive SVG maps in QGIS"
author: Phineas Jensen
date: 2022-12-30
description: "An idea for a game—Projections, filters, and calculated columns—Styling—Generating SVGs with (some) metadata"
tags:
  - gis
  - javascript
---

```
array_find(
	array_reverse(
		array_agg($id, order_by := "POP_EST")
	),
	$id
)+1
```

```
array_find(
	array_sort(
		array_agg(
			$area
		),
		false
	),
	$area
)+1
```

Steps to generate map with metadata:
1. Vector->Data Management Tools->Split Vector Layer
  - Export as gpkg to not deal with Shapefile's attribute limitations
  - Save to a folder
2. Layer->Add Layer
  - Select all generated .gpkg files
3. Plugins->Python Console
  - Show Editor, then Open Script
  - update-style.py
4. Layout Manager

```
" record macro q as ndat
/Page<enter>gat
:g/<g .*\/>/d<enter>
:g/<defs\/>/d<enter>
<num of "rect" occurences>@q
```
