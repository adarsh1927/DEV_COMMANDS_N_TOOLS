# DEV_COMMANDS_N_TOOLS
Use full dev commands and tools
---

## PRINT WHOLE PROJECT IN MD FILE
Save the entire structure, including the **Directory Tree** and **File Content**, in a **Markdown** file so you can easily chat with Cloude, Gemini, or ChatGPT without manually pasting files.  
Open Terminal/PowerShell in the location of the project or any desired child directory, copy and paste the whole set of commands/script and press enter.  

### WINDOWS:
#### GIT Bash
Git Bash is an excellent tool for running Linux commands on Windows.  
You just need to put tree.exe inside C:\Program Files\Git\usr\bin.  
Download from here **[GnuWin32 Tree for Windows](https://sourceforge.net/projects/gnuwin32/files/tree/1.5.2.2/tree-1.5.2.2-bin.zip/download)**.  
Inside the ZIP, navigate into the **`bin`** folder and copy tree.exe and paste it in C:\Program Files\Git\usr\bin    
Close and reopen GIT Bash if it is open currently.  
Open git bash, paste this directly into it:-
```bash
#!/bin/bash

# --- Configuration ---
OUTPUT_FILE="whole_project_structure.md"

EXCLUDE_ARRAY=( ".gitignore" "metadata.json" "README.md" "node_modules" ".git" ".vscode" "dist" "build" "coverage" "$OUTPUT_FILE" "export.md" ".secondry_readme" )
# --- FIX: Add a list of binary extensions to ignore ---
BINARY_EXTENSIONS=( "png" "jpg" "jpeg" "gif" "ico" "svg" "woff" "woff2" "ttf" "eot" "otf" "pdf" "zip" "exe" "dll" "so" "a" "lib" )

# --- Dynamically build exclusion patterns ---
TREE_EXCLUDE_PATTERN=""
for item in "${EXCLUDE_ARRAY[@]}"; do TREE_EXCLUDE_PATTERN+="$item|"; done
TREE_EXCLUDE_PATTERN=${TREE_EXCLUDE_PATTERN%|}

FIND_EXCLUDE_ARGS=()
for item in "${EXCLUDE_ARRAY[@]}"; do FIND_EXCLUDE_ARGS+=(-o -path "./$item"); done
FIND_EXCLUDE_ARGS=("${FIND_EXCLUDE_ARGS[@]:1}")

# --- FIX: Build the exclusion pattern for binary files for 'find' ---
BINARY_EXCLUDE_ARGS=()
for ext in "${BINARY_EXTENSIONS[@]}"; do
    BINARY_EXCLUDE_ARGS+=(-o -iname "*.$ext")
done
# The first '-o' is not needed
BINARY_EXCLUDE_ARGS=("${BINARY_EXCLUDE_ARGS[@]:2}")

# --- Script ---
{
    echo "# Project Structure"
    echo ""
    echo "\`\`\`"
    tree -aF -I "$TREE_EXCLUDE_PATTERN"
    echo "\`\`\`"
    echo ""
    echo "# File Contents"

    # --- FIX: Modified 'find' to also exclude binary files ---
    find . \( "${FIND_EXCLUDE_ARGS[@]}" \) -prune -o -type f -not \( "${BINARY_EXCLUDE_ARGS[@]}" \) -print | while IFS= read -r file; do
        relativePath=$(echo "$file" | sed 's|^\./||')
        extension="${relativePath##*.}"
        if [[ "$relativePath" == "$extension" ]]; then extension="text"; fi
        
        echo "---"
        echo "File: $relativePath"
        echo "---"
        echo ""
        echo "\`\`\`$extension"
        cat "$file"
        echo ""
        echo "\`\`\`"
        echo ""
    done
} | iconv -f "$(locale charmap)" -t "UTF-8" > "$OUTPUT_FILE"

echo "✅ Project successfully exported to '$OUTPUT_FILE' (binary files ignored)."
```
#### PowerShell
1. Save this file in Notepad (do not use any fancy editor, use a raw text editor like Notepad).
2. Copy and paste the script below in Notepad and save it as export.ps1 in your project directory or any desired child directory.
3. Open PowerShell
4. Run command `cd your_project_or_desired_directory` in PowerShell
5. Again, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`
6. Then run `.\export.ps1`
You will get the file whole_project_structure.ps1, change its extension to md or just change the line of script below $outputFile = 'whole_project_structure.md'.

```POWERSHELL (paste this in notepad)
# --- Configuration ---
$outputFile = 'whole_project_structure.md'
$excludeDirs = @('node_modules', '.git', '.vscode', 'dist', 'build', 'coverage')
$maxDepth = 15

# --- Script ---
# Start with a clean file
Clear-Content $outputFile -ErrorAction SilentlyContinue

# --- Part 1: Get and Filter All Items (The Robust Way) ---

# First, get ALL items recursively. We will filter them manually.
$allPotentialItems = Get-ChildItem -Path . -Recurse -Depth $maxDepth | Where-Object { $_.Name -ne $outputFile }

# Now, use a powerful Where-Object filter on the FULL PATH of each item.
$finalFilteredItems = $allPotentialItems | Where-Object {
    $item = $_
    # The '$isMatch' variable will track if the current item's path matches any exclude pattern.
    $isMatch = $false
    foreach ($excludeDir in $excludeDirs) {
        # Check if the full path contains the directory name surrounded by backslashes.
        # This is the most reliable way to check for a directory in a path.
        if ($item.FullName -like "*\$excludeDir\*") {
            $isMatch = $true
            break # Found a match, no need to check other directories.
        }
    }
    # This is the core of the logic: only keep items where a match was NOT found.
    -not $isMatch
}

# --- Part 2: Build the Tree from the CLEAN list ---

$treeLines = @()
$treeLines += '# Project Structure'
$treeLines += ''
$treeLines += '```'

foreach ($item in $finalFilteredItems) {
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

    $treeLines += ($indentation + $prefix + $item.Name)
}

$treeLines += '```'
$treeLines += ''
$treeLines += '# File Contents'

Add-Content -Path $outputFile -Value $treeLines

# --- Part 3: Append File Contents from the CLEAN list ---

# We filter again for files only, but from our already-clean list.
$filesToProcess = $finalFilteredItems | Where-Object { -not $_.PSIsContainer }

foreach ($file in $filesToProcess) {
    $relativePath = $file.FullName.Replace($PWD.Path + '\', '')
    $extension = $file.Extension.TrimStart('.')
    
    if ([string]::IsNullOrEmpty($extension)) {
        $extension = 'text'
    }
    
    $fileContent = @()
    $fileContent += '---'
    $fileContent += ('File: ' + $relativePath)
    $fileContent += '---'
    $fileContent += ''
    $fileContent += ('```' + $extension)
    $fileContent += (Get-Content -Path $file.FullName -Raw)
    $fileContent += '```'
    $fileContent += ''
    
    Add-Content -Path $outputFile -Value $fileContent
}

Write-Host "SUCCESS: Project exported to '$outputFile'."
```

### Linux:

```bash
#!/bin/bash

# --- Configuration ---
OUTPUT_FILE="whole_project_structure.md"

EXCLUDE_ARRAY=( 
    ".gitignore" "metadata.json" "README.md" "node_modules" 
    ".git" ".vscode" "dist" "build" "coverage" "$OUTPUT_FILE" 
    "export.md" ".secondry_readme"
)

BINARY_EXTENSIONS=( 
    "png" "jpg" "jpeg" "gif" "ico" "svg" "webp" "woff" "woff2" 
    "ttf" "eot" "otf" "pdf" "zip" "gz" "tar" "rar" "exe" 
    "dll" "so" "a" "lib" "jar" "mp3" "mp4" "avi" "mov" "webm"
)

# --- Dynamically build exclusion patterns ---
TREE_EXCLUDE_PATTERN=""
for item in "${EXCLUDE_ARRAY[@]}"; do TREE_EXCLUDE_PATTERN+="$item|"; done
TREE_EXCLUDE_PATTERN=${TREE_EXCLUDE_PATTERN%|}

FIND_EXCLUDE_ARGS=()
for item in "${EXCLUDE_ARRAY[@]}"; do FIND_EXCLUDE_ARGS+=(-o -path "./$item"); done
FIND_EXCLUDE_ARGS=("${FIND_EXCLUDE_ARGS[@]:1}")

BINARY_EXCLUDE_ARGS=()
for ext in "${BINARY_EXTENSIONS[@]}"; do BINARY_EXCLUDE_ARGS+=(-o -iname "*.$ext"); done
BINARY_EXCLUDE_ARGS=("${BINARY_EXCLUDE_ARGS[@]:1}")

# --- Script ---

# 1. Create the file header and the clean directory tree
{
    echo "# Project Structure"
    echo ""
    echo "\`\`\`"
    tree -aF --prune -I "$TREE_EXCLUDE_PATTERN"
    echo "\`\`\`"
    echo ""
    echo "# File Contents"
} > "$OUTPUT_FILE"

# 2. Find all non-binary, non-excluded files and append their content
find . \( "${FIND_EXCLUDE_ARGS[@]}" \) -prune -o -type f -not \( "${BINARY_EXCLUDE_ARGS[@]}" \) -print | while IFS= read -r file; do
    relativePath=$(echo "$file" | sed 's|^\./||')
    extension="${relativePath##*.}"

    if [[ "$relativePath" == "$extension" ]]; then
        extension="text"
    fi
    
    # Append (>>) the content for each file to the existing file
    {
        echo "---"
        echo "File: $relativePath"
        echo "---"
        echo ""
        echo "\`\`\`$extension"
        # --- THE CRITICAL FIX ---
        # Use 'sed' to remove the Windows carriage return ('\r') from each line.
        sed 's/\r$//' "$file"
        # --- END OF FIX ---
        echo ""
        echo "\`\`\`"
        echo ""
    } >> "$OUTPUT_FILE"
done

echo "✅ Project successfully exported to '$OUTPUT_FILE' (line endings normalized)."
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
