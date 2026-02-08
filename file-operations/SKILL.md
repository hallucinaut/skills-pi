---
name: file-operations
description: "Perform file system operations, manage files and directories, and handle file transfers. Use when working with files, reading/writing content, managing directories, or implementing file processing."
---

# File Operations Skill

Perform file system operations and manage files and directories.

## When to Use

Use this skill when the user wants to:
- Read/write files
- Manage directories and files
- Process file contents
- Handle file uploads/downloads
- Implement file watching
- Work with file metadata
- Process multiple files
- Handle file errors gracefully

## File Reading

### Reading Files
```javascript
// Node.js
const fs = require('fs');
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

```python
# Python
with open('file.txt', 'r') as f:
    data = f.read()
```

### Reading Line by Line
```bash
# Shell
while IFS= read -r line; do
  echo "$line"
done < input.txt
```

```python
# Python
with open('input.txt', 'r') as f:
    for line in f:
        process(line)
```

## File Writing

### Writing Files
```javascript
// Node.js
fs.writeFile('output.txt', 'Hello World', (err) => {
  if (err) throw err;
});
```

```python
# Python
with open('output.txt', 'w') as f:
    f.write('Hello World')
```

### Appending to Files
```javascript
// Node.js
fs.appendFile('log.txt', 'Error: Something went wrong\n', (err) => {
  if (err) throw err;
});
```

```python
# Python
with open('log.txt', 'a') as f:
    f.write('Error: Something went wrong\n')
```

## Directory Operations

### Directory Creation
```bash
# Create directory
mkdir -p project/src

# Create recursive directories
mkdir -p parent/child/grandchild
```

```javascript
// Node.js
fs.mkdirSync('project', { recursive: true });
fs.mkdir('project/src', { recursive: true }, (err) => {
  if (err) throw err;
});
```

### Directory Listing
```bash
# List files
ls -la

# List recursively
find . -type f

# List with details
ls -lh
```

### Directory Removal
```bash
# Remove directory (empty)
rmdir empty_dir

# Remove recursively (force)
rm -rf directory

# Remove non-empty directory
rm -rf directory/
```

## File Metadata

### File Information
```bash
# Get file details
ls -lh file.txt
stat file.txt

# Get file size
wc -l file.txt
du -h file.txt
```

```javascript
// Node.js
fs.stat('file.txt', (err, stats) => {
  console.log(stats.size);      // Size in bytes
  console.log(stats.mtime);     // Last modified time
  console.log(stats.isDirectory()); // Is directory?
});
```

### File Types
```bash
# Check file type
file file.txt

# Check if exists
test -f file.txt
test -d directory
```

## File Processing

### Copy Files
```bash
# Simple copy
cp source.txt dest.txt

# Recursive copy
cp -r source_dir/ dest_dir/

# Preserve attributes
cp -p source.txt dest.txt
```

### Move/Rename Files
```bash
# Rename
mv old.txt new.txt

# Move to directory
mv file.txt directory/

# Move multiple files
mv file1.txt file2.txt destination/
```

### Delete Files
```bash
# Delete file
rm file.txt

# Delete non-empty directory
rm -r directory/

# Force delete (no prompts)
rm -f file.txt

# Delete all in directory
rm -rf directory/
```

### Find and Process Files
```bash
# Find files by name
find . -name "*.txt"

# Find files by pattern
find . -name "*.log*"

# Find by modification time
find . -mtime -7
```

### Batch Operations
```bash
# Process all files in directory
for file in *.txt; do
  process "$file"
done

# Find and process files
find . -type f -name "*.json" -exec ./process.sh {} \;
```

## File Watching

### Watch for Changes
```javascript
// Node.js fs.watch
fs.watch('file.txt', (eventType, filename) => {
  if (eventType === 'change') {
    console.log('File changed:', filename);
  }
});
```

### Directory Watching
```javascript
// Node.js recursive watch
const watcher = fs.watch('directory', { recursive: true }, (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});
```

## File Transfer

### Upload Files
```javascript
// Node.js file upload
const FormData = require('form-data');
const form = new FormData();
form.append('file', fs.createReadStream('document.pdf'));

axios.post('/upload', form, {
  headers: form.getHeaders()
});
```

### Download Files
```javascript
// Node.js download
fs.download('https://example.com/file.zip', 'output.zip', (err) => {
  if (err) throw err;
});
```

## Error Handling

### File Not Found
```javascript
// Node.js
fs.readFile('missing.txt', (err, data) => {
  if (err) {
    if (err.code === 'ENOENT') {
      console.log('File not found');
    } else {
      console.error('Error reading file:', err);
    }
  }
});
```

### Permission Errors
```javascript
// Check permissions
try {
  fs.accessSync('protected.txt', fs.constants.R_OK);
  console.log('Read permission granted');
} catch (err) {
  console.log('No permission:', err.code);
}
```

## File Compression

### Zip Files
```bash
# Create zip
zip archive.zip file1.txt file2.txt

# Add files to zip
zip archive.zip file3.txt

# Recursively zip directory
zip -r archive.zip directory/
```

### Extract Archives
```bash
# Extract zip
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d destination/

# Extract without preserving permissions
unzip -a archive.zip
```

## Deliverables

- Complete file operation implementation
- Error handling strategies
- File processing utilities
- Documentation
- Examples

## Quality Checklist

- File operations are atomic
- Error handling is comprehensive
- Permissions are checked
- Race conditions are prevented
- Sensitive file operations are secured
- File paths are validated
- Large files are handled properly
- Temporary files are cleaned up
