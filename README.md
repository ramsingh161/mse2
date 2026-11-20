# MSE-II Docker and Docker Compose Debugging Lab - Set A

This README contains the complete procedure for GitHub Codespaces. Run every command in the Codespaces terminal and replace the placeholders before starting.

## 1. Enter the values used throughout the lab

Set your university roll number and name:

```bash
export ROLL_NO="YOUR_ROLL_NUMBER"
export STUDENT_NAME="YOUR_NAME"
```

Use the last two digits of your roll number for the host port:

```bash
export HOST_PORT=$((8000 + 10#${ROLL_NO: -2}))
export SECOND_PORT=$((HOST_PORT + 1))
echo "Roll number: $ROLL_NO"
echo "Host port: $HOST_PORT"
echo "Second container port: $SECOND_PORT"
```

The example for roll number ending in `07` is port `8007`. The example for `45` is `8045`.

## 2. Set the prompt and create the lab folder

```bash
export PS1='[$ROLL_NO \t] \w \$ '
mkdir -p "mse2_${ROLL_NO}"
cd "mse2_${ROLL_NO}"
```

Create the starter files exactly as supplied.

### `.env`

```dotenv
ROLL_NO=CHANGE_ME
HOST_PORT=CHANGE_ME
```

Replace `CHANGE_ME` in `.env`:

```bash
cat > .env <<EOF
ROLL_NO=$ROLL_NO
HOST_PORT=$HOST_PORT
EOF
```

### `index.html`

```html
<html>
<body>
<h1>MSE-II Docker Lab</h1>
<p>Name: YOUR_NAME</p>
<p>Roll No: YOUR_ROLL_NO</p>
</body>
</html>
```

Create it with your details:

```bash
cat > index.html <<EOF
<html>
<body>
<h1>MSE-II Docker Lab</h1>
<p>Name: $STUDENT_NAME</p>
<p>Roll No: $ROLL_NO</p>
</body>
</html>
EOF
```

### Starter `Dockerfile`

```Dockerfile
FROM nginx:alpne
COPY index.htm /usr/share/nginx/html/index.html
EXPOSE 80
```

Create it:

```bash
cat > Dockerfile <<'EOF'
FROM nginx:alpne
COPY index.htm /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

### Starter `docker-compose.yml`

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    port:- "${HOST_PORT}:8080"
```

Create it:

```bash
cat > docker-compose.yml <<'EOF'
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    port:- "${HOST_PORT}:8080"
EOF
```

Check Docker and Docker Compose:

```bash
docker version
docker compose version
```

## 3. Screenshot S0

Display the prompt, command, and all four required files:

```bash
ls -a
```

Take a screenshot of the **whole terminal window**. It must show:

- the prompt containing `[ROLL_NO HH:MM:SS]`
- the `ls -a` command
- `.env`
- `Dockerfile`
- `docker-compose.yml`
- `index.html`

Name the screenshot `S0`.

## 4. Q1: find and fix four mistakes

Run this command after each fix:

```bash
docker compose up -d --build
```

If `curl` is tried immediately after startup, wait one second and retry.

### Q1(a): fix the misspelled compose key

Take an error screenshot first. Then compare the compose file with the spelling reference:

- `services`
- `build`
- `ports`
- `restart` when used

The starter key `port:-` must become `ports:`.

Open or print the file:

```bash
cat docker-compose.yml
nano docker-compose.yml
```

Change the `port:-` line to:

```yaml
    ports:
      - "${HOST_PORT}:8080"
```

Save and run:

```bash
docker compose up -d --build
```

Use a screenshot containing the Docker error for E1, the sentence for E2, and the corrected file plus successful command for E3.

Example sentence:

> I changed `port:-` to `ports:` because Docker Compose does not recognize `port:-` as a valid key.

### Q1(b): fix the Dockerfile base image name

Docker will stop at the Dockerfile line marked `>>>`. The word after `FROM` is misspelled:

```Dockerfile
FROM nginx:alpne
```

Open the file:

```bash
nano Dockerfile
```

Change it to:

```Dockerfile
FROM nginx:alpine
```

Save and run:

```bash
docker compose up -d --build
```

Evidence:

- E1: screenshot of Docker marking the wrong Dockerfile line
- E2: `I changed ____ to ____ because ____`
- E3: `cat Dockerfile` and the command working afterward

Example sentence:

> I changed `nginx:alpne` to `nginx:alpine` because `alpne` is a misspelled image name.

### Q1(c): fix the copied file name

Docker may report that `index.htm` was not found. Check the real file name:

```bash
ls
```

Open the Dockerfile:

```bash
nano Dockerfile
```

Change:

```Dockerfile
COPY index.htm /usr/share/nginx/html/index.html
```

to:

```Dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

Save and run:

```bash
docker compose up -d --build
```

Example sentence:

> I changed `index.htm` to `index.html` because the actual web page file was named `index.html`.

### Q1(d): correct the container-side port

Nginx listens on container port `80`, so the compose port mapping must use `80`, not `8080`.

Open and update:

```bash
nano docker-compose.yml
```

Change the port mapping to:

```yaml
    ports:
      - "${HOST_PORT}:80"
```

Save and run:

```bash
docker compose up -d --build
```

Inspect the container mapping:

```bash
docker compose ps
```

The `PORTS` column should map your host port to `80/tcp`, for example `0.0.0.0:8007->80/tcp`.

Test the page:

```bash
curl "http://localhost:${HOST_PORT}"
```

Wait and retry if it does not respond immediately.

Evidence:

- E1: `curl` error or wrong result and `docker compose ps` with the wrong port
- E2: `I changed ____ to ____ because ____`
- E3: corrected compose file and `curl` printing your page

Example sentence:

> I changed the container port from `8080` to `80` because nginx listens on port `80` inside the container.

At this point, your compose file should contain:

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
```

