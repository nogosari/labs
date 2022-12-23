
# Labs Documentation
This docs contain documentation paper of testing anything. Lets get Started
---
## 🔥 Docker X Containerd
This session using official image from Nginx to test performance between two containerized platform (Docker & Containerd)
Just a simple test, run container and expose port 80. 

#### Results :
|  | Docker |Containerd |
|:--------:| :-------------:| :-------------:|
| CPU Load| 401M |420M |

**Winner : Docker**

---

## 🔥 Docker : Postgres x MySQL

This session compare between postgres & mysql. Which DB that using minimum resource on devices.  
Checked using `docker stats`

**Postgres :**

    CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT     MEM %     NET I/O      BLOCK I/O     PIDS
    fe2e2f130083   postgres-latest   0.00%     31.35MiB / 1.896GiB   1.61%     1.3kB / 0B   0B / 82.8MB   6

**MySQL :**

    CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O      BLOCK I/O        PIDS
    f34f5e8e82d6   mysql-test   0.66%     382.9MiB / 1.896GiB   19.72%    1.6kB / 0B   163MB / 34.4MB   38

**Winner : Postgres**

---
## 🔥 Test Error Inside App Simulation

This session we try to make nodejs app error, based on code.

Error Code Below :

```node
import express from 'express'
import { PORT } from './constant';
import router from './route';

(() => {
  // if(PORT === undefined) throw new Error('port env not defined')
  console.log('starting building docker image app')
  const app = express();
  app.use(router)
  app.listen(8080, () => {
    console.log(`app listening on port ${PORT}`)
  })
  throw Error()
})()
```

Command :

```docker
docker run -p 8080:8080 --name test-nodejs -d nogosari/testing-app
```

Log Error from Docker Logs :

```docker
2022-12-21 10:30:07 Wed, 21 Dec 2022 03:30:07 GMT body-parser deprecated undefined extended: provide extended option at build/route/index.js:8:40
2022-12-21 10:30:07 /app/build/index.js:17
2022-12-21 10:30:07     throw Error();
2022-12-21 10:30:07     ^
2022-12-21 10:30:07 
2022-12-21 10:30:07 Error
2022-12-21 10:30:07     at /app/build/index.js:17:11
2022-12-21 10:30:07     at Object.<anonymous> (/app/build/index.js:18:3)
2022-12-21 10:30:07     at Module._compile (node:internal/modules/cjs/loader:1165:14)
2022-12-21 10:30:07     at Object.Module._extensions..js (node:internal/modules/cjs/loader:1219:10)
2022-12-21 10:30:07     at Module.load (node:internal/modules/cjs/loader:1043:32)
2022-12-21 10:30:07     at Function.Module._load (node:internal/modules/cjs/loader:878:12)
2022-12-21 10:30:07     at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
2022-12-21 10:30:07     at node:internal/main/run_main_module:22:47
2022-12-21 10:30:07 error Command failed with exit code 1.
2022-12-21 10:30:07 yarn run v1.22.19
2022-12-21 10:30:07 $ node ./build/index.js
2022-12-21 10:30:07 starting building docker image app
2022-12-21 10:30:07 info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

Docker Status :

```docker
19ae1e1b382e   nogosari/testing-app   "docker-entrypoint.s…"   4 hours ago   Exited (1) 3 hours ago    test-nodejs
```

If you want to make your container always restart, use this parameters :
```
docker update --restart always test-nodejs
```

NOTE : But parameters `always` is very exhausted on system, alternative better using parameter below. So, there is a limit on how much restarting container.
```docker
docker update --restart on-failure:8 test-nodejs
```

---

## 🔥 Test Error Server Side Simulation

In this event, we simulate how if the problem is on server side. And then, we basically just restart server if there is a problem. From the last test, there is another method to auto start if there is an error, even from apps or server side.

Example Case : Run simple nginx on docker
```docker
docker pull nginx:latest
docker run --name nginx-test -d -p 8080:80 nginx:latest
```
Basically, if we restart server docker container wont run until we start manually. And then, we have to add this flags.
```docker
docker update --restart unless-stopped nginx-test
```

> Now your container always restart on error & after rebooting your system

---