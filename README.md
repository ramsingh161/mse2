# MSE-II Docker & Docker Compose — Set A

Use this guide in the terminal inside the lab machine. Replace `<ROLL_NO>` and `<YOUR_NAME>` with your own values; do not leave placeholders in the files.

## 1. Set the values

For Linux, macOS, WSL, or Git Bash:

```bash
read -p "Enter your university roll number: " ROLL_NO
read -p "Enter your name: " YOUR_NAME
export ROLL_NO
LAST_TWO=${ROLL_NO: -2}
HOST_PORT=$((8000 + 10#$LAST_TWO))
export HOST_PORT
LAB_DIR="mse2_${ROLL_NO}"
mkdir -p "$LAB_DIR"
cd "$LAB_DIR"
printf 'ROLL_NO=%s\nHOST_PORT=%s\n' "$ROLL_NO" "$HOST_PORT" > .env
printf 'LAB folder: %s\nRoll number: %s\nHost port: %s\n' "$LAB_DIR" "$ROLL_NO" "$HOST_PORT"
```

For PowerShell, run instead:

```powershell
$ROLL_NO = Read-Host "Enter your university roll number"
$YOUR_NAME = Read-Host "Enter your name"
$env:ROLL_NO = $ROLL_NO
$LAST_TWO = $ROLL_NO.Substring($ROLL_NO.Length - 2)
$HOST_PORT = 8000 + [int]$LAST_TWO
$LAB_DIR = "mse2_$ROLL_NO"
New-Item -ItemType Directory -Path $LAB_DIR -Force | Out-Null
Set-Location $LAB_DIR
"ROLL_NO=$ROLL_NO`nHOST_PORT=$HOST_PORT" | Set-Content -Encoding ascii .env
"LAB folder: $LAB_DIR`nRoll number: $ROLL_NO`nHost port: $HOST_PORT"
```

Check that `$ROLL_NO` ends in `07` gives port `8007`, and a roll number ending in `45` gives port `8045`.

## 2. Create the four starter files

From Linux, macOS, WSL, or Git Bash:

```bash
cat > .env <<EOF
ROLL_NO=$ROLL_NO
HOST_PORT=$HOST_PORT
EOF

cat > index.html <<EOF
<html>
<body>
<h1>MSE-II Docker Lab</h1>
<p>Name: $YOUR_NAME</p>
<p>Roll No: $ROLL_NO</p>
</body>
</html>
EOF

cat > Dockerfile <<'EOF'
FROM nginx:alpne
COPY index.htm /usr/share/nginx/html/index.html
EXPOSE 80
EOF

cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    port:
      - "${HOST_PORT}:8080"
EOF
```

For PowerShell:

```powershell
"ROLL_NO=$ROLL_NO`nHOST_PORT=$HOST_PORT" | Set-Content -Encoding ascii .env

@"
<html>
<body>
<h1>MSE-II Docker Lab</h1>
<p>Name: $YOUR_NAME</p>
<p>Roll No: $ROLL_NO</p>
</body>
</html>
"@ | Set-Content -Encoding ascii index.html

@"
FROM nginx:alpne
COPY index.htm /usr/share/nginx/html/index.html
EXPOSE 80
"@ | Set-Content -Encoding ascii Dockerfile

@"
services:
  web:
    build: .
    image: web-`${ROLL_NO}:v1
    container_name: web-`${ROLL_NO}
    port:
      - "`${HOST_PORT}:8080"
