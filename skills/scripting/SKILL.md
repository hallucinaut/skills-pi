---
name: scripting
description: "Write automation scripts, command-line tools, and utility programs. Use when automating tasks, creating CLI tools, or implementing shell/python/node scripts."
---

# Scripting Skill

Write automation scripts and utility programs.

## When to Use

Use this skill when the user wants to:
- Create shell scripts (bash, zsh)
- Write automation utilities
- Build command-line tools
- Implement data processing scripts
- Create configuration scripts
- Set up automation workflows

## Script Types

### Shell Scripts
- **Bash scripts**: Unix/Linux automation
- **Zsh scripts**: macOS automation
- **PowerShell**: Windows automation

### Automation Scripts
- **Data processing**: Extract, transform, load
- **File operations**: Move, copy, organize files
- **Scheduled tasks**: Cron jobs, scheduled tasks
- **System administration**: Backup, cleanup, monitoring

### CLI Tools
- **Argument parsing**: Handle CLI arguments
- **Option parsing**: Flags and options
- **Output formatting**: Console output
- **Error handling**: User-friendly errors

## Shell Script Best Practices

### Shebang
```bash
#!/usr/bin/env bash
#!/bin/zsh
```

### Variable Naming
```bash
# Use descriptive names
user_name="alice"
backup_dir="/tmp/backup"
max_retries=3
```

### Error Handling
```bash
# Check command success
if ! command -v curl &> /dev/null; then
  echo "Error: curl is required"
  exit 1
fi

# Trap errors
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail  # Fail on pipe failure
```

### Input Validation
```bash
# Check arguments
if [[ $# -lt 1 ]]; then
  echo "Usage: $0 <input_file>"
  exit 1
fi

# Validate input
input_file="$1"
if [[ ! -f "$input_file" ]]; then
  echo "Error: File not found: $input_file"
  exit 1
fi
```

### User Interaction
```bash
# Ask for confirmation
read -p "Continue? [y/N] " confirm
if [[ ! "$confirm" =~ ^[Yy]$ ]]; then
  echo "Cancelled"
  exit 0
fi

# Read user input
read -p "Enter your name: " username
```

## Python Scripting

### Input Validation
```python
import argparse

def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('--input', required=True, help='Input file')
    parser.add_argument('--output', help='Output file')
    parser.add_argument('--verbose', action='store_true')
    return parser.parse_args()
```

### Error Handling
```python
try:
    with open('data.json', 'r') as f:
        data = json.load(f)
except FileNotFoundError:
    print("Error: File not found")
    sys.exit(1)
except json.JSONDecodeError as e:
    print(f"Error: Invalid JSON: {e}")
    sys.exit(1)
```

### Script Entry Point
```python
if __name__ == '__main__':
    main()
```

## Node.js Scripting

### CLI Tools with Commander
```javascript
const { Command } = require('commander');
const program = new Command();

program
  .name('my-tool')
  .description('A utility tool')
  .version('1.0.0')
  .option('-v, --verbose', 'Verbose output')
  .parse();

const options = program.opts();
console.log('Verbose:', options.verbose);
```

## Data Processing Scripts

### Reading and Writing
```bash
# Read from file
while IFS= read -r line; do
  echo "$line"
done < input.txt

# Write to file
echo "Line 1" > output.txt
echo "Line 2" >> output.txt
```

### Transformation
```bash
# Process lines
cat input.txt | while read line; do
  # Process each line
  echo "Processed: $line"
done
```

## Automation Workflows

### Scheduled Tasks
```bash
# Cron job
0 2 * * * /path/to/script.sh >> /var/log/script.log 2>&1
```

### Processing Data
```bash
# Process CSV
while IFS=, read -r col1 col2 col3; do
  echo "Processing: $col1, $col2"
done < data.csv
```

### File Operations
```bash
# Copy with progress
cp -v source.txt dest.txt

# Find and process files
find . -type f -name "*.log" -exec ./process.sh {} \;
```

## Deliverables

- Complete scripts with error handling
- Usage documentation
- Examples and usage patterns
- Test scripts
- Configuration files

## Quality Checklist

- Shebang is correct
- Exit codes are proper
- Error messages are clear
- Input validation included
- Comments explain complex logic
- Scripts are executable
- Documentation is clear
- Works on target platforms
