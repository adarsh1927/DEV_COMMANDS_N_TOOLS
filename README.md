# DEV_COMMANDS_N_TOOLS
Use full dev commands and tools
---

## PRINT WHOLE PROJECT IN MD FILE
Save the entire structure, including the **Directory Tree** and **File Content**, in a **Markdown** file so you can easily chat with Cloude, Gemini, or ChatGPT without manually pasting files.  
Open Terminal/PowerShell in the location of the project or any desired child directory, copy and paste the whole set of commands/script and press enter.  

### WINDOWS:
1. Save this file in Notepad (do not use any fancy editor, use a raw text editor like Notepad).
2. Copy and paste the script below in Notepad and save it as export.ps1 in your project directory or any desired child directory.
3. Open PowerShell
4. Run command `cd your_project_or_desired_directory` in PowerShell
5. Again, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`
6. Then run `.\export.ps1`
You will get the file whole_project_structure.ps1, change its extension to md or just change the line of script below $outputFile = 'whole_project_structure.md'.

```POWERSHELL (paste this in notepad)
# --- Configuration ---
$outputFile = 'whole_project_structure.ps1'
$excludeDirs = @('node_modules', '.git', '.vscode', 'dist', 'build', 'coverage')
$maxDepth = 15

# --- Script ---
# Start with a clean file
Clear-Content $outputFile -ErrorAction SilentlyContinue

# Get all items first to avoid file lock issues
$allItems = Get-ChildItem -Path . -Recurse -Depth $maxDepth -Exclude $excludeDirs | Where-Object { $_.Name -ne $outputFile }

# Prepare an array to hold all the output lines
$outputLines = @()
$outputLines += '# Project Structure'
$outputLines += ''
$outputLines += '```'

# Build the tree structure
foreach ($item in $allItems) {
    $depth = $item.FullName.Split([System.IO.Path]::DirectorySeparatorChar).Count - $PWD.Path.Split([System.IO.Path]::DirectorySeparatorChar).Count
    
    $indentation = ''
    if ($depth -gt 1) {
        for ($i = 1; $i -lt $depth; $i++) {
            $indentation += '    '
        }
    }
    
    $prefix = ''
    if ($item.PSIsContainer) {
        $prefix = '[Dir]  '
    } else {
        $prefix = '[File] '
    }

    $outputLines += ($indentation + $prefix + $item.Name)
}

$outputLines += '```'
$outputLines += ''
$outputLines += '# File Contents'

# Get all files to process
$filesToProcess = Get-ChildItem -Path . -Recurse -File -Exclude $excludeDirs | Where-Object { $_.Name -ne $outputFile }

# Build the file content sections
foreach ($file in $filesToProcess) {
    $relativePath = $file.FullName.Replace($PWD.Path + '\', '')
    $extension = $file.Extension.TrimStart('.')
    
    if ([string]::IsNullOrEmpty($extension)) {
        $extension = 'text'
    }
    
    $outputLines += '---'
    $outputLines += ('File: ' + $relativePath)
    $outputLines += '---'
    $outputLines += ''
    $outputLines += ('```' + $extension)
    $outputLines += (Get-Content -Path $file.FullName -Raw)
    $outputLines += '```'
    $outputLines += ''
}

# Write all collected lines to the file at once
Add-Content -Path $outputFile -Value $outputLines

Write-Host 'SUCCESS: Project exported to' $outputFile
```

### Linux:

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
