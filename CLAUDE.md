# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Cloudflare Worker project that implements a VLESS protocol proxy service. The main functionality includes:

1. WebSocket proxy for VLESS protocol traffic
2. Subscription generation for various client formats (VLESS, Clash, Singbox)
3. IP optimization tools for finding best performing Cloudflare IPs
4. Support for various proxy configurations (SOCKS5, HTTP, ProxyIP)
5. Integration with Cloudflare KV for persistent storage
6. Dynamic UUID generation for enhanced security

## Repository Structure

- `_worker.js`: Main entry point containing all logic for the Cloudflare Worker
- `wrangler.toml`: Configuration file for Cloudflare deployment
- `README.md`: Project documentation with deployment instructions

## Key Components

### Core Functionality
1. **WebSocket Handler** (`维列斯OverWSHandler`): Manages VLESS protocol over WebSocket connections
2. **TCP Outbound Handler** (`handleTCPOutBound`): Routes traffic to remote servers with proxy support
3. **UDP/DNS Handler** (`handleUDPOutBound`): Processes DNS queries using DoH to Cloudflare's 1.1.1.1
4. **Subscription Generator** (`生成配置信息`): Creates configuration links in multiple formats
5. **IP Optimizer** (`bestIP`): Web interface for testing and selecting optimal Cloudflare IPs

### Supporting Components
1. **SOCKS5/HTTP Proxy Connectors**: Functions for establishing connections through proxies
2. **Configuration Management**: Environment variable processing and validation
3. **KV Storage Interface**: Integration with Cloudflare KV for persistent data storage
4. **NAT64/DNS64 Support**: IPv6 translation for IPv4-only destinations

## Deployment Methods

### Workers Deployment
1. Create a new Worker in Cloudflare dashboard
2. Paste the content of `_worker.js` into the Worker editor
3. Modify the `userID` variable with your own UUID
4. Deploy the Worker

### Pages Deployment (Recommended)
1. Deploy via Cloudflare Pages by uploading project files
2. Set environment variables in Pages settings:
   - Variable name: `UUID`
   - Value: Your UUID
3. After deployment, access your service

## Essential Environment Variables

### Authentication
- `UUID`: Your unique identifier for the VLESS configuration
- `KEY`: Dynamic UUID secret key (alternative to UUID)
- `TIME`: Dynamic UUID validity period (default: 7 days)
- `UPTIME`: Dynamic UUID update time (default: 3 AM Beijing time)

### Proxy Configuration
- `PROXYIP`: Alternative proxy nodes for accessing CFCDN sites
- `HTTP`: HTTP proxy for accessing CFCDN sites (higher priority than PROXYIP)
- `SOCKS5`: SOCKS5 proxy for accessing CFCDN sites (highest priority)
- `GO2SOCKS5`: Force specific domains to use SOCKS5
- `GO2DNS64`: Route specific domains through DNS64 NAT64 tunnel

### IP and Domain Management
- `ADD`: Local preferred TLS domains/IPs
- `ADDAPI`: API address for preferred IPs
- `ADDCSV`: iptest speed test results
- `DLS`: Speed threshold for ADDCSV results

### Subscription Configuration
- `SUB`: Preferred subscription generator domain
- `SUBAPI`: Subscription conversion backend
- `SUBCONFIG`: Subscription conversion configuration file
- `SUBNAME`: Subscription name

## Core Functionality Patterns

### WebSocket Handling
The VLESS protocol is handled through WebSocket connections. The system processes:
1. VLESS headers for authentication and connection metadata
2. Data piping between WebSocket and remote TCP/UDP connections
3. Retry mechanisms for failed connections

### Subscription Generation
The service generates configurations in multiple formats:
1. Base64 encoded VLESS links
2. Clash configuration format
3. Singbox configuration format
4. Integration with external subscription converters

### Proxy Support
Multiple proxy methods are supported:
1. Direct connections to target servers
2. SOCKS5 proxy with authentication
3. HTTP proxy
4. ProxyIP fallback mechanisms
5. DNS64 domain routing for IPv6 translation

### DNS64/NAT64 Support
The system now includes DNS64 domain routing capabilities:
1. Route specific domains through NAT64 tunnels for IPv6-only networks
2. Configure with `GO2DNS64` environment variable or URL parameters
3. Automatic IPv6 translation for IPv4-only destinations
4. Support for wildcard domain patterns

## Integration with External Services

### Cloudflare KV
- Bind a KV namespace named `KV` for persistent storage
- Access online editors at `[YOUR-URL]/[UUID]/edit`
- Manage preferred IP lists through web interface

### Subscription Conversion Services
- Integrate with external subscription conversion backends
- Configure with `SUBAPI` and `SUBCONFIG` variables
- Support for multiple output formats

### Telegram Notifications
- Set `TGTOKEN` and `TGID` for access notifications
- Receive information about subscription requests

## Common Development Tasks

### Adding New Proxy Methods
1. Implement a new connection function similar to `socks5Connect`
2. Add the method to the connection selection logic in `handleTCPOutBound`
3. Update environment variable processing if needed

### Modifying Subscription Formats
1. Update the `配置信息` function to change VLESS link generation
2. Modify format-specific generation in `生成配置信息`
3. Update the web UI display in the HTML generation section

### Extending IP Optimization
1. Add new IP sources to the `GetCFIPs` function
2. Update the web interface in `bestIP` function
3. Modify testing logic in `testIP` functions

### Adding DNS64 Domain Patterns
1. Update the `useDNS64Pattern` function to modify pattern matching logic
2. Add new domain sources to the DNS64 configuration
3. Modify the NAT64 resolution in `resolveToIPv6` function

## Code Patterns and Conventions

### Function Naming
- Chinese function names are used throughout the codebase
- Functions are typically verb-noun combinations describing their purpose

### Error Handling
- Most functions include try/catch blocks
- Error responses are typically returned as text/plain
- Logging is done through console.log statements

### Asynchronous Operations
- Heavy use of async/await patterns
- Promise-based operations for network requests
- Concurrent processing where appropriate (e.g., IP testing)

When working with this codebase, be mindful of the Chinese variable and function names, which are consistently used throughout the project. The architecture is designed specifically for Cloudflare Workers environment with its limitations and capabilities.