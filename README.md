# DEV_COMMANDS_N_TOOLS
use full dev commands and tools
---

# PRINT WHOLE PROJECT IN MD FILE
## WINDOWS:
```POWERSHELL
# --- Configuration ---
$outputFile = "rhysley-bot-complete-project.md"
$excludeDirs = @("node_modules", ".git", ".vscode", "dist", "build")

# --- Script ---
Clear-Content $outputFile -ErrorAction SilentlyContinue
$files = Get-ChildItem -Path . -Recurse -Exclude $excludeDirs | Where-Object { !$_.PSIsContainer }

foreach ($file in $files) {
    $relativePath = $file.FullName.Replace($PWD.Path + "\", "")
    $extension = $file.Extension.TrimStart('.')
    
    # Use a generic language tag if extension is missing (like for Dockerfile)
    if ([string]::IsNullOrEmpty($extension)) {
        $extension = "text"
    }

    Add-Content $outputFile "---"
    Add-Content $outputFile "File: $relativePath"
    Add-Content $outputFile "---"
    Add-Content $outputFile ""
    Add-Content $outputFile "``````$extension"
    Add-Content -Path $outputFile -Value (Get-Content -Path $file.FullName -Raw)
    Add-Content $outputFile "``````"
    Add-Content $outputFile ""
}

Write-Host "✅ Project successfully exported to '$outputFile'"
```

## Linux:

```bash
#!/bin/bash

# --- Configuration ---
OUTPUT_FILE="rhysley-bot-complete-project.md"
EXCLUDE_DIRS=("./node_modules" "./.git" "./.vscode" "./dist" "./build" "./.gitignore")

# --- Build the find command's exclusion parameters ---
prune_paths=()
for dir in "${EXCLUDE_DIRS[@]}"; do
    prune_paths+=(-o -path "$dir")
done

# --- Script ---
# Start with a clean file
> "$OUTPUT_FILE"

# Find all files, excluding specified directories
find . -type f \( "${prune_paths[@]:1}" \) -prune -o -print | while IFS= read -r file; do
    # Get the relative path and extension
    relativePath=$(echo "$file" | sed 's|^\./||')
    extension="${relativePath##*.}"

    # Use a generic "text" for files without an extension (like Dockerfile)
    if [[ "$relativePath" == "$extension" ]]; then
        extension="text"
    fi
    
    # Append the formatted content to the output file
    {
        echo "---"
        echo "File: $relativePath"
        echo "---"
        echo ""
        echo "\`\`\`$extension"
        cat "$file"
        echo ""
        echo "\`\`\`"
        echo ""
    } >> "$OUTPUT_FILE"
done

echo "✅ Project successfully exported to '$OUTPUT_FILE'"
```
