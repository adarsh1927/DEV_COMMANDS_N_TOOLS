# DEV_COMMANDS_N_TOOLS
Use full dev commands and tools
---

## PRINT WHOLE PROJECT IN MD FILE
Save the entire structure, including the **Directory Tree** and **File Content**, in a **Markdown** file so you can easily chat with Cloude, Gemini, or ChatGPT without manually pasting files.  
Open Terminal/PowerShell in the location of the project or any desired child directory, copy and paste the whole set of commands/script and press enter.  

### WINDOWS:
```POWERSHELL
# --- Configuration ---
$outputFile = "project-export-with-tree-command.md"
$excludeDirs = @("node_modules", ".git", ".vscode", "dist", "build", "coverage")

# --- Script ---
# Start with a clean file
Clear-Content $outputFile -ErrorAction SilentlyContinue

# --- Part 1: Generate the Tree using the 'tree' command ---
$treeContent = @()
$treeContent += "# Project Structure"
$treeContent += ""
# --- FIX: Changed double quotes to single quotes ---
$treeContent += '```' 
# Use /F to include files and /A to use ASCII characters for better file compatibility.
# Note: This will show ALL files and folders, as tree.com cannot exclude.
$treeContent += (tree /F /A)
# --- FIX: Changed double quotes to single quotes ---
$treeContent += '```'
$treeContent += ""
$treeContent += "# File Contents"

Add-Content -Path $outputFile -Value $treeContent

# --- Part 2: Append File Contents (this part was already correct) ---
$filesToProcess = Get-ChildItem -Path . -Recurse -File -Exclude $excludeDirs | Where-Object { $_.Name -ne $outputFile }

foreach ($file in $filesToProcess) {
    $relativePath = $file.FullName.Replace($PWD.Path + "\", "")
    $extension = $file.Extension.TrimStart('.')
    
    if ([string]::IsNullOrEmpty($extension)) {
        $extension = "text"
    }
    
    $fileContent = @()
    $fileContent += "---"
    $fileContent += "File: $relativePath"
    $fileContent += "---"
    $fileContent += ""
    $fileContent += ('```' + $extension)
    $fileContent += (Get-Content -Path $file.FullName -Raw)
    $fileContent += '```'
    $fileContent += ""
    
    Add-Content -Path $outputFile -Value $fileContent
}

Write-Host "✅ Project successfully exported to '$outputFile' using the 'tree /F' command."
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
