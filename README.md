# SSH Port Forward GitHub Action

A comprehensive GitHub Action for establishing SSH port forwarding tunnels with support for both local (-L) and remote (-R) forwarding, multiple simultaneous tunnels, jump hosts, and cross-platform compatibility (Windows, Linux, macOS).

## Features

- ✅ **Local and Remote Port Forwarding**: Support for both -L and -R SSH flags
- ✅ **Multiple Simultaneous Tunnels**: Configure multiple forwards in a single connection
- ✅ **Cross-platform support**: Windows, Linux, and macOS
- ✅ **Multiple authentication methods**: Private key, private key file, or password
- ✅ **Jump host support**: Chain multiple SSH connections
- ✅ **Custom SSH flags**: Add any additional SSH parameters
- ✅ **Known hosts management**: Specify trusted hosts
- ✅ **Automatic cleanup**: Terminates SSH tunnels in post-action
- ✅ **Connection testing**: Verifies tunnel establishment
- ✅ **Configurable timeouts**: Set connection and keep-alive intervals
- ✅ **Flexible port specification**: Support for host:port:host:port format

## Usage

### Local Port Forwarding (-L)

Forward local ports to remote destinations through SSH tunnel:

```yaml
- name: Setup Local Port Forwards
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    local_forwards: "3306:db.internal:3306,5432:localhost:5432"
```

### Remote Port Forwarding (-R)

Forward remote ports back to local or other destinations:

```yaml
- name: Setup Remote Port Forwards
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    remote_forwards: "8080:localhost:3000,9090:service.local:8080"
```

### Combined Local and Remote Forwarding

Use both local and remote forwarding in the same tunnel:

```yaml
- name: Setup Bidirectional Port Forwards
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    local_forwards: "3306:db.internal:3306"
    remote_forwards: "8080:localhost:3000"
```

### Advanced Format with Specific Bind Addresses

Specify bind addresses for more control:

```yaml
- name: Setup Advanced Port Forwards
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    local_forwards: "127.0.0.1:3306:db.internal:3306,0.0.0.0:5432:postgres.internal:5432"
    remote_forwards: "0.0.0.0:8080:localhost:3000"
```

### With Jump Host

```yaml
- name: Setup SSH Tunnel through Jump Host
  uses: your-org/ssh-port-forward@v1
  with:
    host: internal-server.local
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    jump_hosts: jumpuser@jump.example.com:22
    local_forwards: "5432:localhost:5432"
```

### Multiple Jump Hosts with Multiple Forwards

```yaml
- name: Setup Complex SSH Tunnels
  uses: your-org/ssh-port-forward@v1
  with:
    host: final-destination.local
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    jump_hosts: jump1@first.example.com:22,jump2@second.example.com:2222
    local_forwards: "8080:web.internal:80,3306:db.internal:3306"
    remote_forwards: "9000:localhost:9000"
```

### Database and Web Service Access

```yaml
- name: Setup Multi-Service Access
  uses: your-org/ssh-port-forward@v1
  with:
    host: bastion.example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    local_forwards: |
      3306:mysql.internal:3306,
      5432:postgres.internal:5432,
      6379:redis.internal:6379,
      8080:api.internal:80
```

### With Custom SSH Options

```yaml
- name: Setup SSH Tunnel with Custom Options
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    local_forwards: "3306:localhost:3306"
    ssh_flags: -o StrictHostKeyChecking=yes -o PreferredAuthentications=publickey
    timeout: 60
    keep_alive: 30
```

### Real-World Development Scenario

```yaml
- name: Setup Development Environment Access
  uses: your-org/ssh-port-forward@v1
  with:
    host: dev-bastion.company.com
    username: developer
    private_key: ${{ secrets.DEV_SSH_KEY }}
    jump_hosts: gateway@vpn.company.com:22
    local_forwards: |
      3306:mysql-dev.internal:3306,
      5432:postgres-dev.internal:5432,
      6379:redis-dev.internal:6379,
      8080:api-dev.internal:8080,
      9200:elasticsearch-dev.internal:9200
    remote_forwards: "3000:localhost:3000"
    
- name: Run Database Tests
  run: |
    # MySQL available on localhost:3306
    mysql -h localhost -P 3306 -u testuser -p < test.sql
    
    # PostgreSQL available on localhost:5432
    psql -h localhost -p 5432 -U testuser -d testdb
    
    # API available on localhost:8080
    curl http://localhost:8080/health
    
    # Local dev server accessible from remote on port 3000
    npm start # Server runs on port 3000, accessible via remote forward
```

## Port Forward Format

### Local Forwards (-L)

Local forwards use the format: `[bind_address:]port:host:hostport`

- `3306:localhost:3306` - Forward local port 3306 to remote localhost:3306
- `127.0.0.1:3306:db.internal:3306` - Bind to specific local address
- `0.0.0.0:8080:web.internal:80` - Bind to all interfaces (allow external access)
- `5432:postgres.company.internal:5432` - Forward to different remote host

### Remote Forwards (-R)

Remote forwards use the format: `[bind_address:]port:host:hostport`

