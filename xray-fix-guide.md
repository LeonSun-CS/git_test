# Xray REALITY VPN Client Connection Fix

## Issues Identified

### 1. Missing SNI/Peer (CRITICAL)
The client configuration has an empty `peer` field, which is required for REALITY protocol.

### 2. Missing ALPN
The ALPN field is empty, which should match the destination server's supported protocols.

### 3. Possible Key Mismatch
The public/private key pair needs verification.

## Required Client Configuration Changes

Update your client configuration with these values:

```json
{
  "host": "34.30.104.106",
  "port": "443",
  "type": "VLESS",
  "xtls": 2,
  "uuid": "a453fc6d-74df-4824-98e2-fc6d48abe04f",
  "publicKey": "Go267uWFftMGH3dwaVTgkZcP6Grr7xvRdnmZejDuax0",
  "peer": "www.microsoft.com",          // ← ADD THIS
  "alpn": "h2,http/1.1",                // ← ADD THIS
  "shortId": "",
  "tls": true
}
```

## Key Required Changes:

1. **peer**: Must be set to one of the serverNames from your server config
   - Options: `"microsoft.com"` or `"www.microsoft.com"`
   - Recommended: `"www.microsoft.com"`

2. **alpn**: Set to `"h2,http/1.1"` to match Microsoft's supported protocols

3. **Verify Keys**: Ensure the publicKey on client matches the privateKey on server
   - If unsure, regenerate using: `xray x25519`
   - Use the private key in server config
   - Use the public key in client config

## How to Verify Key Pair

On your server, run:
```bash
xray x25519
```

Output example:
```
Private key: [some-base64-string]
Public key: [some-base64-string]
```

- Copy the **Private key** to server config → `realitySettings.privateKey`
- Copy the **Public key** to client config → `publicKey`

## Testing Steps

1. Update client configuration with peer and alpn values
2. Restart/reconnect the VPN client
3. Test connection to google.com
4. Check server logs: `/var/log/xray/error.log` and `/var/log/xray/access.log`

## Additional Troubleshooting

If still not working after fixes:

1. **Check server logs:**
   ```bash
   tail -f /var/log/xray/error.log
   tail -f /var/log/xray/access.log
   ```

2. **Verify firewall allows port 443:**
   ```bash
   sudo ufw status
   # or
   sudo iptables -L -n
   ```

3. **Test if Xray is listening:**
   ```bash
   sudo netstat -tlnp | grep 443
   # or
   sudo ss -tlnp | grep 443
   ```

4. **Restart Xray service:**
   ```bash
   sudo systemctl restart xray
   sudo systemctl status xray
   ```

## Standard VLESS+REALITY Client Config Template

For reference, a complete client config should look like:

```json
{
  "host": "YOUR_SERVER_IP",
  "port": "443",
  "type": "VLESS",
  "uuid": "YOUR_UUID",
  "flow": "xtls-rprx-vision",
  "tls": true,
  "xtls": 2,
  "peer": "www.microsoft.com",
  "publicKey": "YOUR_PUBLIC_KEY",
  "shortId": "",
  "alpn": "h2,http/1.1"
}
```

## Why These Settings Matter

- **peer (SNI)**: REALITY protocol disguises traffic as HTTPS to the specified domain. The client must tell the server which domain it's pretending to visit.
- **alpn**: Application protocols must match what the real destination supports to avoid detection.
- **publicKey/privateKey**: Encryption keypair - they must match or handshake will fail.
