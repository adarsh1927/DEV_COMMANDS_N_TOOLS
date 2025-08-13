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

# FIX 1: Define excludes as a proper Bash array. This is cleaner and more reliable.
EXCLUDE_ARRAY=( 
    ".gitignore" 
    "metadata.json" 
    "README.md" 
    "node_modules" 
    ".git" 
    ".vscode" 
    "dist" 
    "build" 
    "coverage"
    "$OUTPUT_FILE" # Also exclude the output file itself
)

# --- Dynamically build the exclusion patterns for each command ---

# FIX 2: Build the pipe-separated pattern for the 'tree' command
TREE_EXCLUDE_PATTERN=""
for item in "${EXCLUDE_ARRAY[@]}"; do
    TREE_EXCLUDE_PATTERN+="$item|"
done
# Remove the trailing pipe character
TREE_EXCLUDE_PATTERN=${TREE_EXCLUDE_PATTERN%|}

# FIX 3: Build the '-path ... -o ...' arguments for the 'find' command
FIND_EXCLUDE_ARGS=()
for item in "${EXCLUDE_ARRAY[@]}"; do
    FIND_EXCLUDE_ARGS+=(-o -path "./$item")
done
# The first '-o' is not needed, so we remove it from the array
FIND_EXCLUDE_ARGS=("${FIND_EXCLUDE_ARGS[@]:1}")

# --- Script ---
# 1. Start with a clean file and add the Tree Header
{
    echo "# Project Structure"
    echo ""
    echo "\`\`\`"
    # Use the correctly formatted pattern for 'tree'
    tree -aF --prune -I "$TREE_EXCLUDE_PATTERN"
    echo "\`\`\`"
    echo ""
    echo "# File Contents"
} > "$OUTPUT_FILE"

# 2. Append the contents of each file
# Use the correctly formatted arguments for 'find'
find . \( "${FIND_EXCLUDE_ARGS[@]}" \) -prune -o -type f -print | while IFS= read -r file; do
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

## SSH PERMISSION ON WINDOWS

**Please follow these steps carefully.**

1.  **Open PowerShell as Administrator:**
    *   Click the Start Menu.
    *   Type `PowerShell`.
    *   **Right-click** on "Windows PowerShell" and select **"Run as administrator"**. You must run it as an administrator to change ownership and permissions correctly.

2.  **Run the Following Commands One by One:**
    *   Copy each command block below, paste it into the PowerShell window, and press Enter. Wait for it to complete before moving to the next one.
    *   These commands will navigate to your user profile, take ownership of the `.ssh` folder, and then lock it down so only your user can access it.

    **Command 1: Go to your user profile directory**
    ```powershell
    cd C:\Users\your_user_name
    ```

    **Command 2: Take ownership of the `.ssh` folder and everything inside it.**
    ```powershell
    icacls .ssh /T /Q /C /setowner "your_user_name"
    ```

    **Command 3: Reset all permissions on the folder.**
    ```powershell
    icacls .ssh /T /Q /C /reset
    ```

    **Command 4: Grant your user (`your_user_name`) full control.** (This is the most important step).
    ```powershell
    icacls .ssh /T /Q /C /grant your_user_name:F
    ```

    **Command 5: Remove permissions for everyone else (inheritance).**
    ```powershell
    icacls .ssh /T /Q /C /inheritance:r
    ```

3.  **Verify the Permissions (Optional but Recommended):**
    *   Run this final command to see who has access to the `config` file.
    ```powershell
    icacls C:\Users\your_user_name\.ssh\config
    ```
    *   The output should show a line like `your_user_name:(F)` or `LAPTOP\your_user_name:(F)` and **no other users or groups** (like `Administrators` or `SYSTEM`). If you see other users, something went wrong, but these commands are usually very effective.
