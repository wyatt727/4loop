# 4loop

A versatile command-line utility for iterating commands over file contents or a specified number of times.

## Description

4loop executes a given command multiple times, replacing the placeholder `LINE` with each line from an input file, or simply repeating the command a specified number of times. It's particularly useful for batch operations, network scanning, and automating repetitive tasks.

## Installation

```bash
git clone https://github.com/wyatt727/4loop.git
cd 4loop
chmod +x 4loop
```

To make it available system-wide:
```bash
sudo cp 4loop /usr/local/bin/
```

## Usage

```bash
4loop 'command' input [timeout]
```

### Parameters

- `command`: The command to execute (use `LINE` as a placeholder for substitution)
- `input`: Can be:
  - A filename containing lines to iterate over
  - An integer specifying the number of iterations
  - `INPUT` to read from stdin
- `timeout` (optional): Delay between iterations in seconds (default: 0.075)

## Examples

### Basic File Iteration
Iterate over lines in a file, replacing `LINE` with each line's content:

```bash
# If test.txt contains numbers 1-5 (one per line)
4loop 'echo LINE' test.txt
# Output:
# 1
# 2
# 3
# 4
# 5
```

### Repeat Command N Times
Execute a command a specific number of times:

```bash
4loop 'echo hello world' 5
# Output:
# hello world
# hello world
# hello world
# hello world
# hello world
```

### Using Line Count from File
Execute command as many times as there are lines in the file (without using the content):

```bash
# If test.txt has 5 lines
4loop 'echo foo' test.txt
# Output:
# foo
# foo
# foo
# foo
# foo
```

### Piped Input
Read input from stdin:

```bash
seq 1 5 | 4loop 'echo -LINE-' INPUT
# Output:
# -1-
# -2-
# -3-
# -4-
# -5-
```

### Custom Timeout
Add a delay between iterations (in seconds):

```bash
# No delay between commands
4loop 'echo LINE' file.txt 0

# 1 second delay between commands
4loop 'ping -c 1 LINE' hosts.txt 1
```

## Real-World Use Cases

### Network Scanning
Check NFS exports on multiple hosts:

```bash
# If port-2049.txt contains IP addresses with open NFS ports
4loop 'showmount -e LINE' port-2049.txt
```

### Web Reconnaissance
Check HTTP headers for multiple domains:

```bash
4loop 'curl -I LINE' domains.txt 0.5
```

### File Operations
Create multiple directories:

```bash
echo -e "project1\nproject2\nproject3" | 4loop 'mkdir LINE' INPUT
```

### Port Scanning
Quick connectivity check:

```bash
4loop 'nc -zv LINE 80' ip-list.txt 0.1
```

### Mass File Processing
Convert multiple images:

```bash
ls *.png | 4loop 'convert LINE LINE.jpg' INPUT
```

## Features

- Simple and intuitive syntax
- Flexible input methods (file, number, or stdin)
- Configurable delay between iterations
- LINE placeholder for dynamic command construction
- Lightweight with minimal dependencies

## Requirements

- Bash shell
- Standard Unix utilities (cat, wc, head, tail)

## License

This project is open source and available under the MIT License.

## Author

Created for simplifying repetitive command-line tasks and batch operations.