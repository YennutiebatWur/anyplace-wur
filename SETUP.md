# Anyplace Local Setup Guide

## Prerequisites

Install the following before starting:

- **Java 8** — [Eclipse Temurin 8](https://adoptium.net/)
- **sbt** — `brew install sbt`
- **MongoDB 5.0** — already in `mongodb-macos-x86_64-5.0.31/`
- **Node.js** — `brew install node`
- **bower** — `npm install -g bower`
- **grunt-cli** — `npm install -g grunt-cli`

---

## 1. Fix Java Version for sbt

Homebrew's `sbt` defaults to the latest Java, which is incompatible with Scala 2.12.
Force Java 8 by adding this to `~/.zshrc` (or `~/.bashrc`):

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-8.jdk/Contents/Home
```

Then reload:

```bash
source ~/.zshrc
```

Verify with: `java -version` — should show `1.8.x`.

---

## 2. Start MongoDB

```bash
cd /Users/yennutiebatnibi/Downloads/mongodb-macos-x86_64-5.0.31
./bin/mongod --dbpath data/
```

Leave this terminal open.

### Create app database user

In a separate terminal:

```bash
/Users/yennutiebatnibi/Downloads/mongodb-macos-x86_64-5.0.31/bin/mongo anyplace --eval '
db.createUser({
  user: "anyplace",
  pwd: "anyplace123",
  roles: [{ role: "readWrite", db: "anyplace" }]
})'
```

---

## 3. Configure the Server

> **Already done** in this repo — `server/conf/app.private.conf` is configured with the local MongoDB credentials. Skip this step unless setting up on a new machine.

Copy the example config:

```bash
cp server/conf/app.private.example.conf server/conf/app.private.conf
```

Edit `server/conf/app.private.conf` and set:

```hocon
server.address="http://localhost"
server.port="9000"
application.secret="<generate with: sbt playGenerateSecret>"

mongodb.hostname="localhost"
mongodb.app.username="anyplace"
mongodb.app.password="anyplace123"
mongodb.port=27017
mongodb.database="anyplace"

password.salt="<random string>"
password.pepper="<random string>"
```

---

## 4. Run the Backend

```bash
cd server
sbt run
```

Server starts on `http://localhost:9000`.
Verify: `http://localhost:9000/api/version`

> Keep this terminal open. Press Enter to stop the server.

---

## 5. Build the Frontend Web Apps

Each app needs to be built once. Run these from the repo root:

### Configure API URL (once, shared across all apps)

Edit `clients/web/shared/js/anyplace-core-js/api.js`:

```javascript
API.url = "http://localhost:9000/api"
```

### Build each app

```bash
# Viewer
cd clients/web/anyplace_viewer
npm install && bower install && npm install grunt --save-dev
node_modules/.bin/grunt deploy

# Architect
cd ../anyplace_architect
npm install && bower install && npm install grunt --save-dev
node_modules/.bin/grunt deploy

# Viewer Campus
cd ../anyplace_viewer_campus
npm install && bower install
node /path/to/anyplace_viewer/node_modules/grunt/node_modules/grunt-cli/bin/grunt deploy --force
```

> For `anyplace_viewer_campus`, use the grunt binary from `anyplace_viewer/node_modules` since its own grunt 0.4.x doesn't install a bin on Node 25.

---

## 6. Serve the Frontend

```bash
cd clients/web
npx serve -p 8080
```

Open in browser:

| App | URL |
|-----|-----|
| Viewer | http://localhost:8080/anyplace_viewer/ |
| Architect | http://localhost:8080/anyplace_architect/ |
| Viewer Campus | http://localhost:8080/anyplace_viewer_campus/ |

---

## 7. Create an Account

Register via the Architect UI sign-in form, or via curl:

```bash
curl -X POST http://localhost:9000/api/user/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Admin","email":"admin@test.com","username":"admin","password":"admin123","external":"anyplace"}'
```

---

## Notes
- To run on a different port: `sbt "run 9001"`
