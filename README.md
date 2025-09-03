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
4loop [-z|-c] 'command' input1 [input2] [timeout]
```

### Parameters

- `command`: The command to execute
  - `LINE`, `LINE1`, `LINE2` require actual data (file or INPUT)
  - With single input: all three variables map to the same value
  - With two inputs: `LINE1` = first file, `LINE2` = second file, `LINE` = first file
  - **Important**: Numbers alone cannot be used with LINE variables
- `input1`: Primary input source
  - A filename containing lines to iterate over (works with LINE)
  - An integer for simple repetition (only works WITHOUT LINE)
  - `INPUT` to read from stdin (works with LINE)
- `input2` (optional): Secondary input source for two-file operations
- `timeout` (optional): Delay between iterations (default: 0.075 seconds)
  - Use suffixes for clarity: `3s` (seconds), `500ms` (milliseconds)
  - Without suffix: decimals treated as timeout, whole numbers may be ambiguous

### Iteration Modes

- **Default (Cartesian Product)**: Every line from input1 with every line from input2
- **`-z` (Zip Mode)**: Pair lines 1:1, stops at shortest file
- **`-c` (Cycle Mode)**: Repeat shorter file to match longer file's length

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

### Important: LINE Variables Require Data

**LINE, LINE1, and LINE2 only work with actual data sources** (files or INPUT):

```bash
# ✗ WRONG - Will error
4loop 'echo LINE' 5              # Error: LINE needs data
4loop 'echo LINE1:LINE2' 3 3     # Error: LINE needs data

# ✓ CORRECT - Provide actual data
seq 1 5 | 4loop 'echo LINE' INPUT       # Uses piped data
4loop 'echo LINE' file.txt              # Uses file data
4loop 'echo hello' 5                    # No LINE, just repeat
```

### Timeout Specification

Use suffixes for clear timeouts:

```bash
# Explicit timeouts with suffixes
4loop 'echo hello' 3 3s          # 3 iterations, 3 second delay
4loop 'curl api.test' 5 500ms    # 5 iterations, 500ms delay

# Without LINE, "3 3" means 3×3 iterations
4loop 'echo hello' 3 3            # 9 iterations (3×3)
4loop 'echo hello' 3 3s           # 3 iterations with 3s delay
```

### Repeat Command N Times
Execute a command a specific number of times (no LINE substitution):

```bash
4loop 'echo hello world' 5
# Output:
# hello world
# hello world  
# hello world
# hello world
# hello world

# Note: Using LINE with a number will error:
# 4loop 'echo LINE' 5  ← ERROR: LINE needs data
# Use this instead:
seq 1 5 | 4loop 'echo LINE' INPUT  # Outputs: 1, 2, 3, 4, 5
```

### Two-File Operations

#### Cartesian Product (Default)
Test all combinations:

```bash
# hosts.txt: host1, host2
# ports.txt: 80, 443
4loop 'nc -zv LINE1 LINE2' hosts.txt ports.txt
# Tests: host1:80, host1:443, host2:80, host2:443
```

#### Zip Mode (-z)
Pair corresponding lines:

```bash
# users.txt: alice, bob, charlie
# passwords.txt: pass1, pass2, pass3
4loop -z 'adduser LINE1 --password LINE2' users.txt passwords.txt
# Pairs: alice:pass1, bob:pass2, charlie:pass3
```

#### Cycle Mode (-c)
Repeat shorter file:

```bash
# files.txt: file1, file2, file3, file4, file5
# servers.txt: server1, server2
4loop -c 'scp LINE1 LINE2:backup/' files.txt servers.txt
# Maps: file1→server1, file2→server2, file3→server1, file4→server2, file5→server1
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

## Timeout Handling & Disambiguation

Timeouts use suffixes (`s`, `ms`) to avoid ambiguity with numeric inputs:

| Command | Result | Why |
|---------|--------|-----|
| `4loop 'echo hello' 3 3` | Repeats 9 times (3×3) | No LINE = numeric iteration |
| `4loop 'echo hello' 3 3s` | Repeats 3 times, 3s delay | Explicit timeout with `s` |
| `4loop 'echo hello' 3 0.5` | Repeats 3 times, 0.5s delay | Decimal = timeout |
| `4loop 'echo LINE' 3` | **ERROR** | LINE requires data, not numbers |
| `seq 1 3 \| 4loop 'echo LINE' INPUT` | Works: outputs 1,2,3 | INPUT provides data |
| `4loop 'echo LINE' file.txt 2s` | Works: reads file, 2s delay | File provides data |

## Advanced Examples

### Security Testing

#### Password Spraying (Cartesian)
```bash
# Test one password against all users on all servers
4loop 'crackmapexec smb LINE2 -u LINE1 -p Winter2025!' users.txt targets.txt
```

#### Credential Testing (Zip)
```bash
# Test specific username:password pairs
4loop -z 'hydra -l LINE1 -p LINE2 ssh://target.com' users.txt passwords.txt
```

#### Distributed Scanning (Cycle)
```bash
# Distribute scans across multiple jump hosts
4loop -c 'ssh LINE2 "nmap -sV LINE1"' targets.txt jumphosts.txt
```

### Web Application Testing

#### Subdomain Enumeration
```bash
# Check all subdomain+domain combinations
4loop 'dig +short LINE1.LINE2' subdomains.txt domains.txt
```

#### API Fuzzing with Proxies
```bash
# Rotate through proxies while fuzzing
4loop -c 'curl -x LINE2 "api.target.com/LINE1"' endpoints.txt proxies.txt
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

### Complex Attack Chains
```bash
# Stage 1: Find valid accounts (cartesian)
4loop 'kerbrute userenum -d LINE2 LINE1' users.txt domains.txt > valid.txt

# Stage 2: Test with rotating proxies (cycle)
4loop -c 'proxychains -q -f LINE2 smbclient -L //target/share -U LINE1%Pass123' valid.txt proxies.txt

# Stage 3: Exploit specific pairs (zip)
4loop -z 'impacket-psexec LINE1:LINE2@target.com' compromised_users.txt passwords.txt
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