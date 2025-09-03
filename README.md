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
  - Use `LINE`, `LINE1`, or `LINE2` as placeholders for substitution
  - With single input: all three map to the same value
  - With two inputs: `LINE1` = first file, `LINE2` = second file, `LINE` = first file
- `input1`: Primary input source
  - A filename containing lines to iterate over
  - An integer specifying the number of iterations
  - `INPUT` to read from stdin
- `input2` (optional): Secondary input source for two-file operations
- `timeout` (optional): Delay between iterations (default: 0.075 seconds)
  - Supports suffixes: `3s` (seconds), `500ms` (milliseconds)
  - Without suffix: decimal numbers are timeouts, integers may be inputs

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

### Unambiguous Timeout Specification
Use time suffixes to clearly specify timeouts:

```bash
# 3 iterations with 3-second delay (explicit)
4loop 'echo hello' 3 3s

# 5 iterations with 500ms delay
4loop 'ping -c 1 target.com' 5 500ms

# Without suffix, "3 3" would be two inputs!
4loop 'echo LINE' 3 3     # Outputs: 1,2,3 repeated 3 times
4loop 'echo LINE' 3 3s    # Outputs: 1,2,3 with 3s delays
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

## Timeout Handling

The tool intelligently handles timeouts to avoid ambiguity:

| Command | Interpretation |
|---------|----------------|
| `4loop 'echo LINE' 3 3` | Two numeric inputs (3×3 cartesian) |
| `4loop 'echo LINE' 3 3s` | 3 iterations, 3 second timeout |
| `4loop 'echo LINE' 3 0.5` | 3 iterations, 0.5s timeout (decimal = timeout) |
| `4loop 'echo LINE1:LINE2' 3 3` | Two inputs (command uses LINE2) |
| `seq 5 \| 4loop 'echo LINE' INPUT 2s` | Piped input, 2 second timeout |

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