"@ | Set-Content -Encoding ascii docker-compose.yml
```

Set your terminal prompt so it shows the roll number and time.

Linux, macOS, WSL, or Git Bash:

```bash
export PS1="[$ROLL_NO \t] \w \$ "
```

PowerShell:

```powershell
function prompt { "[$env:ROLL_NO $(Get-Date -Format HH:mm:ss)] $(Get-Location)> " }
```

Take screenshot **S0**:

```bash
ls -a
```

PowerShell:

```powershell
dir -Force
```

Verify that S0 shows the prompt, command, and `.env`, `index.html`, `Dockerfile`, and `docker-compose.yml`.

## 3. Q1 — find and fix all four mistakes

Start with:

```bash
docker compose up -d --build
```

If a curl command runs just after a container starts, wait one second and run it again.

### Q1(a) — compose-file spelling

Docker should name an unexpected word in `docker-compose.yml`. Compare it with:

`services`, `build`, `image`, `container_name`, `ports`, `restart`

Take screenshot **E1** of the error.

The mistake is `port`; change it to `ports`. A bash command can make this first correction:

```bash
sed -i 's/^[[:space:]]*port:/    ports:/' docker-compose.yml
cat docker-compose.yml
docker compose up -d --build
```

On macOS, if `sed -i` reports an error, use VS Code or `sed -i '' 's/^port:/ports:/' docker-compose.yml`.

PowerShell:

```powershell
(Get-Content docker-compose.yml) -replace '^\s*port:', '    ports:' | Set-Content docker-compose.yml
Get-Content docker-compose.yml
docker compose up -d --build
```

Take screenshot **E3** of `cat docker-compose.yml` and the command result. In PowerShell, use `Get-Content` instead of `cat`.

Write this sentence in your own PDF:

> I changed port to ports because Docker Compose needs the correct ports key.

### Q1(b) — Dockerfile word spelling

The build stops and marks a Dockerfile line containing `>>>`. The word after `FROM` is misspelt: `alpne`.

Take screenshot **E1** of this build error.

Fix `alpne` to `alpine`:

```bash
sed -i 's/nginx:alpne/nginx:alpine/' Dockerfile
cat Dockerfile
docker compose up -d --build
```

PowerShell:

```powershell
(Get-Content Dockerfile) -replace 'nginx:alpne', 'nginx:alpine' | Set-Content Dockerfile
Get-Content Dockerfile
docker compose up -d --build
```

Take screenshot **E3** of the corrected Dockerfile and the working command.

Write:

> I changed alpne to alpine because alpine is the misspelt nginx image tag.

### Q1(c) — wrong web-page file name

Docker says a file was not found. Check the real file name:

```bash
ls
```

PowerShell:

```powershell
dir
```

The file is `index.html`, but the Dockerfile currently says `index.htm`.

Take screenshot **E1** of the not-found error and the `ls`/`dir` output.

Fix the COPY source:

```bash
sed -i 's/COPY index.htm /COPY index.html /' Dockerfile
cat Dockerfile
docker compose up -d --build
```

PowerShell:

```powershell
(Get-Content Dockerfile) -replace 'COPY index\.htm ', 'COPY index.html ' | Set-Content Dockerfile
Get-Content Dockerfile
docker compose up -d --build
```

Take screenshot **E3** of the corrected Dockerfile and the working build.

Write:

> I changed index.htm to index.html because the source file exists under the second name.

### Q1(d) — wrong container port

Test the page:

```bash
curl "http://localhost:$HOST_PORT"
```

PowerShell:

```powershell
curl.exe "http://localhost:$HOST_PORT"
docker compose ps
```

Read the `PORTS` column:

```bash
docker compose ps
```

The compose port currently maps port `8080` inside the container, but nginx listens on port `80`.

Take screenshot **E1** of the curl error and the wrong `PORTS` mapping.

Fix `8080` to `80`:

```bash
sed -i 's/:8080"/:80"/' docker-compose.yml
cat docker-compose.yml
docker compose up -d --build
sleep 1
curl "http://localhost:$HOST_PORT"
```

PowerShell:

```powershell
(Get-Content docker-compose.yml) -replace ':8080"', ':80"' | Set-Content docker-compose.yml
Get-Content docker-compose.yml
docker compose up -d --build
Start-Sleep -Seconds 1
curl.exe "http://localhost:$HOST_PORT"
```

The curl output must show your name and roll number. Take screenshot **E3** of the corrected file and successful curl.

Write:

> I changed 8080 to 80 because nginx listens on container port 80, not 8080.

## 4. Q2 — start a second container by hand

Confirm the compose image exists:

```bash
docker images
```

The image name should be `web-${ROLL_NO}:v1`.

Calculate the second port:

```bash
SECOND_PORT=$((HOST_PORT + 1))
export SECOND_PORT
echo "$SECOND_PORT"
```

PowerShell:

```powershell
$SECOND_PORT = $HOST_PORT + 1
echo $SECOND_PORT
```

Start the second container without Compose:

```bash
docker run -d --name "second-$ROLL_NO" -p "$SECOND_PORT:80" "web-$ROLL_NO:v1"
```

PowerShell:

```powershell
docker run -d --name "second-$ROLL_NO" -p "${SECOND_PORT}:80" "web-$ROLL_NO:v1"
```

Evidence: screenshot the `docker run` command and Docker's container ID output.

Wait briefly, then curl:

```bash
sleep 1
curl "http://localhost:$SECOND_PORT"
```

PowerShell:

```powershell
Start-Sleep -Seconds 1
curl.exe "http://localhost:$SECOND_PORT"
```

Screenshot the page output.

Show both containers:

```bash
docker ps
```

The output must show both `web-$ROLL_NO` and `second-$ROLL_NO` with port mappings. Take the third Q2 screenshot.

Remove the second container:

```bash
docker rm -f "second-$ROLL_NO"
```

PowerShell:

```powershell
docker rm -f "second-$ROLL_NO"
```

## 5. Q3(i) — edit the page without rebuilding

Create `site` and copy the page into it:

```bash
mkdir -p site
cp index.html site/index.html
printf '\n<p>Edited without rebuild</p>\n' >> site/index.html
cat site/index.html
```

PowerShell:

```powershell
New-Item -ItemType Directory -Path site -Force | Out-Null
Copy-Item index.html site/index.html
Add-Content site/index.html "`n<p>Edited without rebuild</p>"
Get-Content site/index.html
```

Add a volumes entry under `web` and keep the existing compose entries. The final compose file should look like this:

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
```

