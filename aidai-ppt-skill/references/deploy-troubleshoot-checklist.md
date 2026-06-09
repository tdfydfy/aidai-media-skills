# Deployment Troubleshooting Checklist (1Panel/OpenResty + Cloudflare)

A systematic checklist for when the user says "没有变化" or "页面不存在" after you've deployed an HTML file.

## Phase 1: File Is In The Right Place?

OpenResty runs inside a Docker container on 1Panel servers. The bind mount is:
```
Host: /opt/1panel/www  →  Container: /www
```

The site config says `root /www/sites/<site>/index/`. So the file must be at:

```
/opt/1panel/www/sites/<site>/index/presentation.html
```

**NOT** at `/www/sites/<site>/index/presentation.html` (host-level path that Docker doesn't see).

### How to check

```bash
# 1. Check both paths — if they differ, the Docker path wins
ls -la /opt/1panel/www/sites/<site>/index/
ls -la /www/sites/<site>/index/

# 2. Check if Nginx/OpenResty sees the file (inside the container)
docker exec <container-id> ls -la /www/sites/<site>/index/

# 3. Check file content is correct (has the new code)
docker exec <container-id> head -10 /www/sites/<site>/index/presentation.html

# 4. Check the nginx config to confirm root path
cat /opt/1panel/www/conf.d/<site>.conf
# Look for:  root /www/sites/<site>/index;
#           autoindex on;
#           try_files $uri $uri/ /index.html;
```

### Diagnosis

| Symptom | Likely Cause |
|---|---|
| File exists at `/www/...` but not at `/opt/1panel/www/...` | Copied to wrong path — Docker doesn't see it |
| File at `/opt/1panel/www/...` but curl returns autoindex page | Permission issue (not readable by nginx user) |
| File at both paths, curl returns 200 but wrong content | Cloudflare cache (go to Phase 3) |
| File not found at all | Never copied (missed the cp command) |

## Phase 2: Is Nginx Serving The Correct Content?

```bash
# Test via localhost (bypasses Cloudflare entirely)
curl -s "https://localhost/presentation.html" \
  -H "Host: show.daiworld.cc.cd" 2>/dev/null | head -20

# If localhost SSL fails (cert mismatch), test inside Docker
docker exec <container-id> curl -s http://localhost/presentation.html \
  -H "Host: show.daiworld.cc.cd" | head -20

# Check response code
docker exec <container-id> curl -s -o /dev/null -w "%{http_code}" \
  http://localhost/presentation.html -H "Host: show.daiworld.cc.cd"
# Expected: 301 (HTTP → HTTPS) or 200
```

## Phase 3: Is Cloudflare Caching The Old Version?

The domain `show.daiworld.cc.cd` is behind CF proxy (`CF代理(DYNAMIC)`).

```bash
# Check CF cache status header
curl -sI "https://show.daiworld.cc.cd/presentation.html?v=$(date +%s)" \
  | grep -i "cf-cache-status"
```

| `cf-cache-status` | Meaning | Action |
|---|---|---|
| `DYNAMIC` | Not cached by CF | Browser cache is the problem (user needs hard refresh) |
| `HIT` | Cached by CF | Need to bypass (see below) |
| `MISS` / `REVALIDATED` | Fresh from origin | Server content is correct |

### Bypass methods (in preference order)

1. **New filename** (most reliable) — Create a file with a name CF has never seen:
   ```bash
   SUFFIX=$(date +%s | md5sum | head -c 8)
   cp /opt/1panel/www/sites/show/index/presentation.html \
      "/opt/1panel/www/sites/show/index/slides-${SUFFIX}.html"
   echo "https://show.daiworld.cc.cd/slides-${SUFFIX}.html"
   ```

2. **Cache-busting meta tags** (prevents future caching, doesn't clear existing):
   ```html
   <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
   <meta http-equiv="Pragma" content="no-cache">
   <meta http-equiv="Expires" content="0">
   ```

3. **Query string** (less reliable with CF — depends on cache key config):
   `https://show.daiworld.cc.cd/presentation.html?v=N`

4. **CF API purge** (requires API token, only if available)
5. **Direct IP access** — usually blocked by firewall (only CF IPs allowed on 80/443)

## Phase 4: Tell The User What To Do

If all server-side checks pass but the user still sees old content:

- "请硬刷新 (Ctrl+Shift+R 或 Cmd+Shift+R)"
- "请在无痕/隐私窗口打开这个新URL: https://show.daiworld.cc.cd/slides-xxxx.html"
- "或者清一下浏览器缓存: Chrome地址栏左边🔒 → Site settings → Clear data → 刷新"

## Quick Fire Drill

When user says "没变化":

```bash
# 1. Verify file location
container=$(docker ps | grep openresty | awk '{print $1}')
ls -la /opt/1panel/www/sites/show/index/presentation.html

# 2. Verify file content
docker exec $container grep -c "split-img\|half-img\|deco" /www/sites/show/index/presentation.html

# 3. Check CF cache
curl -sI "https://show.daiworld.cc.cd/presentation.html?v=$(date +%s)" | grep -i "cf-cache-status"

# 4. If all good → new filename
suffix=$(date +%s | md5sum | head -c 8)
cp /opt/1panel/www/sites/show/index/presentation.html \
   "/opt/1panel/www/sites/show/index/alt-${suffix}.html"
echo "Try: https://show.daiworld.cc.cd/alt-${suffix}.html"
```
