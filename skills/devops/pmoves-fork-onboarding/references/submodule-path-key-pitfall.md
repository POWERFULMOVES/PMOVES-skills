# Submodule path= key pitfall (git 2.52.0.windows.1)

## Symptom
`git submodule status <path>` → `fatal: no submodule mapping found in .gitmodules for path '<path>'`
`git submodule init` → `fatal: No url found for submodule path '<path>' in .gitmodules`

...while BOTH of these succeed:
- `git config --file .gitmodules --get submodule.<name>.url` → returns the URL
- `git config --blob HEAD:.gitmodules --get submodule.<name>.url` → returns the URL

## Root cause
The submodule machinery resolves gitlink path → section via the `submodule.<name>.path` VALUE, and this build does not fall back to section-name == path. Sections written with only `url`/`branch`/`shallow` are invisible to it. Every pre-existing PMOVES.AI section carries an explicit `path = <dir>` line — that's why the old ones work and hand-added ones don't.

## Fix
Add `path = <dirname>` as the first key in the section:

```
[submodule "PMOVES-composio"]
	path = PMOVES-composio
	url = https://github.com/POWERFULMOVES/PMOVES-composio.git
	branch = PMOVES.AI-Edition-Hardened
	shallow = true
```

Then `git add .gitmodules` and re-run `git submodule status` — resolves immediately.

## Red herrings encountered (do NOT chase)
- CRLF/LF mixed line endings in .gitmodules — normalizing changes nothing
- Stale `.git/config` submodule sections — removing them changes nothing
- `git submodule--helper list/name` subcommands — removed in modern git

## Diagnosis one-liner
```python
import re
raw = open('.gitmodules').read()
for s in re.split(r'\[submodule "', raw)[1:]:
    name = s.split('"]')[0]
    if 'path =' not in s: print('MISSING path:', name)
```