Replace the compose file with this block now.

Linux, macOS, WSL, or Git Bash:

```bash
cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
EOF

cat docker-compose.yml
docker compose up -d
sleep 1
curl "http://localhost:$HOST_PORT"
```

PowerShell:

```powershell
@'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
'@ | Set-Content -Encoding ascii docker-compose.yml

Get-Content docker-compose.yml
docker compose up -d
Start-Sleep -Seconds 1
curl.exe "http://localhost:$HOST_PORT"
```

Take evidence: the compose lines, then `docker compose up -d` followed by curl showing `Edited without rebuild`.

## 6. Q3(ii) — pass the roll number to the container

Add this environment entry under `web`, while keeping the volumes entry:

```yaml
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
```

The final compose file should now be:

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
```

Replace it with one command:

```bash
cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
EOF
```

PowerShell can create the same file with `Get-Content docker-compose.yml` as a guide, then edit it in VS Code or copy the full here-string from the bash block (PowerShell accepts the YAML text unchanged).

Start Compose and print the environment variable:

```bash
docker compose up -d
docker exec "web-$ROLL_NO" printenv STUDENT_ROLL
```

PowerShell:

```powershell
docker compose up -d
docker exec "web-$ROLL_NO" printenv STUDENT_ROLL
```

Take screenshots of the compose environment lines and the command output showing your roll number.

## 7. Q4 — add a health check

Add this block under `web`, alongside `environment`:

```yaml
    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://127.0.0.1:80/"]
      interval: 10s
      timeout: 3s
      retries: 3
```

Replace the compose file with the complete final version:

```bash
cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://127.0.0.1:80/"]
      interval: 10s
      timeout: 3s
      retries: 3
EOF

cat docker-compose.yml
docker compose up -d
sleep 20
docker compose ps
```

PowerShell:

```powershell
@'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    restart: unless-stopped
    volumes:
      - ./site:/usr/share/nginx/html:ro
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://127.0.0.1:80/"]
      interval: 10s
      timeout: 3s
      retries: 3
'@ | Set-Content -Encoding ascii docker-compose.yml

Get-Content docker-compose.yml
docker compose up -d
Start-Sleep -Seconds 20
docker compose ps
```

The `web-$ROLL_NO` container must show `(healthy)`. If it is still `starting`, wait another 10 seconds and run `docker compose ps` again.

Take a screenshot of the healthcheck compose lines and a second screenshot of `docker compose ps` showing `(healthy)`.

Write two sentences in your own words:

1. The check uses container port `80` because that is the port nginx listens on inside the container.
2. The host port is only a mapping used outside Docker, so it is not needed for a request made inside the container.

## 8. Final evidence and PDF

Make sure these items are visible:

1. Cover page: name, roll number, section, Set A, and the typed declaration.
2. S0 screenshot.
3. Q1(a)–Q1(d): E1, E2 sentence, and E3 for each.
4. Q2: `docker run` output, curl output, and `docker ps`.
5. Q3: compose lines and command evidence.
6. Q4: compose lines and `(healthy)` evidence.

Useful final checks:

```bash
ls -la
cat docker-compose.yml
docker compose ps
curl "http://localhost:$HOST_PORT"
docker exec "web-$ROLL_NO" printenv STUDENT_ROLL
```

PowerShell:

```powershell
dir -Force
Get-Content docker-compose.yml
docker compose ps
curl.exe "http://localhost:$HOST_PORT"
docker exec "web-$ROLL_NO" printenv STUDENT_ROLL
```

Save the PDF as `<ROLL_NO>_MSE2_SetA.pdf`, in the order required by the exam instructions.