- `8080:localhost:3000` - Remote port 8080 forwards to local port 3000
- `0.0.0.0:9000:localhost:9000` - Allow external access to remote forward
- `127.0.0.1:5432:db.local:5432` - Bind remote forward to specific address

### Multiple Forwards

Separate multiple forwards with commas:

```yaml
local_forwards: "3306:db1:3306,5432:db2:5432,6379:redis:6379"
remote_forwards: "8080:localhost:3000,9090:localhost:8080"
```

You can also use YAML multiline format for better readability:

```yaml
local_forwards: |
  3306:mysql.internal:3306,
  5432:postgres.internal:5432,
  6379:redis.internal:6379
```

### Using Password Authentication (Not Recommended)

```yaml
- name: Setup SSH Tunnel with Password
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    password: ${{ secrets.SSH_PASSWORD }}
    local_forwards: "3306:localhost:3306"
```

### Using Private Key File Path

```yaml
- name: Setup SSH Key File
  run: |
    echo "${{ secrets.SSH_PRIVATE_KEY }}" > /tmp/ssh_key
    chmod 600 /tmp/ssh_key

- name: Setup SSH Tunnel
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key_path: /tmp/ssh_key
    local_forwards: "3306:localhost:3306"
```

### With Known Hosts

```yaml
- name: Setup SSH Tunnel with Known Hosts
  uses: your-org/ssh-port-forward@v1
  with:
    host: example.com
    username: myuser
    private_key: ${{ secrets.SSH_PRIVATE_KEY }}
    known_hosts: |
      example.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...
      bastion.example.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5...
    local_port: 3306
    remote_port: 3306
```

## Complete Workflow Example

```yaml
name: Deploy with Database Access

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup SSH Tunnel to Production Database
      uses: your-org/ssh-port-forward@v1
      with:
        host: db-bastion.example.com
        username: deploy-user
        private_key: ${{ secrets.SSH_PRIVATE_KEY }}
        jump_hosts: jump@gateway.example.com:22
        local_forwards: "5432:prod-db.internal:5432"
        ssh_flags: -o StrictHostKeyChecking=yes
        timeout: 30
        keep_alive: 60
    
    - name: Run Database Migration
      env:
        DATABASE_URL: postgres://user:pass@localhost:5432/mydb
      run: |
        npm install
        npm run migrate
    
    - name: Deploy Application
      run: |
        # Application deployment commands
        echo "Deploying application..."
    
    # SSH tunnel is automatically cleaned up in post-action
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `host` | Target SSH host | ✅ | |
| `port` | SSH port | | `22` |
| `username` | SSH username | ✅ | |
| `private_key` | SSH private key content | | |
| `private_key_path` | Path to SSH private key file | | |
| `password` | SSH password (not recommended) | | |
| `known_hosts` | SSH known_hosts content | | |
| `local_forwards` | Local port forwards (-L). Format: "[host:]port:host:port" | | |
| `remote_forwards` | Remote port forwards (-R). Format: "[host:]port:host:port" | | |
| `jump_hosts` | Comma-separated jump hosts (user@host:port) | | |
| `ssh_flags` | Additional SSH flags | | `-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null` |
| `timeout` | Connection timeout in seconds | | `30` |
| `keep_alive` | Keep-alive interval in seconds | | `60` |

**Note**: Either `local_forwards` or `remote_forwards` (or both) must be specified.

## Outputs

| Output | Description |
|--------|-------------|
| `tunnel_pid` | Process ID of the SSH tunnel |
| `local_forwards` | Active local port forwards |
| `remote_forwards` | Active remote port forwards |

## Authentication Methods

### 1. Private Key (Recommended)

Store your private key in GitHub Secrets and reference it:

```yaml
with:
  private_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

### 2. Private Key File

If you have the private key in a file:

```yaml
with:
  private_key_path: /path/to/private/key
```

### 3. Password (Not Recommended)

For password authentication (less secure):

```yaml
with:
  password: ${{ secrets.SSH_PASSWORD }}
```

## Jump Hosts

Specify jump hosts in the format `user@host:port`, separated by commas:

```yaml
with:
  jump_hosts: jump1@gateway.com:22,jump2@internal.com:2222
```

## Security Best Practices

1. Use private key authentication instead of passwords
2. Store sensitive data in GitHub Secrets
3. Use `StrictHostKeyChecking=yes` in production
4. Provide known_hosts when possible
5. Use minimal SSH flags for your use case
6. Set appropriate timeouts

## Troubleshooting

### Connection Issues

- Verify SSH credentials and host accessibility
- Check if jump hosts are correctly specified
- Ensure private key format is correct (OpenSSH format)
- Verify firewall rules allow the connection

### Port Already in Use

If a local port is already in use, choose a different port:

```yaml
with:
  local_forwards: "13306:db.internal:3306"  # Instead of 3306:db.internal:3306
```

### Permission Denied

Ensure the SSH user has appropriate permissions:
- User exists on the target host
- Private key matches the public key in `~/.ssh/authorized_keys`
- SSH service is running on the target host

### Windows-Specific Issues

On Windows runners, ensure:
- Private key has correct line endings (LF, not CRLF)
- Path separators use forward slashes or escaped backslashes

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test across different platforms
5. Submit a pull request

## License

This action is available under the MIT License.