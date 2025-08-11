# DEV_COMMANDS_N_TOOLS
Use full dev commands and tools
---

## PRINT WHOLE PROJECT IN MD FILE
Save the entire structure, including the **Directory Tree** and **File Content**, in a **Markdown** file so you can easily chat with Cloude, Gemini, or ChatGPT without manually pasting files.  
Open Terminal/PowerShell in the location of the project or any desired child directory, copy and paste the whole set of commands/script and press enter.  

### WINDOWS:
```POWERSHELL
# --- Configuration ---
$outputFile = "project-export-guaranteed.md"
$excludeDirs = @("node_modules", ".git", ".vscode", "dist", "build", "coverage")
$maxDepth = 15 

# --- Script ---
# 1. Start with a clean file
Clear-Content $outputFile -ErrorAction SilentlyContinue

# --- Part 1: Generate the Directory Tree (Ultra-Compatible Method) ---

# First, get the complete list of items and store it in a variable.
$allItems = Get-ChildItem -Path . -Recurse -Depth $maxDepth -Exclude $excludeDirs | Where-Object { $_.Name -ne $outputFile }

# Now, process the in-memory list to generate the tree lines.
$treeLines = @()
$treeLines += "# Project Structure"
$treeLines += ""
$treeLines += "```"

foreach ($item in $allItems) {
    $depth = $item.FullName.Split([System.IO.Path]::DirectorySeparatorChar).Count - $PWD.Path.Split([System.IO.Path]::DirectorySeparatorChar).Count
    
    # --- FIX 1: ULTRA-COMPATIBLE INDENTATION USING A FOR LOOP ---
    $indent = ""
    for ($i = 1; $i -lt $depth; $i++) {
        $indent += "    " # Add four spaces for each level
    }
    
    # --- FIX 2: REPLACED EMOJIS WITH PLAIN ASCII CHARACTERS ---
    $prefix = ""
    if ($item.PSIsContainer) {
        $prefix = "[D]" # [D] for Directory
    } else {
        $prefix = "[F]" # [F] for File
    }

    $treeLines += "$indent$prefix $($item.Name)"
}
$treeLines += "```"
$treeLines += ""
$treeLines += "# File Contents"

# Finally, write the complete tree to the file in one operation.
Add-Content -Path $outputFile -Value $treeLines

# --- Part 2: Append File Contents Safely ---

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

Write-Host "✅ Project with directory tree successfully exported to '$outputFile'."
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
