# Pesach 2026 Shopping Checklist — Claude Context

## Live URL
https://alastairlivingston-sudo.github.io/pesach-2026/

## What this is
Shared real-time shopping checklist for Pesach 2026.
Two users (Alastair + Elly) tick items on their phones; changes sync within 5s via polling.
Single HTML file, no build step, GitHub Pages hosting.

## Architecture
- Single file: index.html (all HTML, CSS, JS)
- Sync: JSONBin.io REST API
- Hosting: GitHub Pages, main branch root

## Sync mechanism
- Load: GET https://api.jsonbin.io/v3/b/69c04ecbaa77b81da90b9120/latest
- Tick/untick: PUT same URL with {checked:{...}}
- Poll every 5s (skipped if save in-flight via isSaving flag)
- Status: green dot = OK, red = failed, timestamp = last sync

## Credentials (in index.html)
- JSONBin Master Key: $2a$10$/PVj5wfp1Tea7CnmdaeteO/srG2mzYsP9Gdy6mkqpG4H78XMyfb6.
- JSONBin Bin ID: 69c04ecbaa77b81da90b9120
- GitHub PAT: stored in the git remote URL of the local repo (passwordless push)

## Local repo path
C:\Users\alastair.livingston\OneDrive\Alastair's Documents\Claude\pesach-2026\

This IS the working copy. Edit index.html here directly, then push.

## Deploy workflow
```bash
cd "C:/Users/alastair.livingston/OneDrive/Alastair's Documents/Claude/pesach-2026"
git add index.html
git commit -m "describe change"
git push
```
No token needed — already embedded in the remote URL.

## Data model

### ITEMS array
{ id, n (name), d (detail), store (kk|tesco|ms|butcher), t (tags) }
Tags: parve, kitniot, meat, dairy, kp, warn

### MEALS array
{ day, dayLabel, slot ("Breakfast - ...", "Lunch - ...", "Dinner - ..."), who, items[] }
Day keys: fri01, wed01, fri03, sat04, sun05, mon06, tue07, wed08, thu09
Who values: All, Alastair, Elly, Kids, Alastair & kids, Elly & kids

### STORE_GROUPS array
{ store, label, groups[{ name, ids[] }] }

### checked object (synced state)
{ itemId: true|false, ... } stored in JSONBin as { checked: {...} }

## Views
1. By Store: store > aisle > items. Store filter pills in header.
2. By Meal: day > meal slot > items with store badges. Day filter pills in header.
3. Meal Plan: day cards with Breakfast/Lunch/Dinner rows per person.
   Tap any meal -> bottom sheet slides up showing that meals shopping items.

## Design system
- Font: -apple-system (SF Pro on iPhone)
- Background: #f2f2f7 (iOS grey)
- Cards: #ffffff
- Header: frosted glass backdrop-filter blur(20px), sticky, includes segmented control + filter tabs
- Checkboxes: circular, #34c759 fill when checked
- Store colours: KK #ff3b30, Tesco #007aff, M&S #34c759, Butcher #ff9500
- Safe areas: env(safe-area-inset-top/bottom) in header + FAB row + body padding

## Key JS functions
load(cb)            - GET bin, populate checked, call cb
save()              - PUT checked to bin
poll()              - GET bin, applyCheckedToDOM if changed
startPolling()      - setInterval 5000ms -> poll()
toggleItem(id)      - flip checked[id], save(), update DOM
applyCheckedToDOM() - sync all .item elements to checked state
renderStoreView()   - build By Store DOM
renderMealView()    - build By Meal DOM
renderMealPlanView()- build Meal Plan DOM
showMealDetail(meal)- open bottom sheet with meal ingredients
closeSheet()        - close bottom sheet
switchView(v)       - toggle store | meal | plan
resetAll()          - clear all checked (with confirm)
doneAll()           - mark visible items done
