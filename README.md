# DEV_COMMANDS_N_TOOLS
use full dev commands and tools
---

# PRINT WHOLE PROJECT IN MD FILE
## WINDOWS:
```POWERSHELL
# --- Configuration ---
$outputFile = "whole_project_structure.md"
$excludeDirs = @("node_modules", ".git", ".vscode", "dist", "build", "coverage")
$maxDepth = 10 # Prevents infinitely deep loops in case of symlink issues

# --- Script ---
# 1. Start with a clean file and add the Tree Header
Clear-Content $outputFile -ErrorAction SilentlyContinue
Add-Content $outputFile "# Project Structure"
Add-Content $outputFile ""
Add-Content $outputFile "````"

# 2. Generate and Add the Directory Tree
Get-ChildItem -Path . -Recurse -Depth $maxDepth -Exclude $excludeDirs | ForEach-Object {
    $depth = $_.FullName.Split([System.IO.Path]::DirectorySeparatorChar).Count - $PWD.Path.Split([System.IO.Path]::DirectorySeparatorChar).Count
    $indent = "    " * ($depth - 1)
    $prefix = if ($_.PSIsContainer) { "📁" } else { "📄" }
    "$indent$prefix $($_.Name)" | Add-Content -Path $outputFile
}

Add-Content $outputFile "````"
Add-Content $outputFile ""
Add-Content $outputFile "# File Contents"

# 3. Append the contents of each file
$files = Get-ChildItem -Path . -Recurse -File -Exclude $excludeDirs

foreach ($file in $files) {
    $relativePath = $file.FullName.Replace($PWD.Path + "\", "")
    $extension = $file.Extension.TrimStart('.')
    
    if ([string]::IsNullOrEmpty($extension)) {
        $extension = "text"
    }

    Add-Content $outputFile "---"
    Add-Content $outputFile "File: $relativePath"
    Add-Content $outputFile "---"
    Add-Content $outputFile ""
    Add-Content $outputFile "````$extension"
    Add-Content -Path $outputFile -Value (Get-Content -Path $file.FullName -Raw)
    Add-Content $outputFile "````"
    Add-Content $outputFile ""
}

Write-Host "✅ Project with directory tree successfully exported to '$outputFile'"
```

## Linux:

```bash
#!/bin/bash

# --- Configuration ---
OUTPUT_FILE="whole_project_structure.md"
EXCLUDE_DIRS="node_modules|.git|.vscode|dist|build|coverage"

# --- Script ---
# 1. Start with a clean file and add the Tree Header
{
    echo "# Project Structure"
    echo ""
    echo "\`\`\`"
    # 2. Generate and Add the Directory Tree
    tree -aF --prune -I "$EXCLUDE_DIRS"
    echo "\`\`\`"
    echo ""
    echo "# File Contents"
} > "$OUTPUT_FILE"

# 3. Append the contents of each file
find . -path "./$EXCLUDE_DIRS" -prune -o -type f -print | while IFS= read -r file; do
    # This check is important to skip the output file itself
    if [[ "$file" == "./$OUTPUT_FILE" ]]; then
        continue
    fi

    relativePath=$(echo "$file" | sed 's|^\./||')
    extension="${relativePath##*.}"

    if [[ "$relativePath" == "$extension" ]]; then
        extension="text"
    fi
    
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

echo "✅ Project with directory tree successfully exported to '$OUTPUT_FILE'"
```