## 5. Q2: start a second container by hand

Confirm that the image exists:

```bash
docker images
```

Use the host port `HOST_PORT + 1`:

```bash
docker run --help

docker run -d \
  --name "second-${ROLL_NO}" \
  -p "${SECOND_PORT}:80" \
  "web-${ROLL_NO}:v1"
```

Test the second container:

```bash
curl "http://localhost:${SECOND_PORT}"
```

Show both containers and mappings:

```bash
docker ps
```

Remove the second container:

```bash
docker rm -f "second-${ROLL_NO}"
```

Evidence:

- `docker run --help` or the documentation and the Docker run command plus its output
- `curl` on the new port printing the page
- `docker ps` showing the Compose container and `second-ROLL_NO`, with both port mappings

## 6. Q3(i): edit the page without rebuilding

Create the mounted site directory and copy the page into it:

```bash
mkdir -p site
cp index.html site/index.html
nano site/index.html
```

Add this line inside the `<body>`:

```html
<p>Edited without rebuild</p>
```

Add a bind mount under `web` in `docker-compose.yml`:

```yaml
    volumes:
      - ./site:/usr/share/nginx/html:ro
```

The Compose container should now be similar to:

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
```

Restart without rebuilding:

```bash
docker compose up -d
docker compose ps
```

Test the edited page:

```bash
curl "http://localhost:${HOST_PORT}"
```

`curl` should show `Edited without rebuild`.

## 7. Q3(ii): pass the roll number into the container

Add this entry under `web` in `docker-compose.yml`:

```yaml
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
```

The final value comes from `.env`; do not replace it with a typed roll number.

Apply the Compose configuration:

```bash
docker compose up -d
```

Print the environment variable inside the container:

```bash
docker exec "web-${ROLL_NO}" printenv STUDENT_ROLL
```

The output must equal your roll number.

## 8. Q4: add a health check

Add this `healthcheck` under `web` in `docker-compose.yml`:

```yaml
    healthcheck:
      test: ["CMD-SHELL", "wget -q -O /dev/null http://127.0.0.1:80/"]
      interval: 10s
      timeout: 3s
      retries: 3
```

A complete final example is:

```yaml
services:
  web:
    build: .
    image: web-${ROLL_NO}:v1
    container_name: web-${ROLL_NO}
    ports:
      - "${HOST_PORT}:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
    environment:
      STUDENT_ROLL: "${ROLL_NO}"
    healthcheck:
      test: ["CMD-SHELL", "wget -q -O /dev/null http://127.0.0.1:80/"]
      interval: 10s
      timeout: 3s
      retries: 3
```

Apply and wait:

```bash
docker compose up -d
sleep 20
docker compose ps
```

The web container must show `(healthy)`.

If it is not healthy, inspect it:

```bash
docker compose ps
docker compose logs web
docker inspect "web-${ROLL_NO}" --format '{{json .State.Health}}'
```

Answer in two sentences:

> The health check uses container port `80` because the check executes inside the container and nginx listens there. `HOST_PORT` is used only for a host-to-container mapping and is not the port the container itself should test.

## 9. Capture and organize screenshots

Use the Codespaces camera or screenshot tool. For each image:

- capture the whole terminal window
- keep the prompt visible
- keep the command visible
- keep Docker output visible
- do not crop, edit, reorder, or reuse screenshots

Suggested labels:

- `S0`
- `Q1(a) E1`
- `Q1(a) E2`
- `Q1(a) E3`
- `Q1(b) E1`
- `Q1(b) E2`
- `Q1(b) E3`
- `Q1(c) E1`
- `Q1(c) E2`
- `Q1(c) E3`
- `Q1(d) E1`
- `Q1(d) E2`
- `Q1(d) E3`
- `Q2 run`
- `Q2 curl`
- `Q2 ps`
- `Q3(i) compose`
- `Q3(i) curl`
- `Q3(ii) compose`
- `Q3(ii) exec`
- `Q4 compose`
- `Q4 ps`

For Q1, use these file-viewing commands in the final E3 screenshots:

```bash
cat docker-compose.yml
cat Dockerfile
ls
```

## 10. Create the PDF

Use the order below:

1. Cover page:
   - name
   - roll number
   - section
   - Set A
   - typed sentence: `I, <name>, Roll No <ROLL_NO>, ...`
2. Step 0: `S0`
3. Q1:
   - Q1(a): E1, E2 sentence, E3
   - Q1(b): E1, E2 sentence, E3
   - Q1(c): E1, E2 sentence, E3
   - Q1(d): E1, E2 sentence, E3
4. Q2:
   - run command and output
   - curl result
   - docker ps result
5. Q3:
   - compose lines and restart/curl evidence
   - compose lines and printenv evidence
6. Q4:
   - compose lines
   - `(healthy)` result
   - two answer sentences

Name the PDF:

```text
<ROLL_NO>_MSE2_SetA.pdf
```

If LibreOffice or a text editor is available in your Codespaces environment, export or print the document to PDF. Download the PDF locally before submitting it.

## 11. Useful cleanup commands

Stop the Compose project:

```bash
docker compose down
```

Remove unused containers and images only after the PDF and evidence are complete:

```bash
docker rm -f "second-${ROLL_NO}" 2>/dev/null || true
docker compose down --rmi local
```
