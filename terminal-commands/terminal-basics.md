# Terminal Commands Cheat Sheet (Mac)

## Navigation

### See where you are
```bash
pwd
```

### List files and folders
```bash
ls                    # Basic list
ls -la                # Detailed list with hidden files
ls -lh                # List with human-readable sizes
```

### Change directory
```bash
cd foldername         # Go into folder
cd ..                 # Go up one level
cd ~                  # Go to home directory
cd ~/Desktop          # Go to Desktop
```

---

## File Operations

### Create files
```bash
touch filename.txt    # Create empty file
echo "text" > file.txt    # Create file with content
```

### Create folders
```bash
mkdir foldername      # Create single folder
mkdir -p path/to/folder   # Create nested folders
```

### Copy files/folders
```bash
cp file.txt newfile.txt           # Copy file
cp -r folder/ newfolder/          # Copy folder
```

### Move/Rename
```bash
mv oldname.txt newname.txt        # Rename file
mv file.txt ~/Desktop/            # Move file
```

### Delete (be careful!)
```bash
rm filename.txt       # Delete file
rm -r foldername      # Delete folder and contents
rm -rf foldername     # Force delete (no confirmation)
```

---

## Viewing Files

### Read file contents
```bash
cat filename.txt      # Show entire file
head filename.txt     # Show first 10 lines
tail filename.txt     # Show last 10 lines
less filename.txt     # View file (press q to quit)
```

---

## Search

### Find files
```bash
find . -name "*.txt"              # Find all .txt files
find ~/Desktop -name "project*"   # Find files starting with "project"
```

### Search inside files
```bash
grep "search term" filename.txt   # Search in one file
grep -r "search term" .           # Search in all files
```

---

## System Info

### Check disk space
```bash
df -h
```

### Check folder size
```bash
du -sh foldername
```

### See running processes
```bash
top                   # Interactive view (press q to quit)
ps aux                # List all processes
```

---

## Useful Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Auto-complete file/folder names |
| `Ctrl + C` | Cancel current command |
| `Ctrl + A` | Go to beginning of line |
| `Ctrl + E` | Go to end of line |
| `Ctrl + L` | Clear screen (or type `clear`) |
| `↑` / `↓` | Scroll through previous commands |

---

## Tips

- Use `Tab` to auto-complete - it saves time!
- Be careful with `rm -rf` - deleted files can't be recovered
- `pwd` is your friend when you're lost
- `ls -la` shows hidden files (those starting with `.`)

---

**Created:** November 26, 2024
**Last Updated:** November 26, 2024
