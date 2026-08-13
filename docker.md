# Docker - Why We Use It

Before spinning up Splunk (or a whole SOC lab), it's worth understanding **why Docker is the tool of choice** for standing this environment up. This section covers the reasoning; the rest of the doc covers the how.

**Contents**

1. [Why we use Docker](#docker---why-we-use-it) - the problem it solves and the key concepts
2. [Containers vs Virtual Machines](#containers-vs-virtual-machines) - why containers, not VMs
3. [Containers in Docker](#containers-in-docker) - what a container actually is
4. [Images](#images) - the blueprint containers run from
5. [Docker command reference](#docker-command-reference) - the CLI commands you'll use
6. [Prerequisites](#prerequisites) - what to install before the lab
7. [Deploying Splunk in Docker](#deploying-splunk-in-docker) - the step-by-step lab
8. [Reproducible setup with Docker Compose](#reproducible-setup-with-docker-compose) - the one-command version

## The core problem Docker solves

Software behaves differently across operating systems, versions, and dependency setups - the classic **"it works on my machine"** problem. Docker fixes this by **packaging an app with everything it needs** (OS libraries, runtime, configuration) into a **container** that runs the same way everywhere. You ship the environment, not just the code.

## Why it's useful here (Splunk / SOC context)

- **Spin up Splunk in one command** - instead of downloading an installer, configuring services, and wiring up dependencies by hand, you run a single `docker run` and get a working Splunk instance in minutes.
- **No mess on your host machine** - Splunk (and any test tools, log generators, or deliberately vulnerable apps you want to monitor) run **isolated** in containers. Delete the container and your laptop is clean again - nothing left installed or half-configured.
- **Reproducible lab** - you can define the entire SOC lab (Splunk + forwarders + a target machine generating logs) in a file and rebuild it **identically** any time. Ideal for learning and for sharing a setup with others.
- **Easy to reset** - broke something while experimenting? Tear the container down and start fresh in seconds, rather than reinstalling from scratch.

## The key concepts

A quick vocabulary so the commands later make sense:

| Term | What it is |
| ---- | ---------- |
| **Image** | A read-only blueprint/template for a container (e.g. the official `splunk/splunk` image). |
| **Container** | A running instance of an image - the actual live Splunk process. |
| **Volume** | Persistent storage that lets data survive even when the container is deleted. |
| **Docker Compose** | A file (`docker-compose.yml`) that defines and runs **multiple containers together** (e.g. Splunk + a data source) as one lab. |

## In one line

> Docker lets you run Splunk - and a whole SOC lab - in **isolated, reproducible, disposable** containers: fast to set up, easy to reset, and identical on any machine.

---

# Containers vs Virtual Machines

The natural question is: *why not just use a virtual machine?* VMs solve a similar problem, but they're the heavier, older approach. Understanding the difference is the clearest way to see why containers are the right fit for a SOC lab.

## What a Virtual Machine (VM) is

A **VM** emulates an *entire computer* in software. A layer called a **hypervisor** (e.g. VMware, VirtualBox, Hyper-V) carves your physical machine's resources into one or more virtual machines, and **each VM runs its own full guest operating system** - its own kernel, drivers, and system processes - on top of your real one.

So to run Splunk in a VM, you'd boot a whole guest OS (say, a full Linux install) *just* to run one application inside it.

## Why VMs aren't ideal here

- **Heavy** - every VM ships a complete OS, so it's measured in **gigabytes** and takes **minutes to boot**. A container shares the host's kernel and is measured in **megabytes**, starting in **seconds**.
- **Resource-hungry** - each VM reserves its own slice of CPU and RAM for a full OS. Running Splunk + a forwarder + a target machine as three VMs could swallow your whole laptop; the same as three containers is comparatively light.
- **Slow to spin up / tear down** - the fast "break it, reset it, try again" loop that makes a learning lab pleasant is painful when each reset means rebooting an OS.
- **Harder to reproduce & share** - a VM is a big opaque disk image. A container is defined by a small, readable text file (**Dockerfile** / **Compose**) you can version-control, diff, and hand to someone else.

## The key difference (why containers are lighter)

- A **VM virtualises the hardware** → every guest needs its **own full OS + kernel**.
- A **container virtualises the OS** → all containers **share the host's kernel** and package only the app and its dependencies.

That single distinction is why containers are smaller, faster, and cheaper to run.

```
    VIRTUAL MACHINES                    CONTAINERS

  ┌──────┐ ┌──────┐ ┌──────┐        ┌──────┐ ┌──────┐ ┌──────┐
  │ App  │ │ App  │ │ App  │        │ App  │ │ App  │ │ App  │
  ├──────┤ ├──────┤ ├──────┤        ├──────┤ ├──────┤ ├──────┤
  │Guest │ │Guest │ │Guest │        │ deps │ │ deps │ │ deps │
  │  OS  │ │  OS  │ │  OS  │        └──────┴─┴──────┴─┴──────┘
  └──────┴─┴──────┴─┴──────┘        ┌──────────────────────┐
  ┌──────────────────────┐          │    Docker Engine     │
  ├──────────────────────┤          ├──────────────────────┤
  │      Hypervisor      │          │       Host OS        │
  ├──────────────────────┤          ├──────────────────────┤
  │       Host OS        │          │       Hardware       │
  ├──────────────────────┤          └──────────────────────┘
  │       Hardware       │
  └──────────────────────┘
```

## In one line

> A **VM** virtualises a whole computer (full OS per app - heavy and slow); a **container** virtualises just the OS layer (shared kernel, app-only - light and fast). For a disposable, reproducible SOC lab, containers win.

---

# Containers in Docker

We've seen *why* containers beat VMs; this section looks more closely at what a container **is** in day-to-day use. Three properties capture it: a container is **self-contained**, **pre-packaged**, and it **only uses what it needs, when it needs it**.

## What a container actually is

A **container** is a **self-contained, running environment** for a single application. It bundles the app together with *everything it needs to run* - its code, runtime, system libraries, and configuration - into one unit that's isolated from everything else on the machine.

### 1. Self-contained (isolated)

Each container runs in its **own walled-off space**. It has its own filesystem, its own processes, and its own network view - so what happens inside one container doesn't touch the host or other containers.

- Splunk running in one container can't clash with a tool in another (no conflicting library versions, no "one install broke the other").
- Nothing gets installed onto your actual host OS - the mess stays inside the container.
- Delete the container and its whole environment vanishes cleanly, leaving no trace behind.

### 2. Pre-packaged (built from an image)

A container is created from an **image** - a ready-made, pre-packaged template that already contains the app and all its dependencies baked in (the [Images](#images) section covers this in depth).

- You don't assemble Splunk piece by piece; you pull the official `splunk/splunk` **image** and it's *already* got everything set up correctly inside.
- The image is **immutable** (read-only). Every container started from it is identical - that's what makes the setup reproducible across machines.
- "Pre-packaged" is exactly why it starts in seconds: the hard work of installing and configuring was done **once**, when the image was built.

### 3. Only uses what it needs, when it needs it (lightweight)

A container is **lean**. It carries only the dependencies that *its* app requires - not a whole operating system - and it consumes host resources (CPU, RAM) only while it's actually doing work.

- Because containers **share the host's kernel** (unlike VMs), they don't each haul around a full OS - just the app-specific bits.
- An idle container sits at near-zero resource use; it draws CPU/memory only when the app inside is active.
- This is why you can comfortably run **several** containers at once - Splunk plus a log generator plus a target host - on a normal laptop.

## In one line

> A container is a **self-contained, pre-packaged, lightweight** environment: isolated so it can't clash with anything else, built from a ready-made image so it's instantly reproducible, and lean enough to run only what it needs, when it needs it.

---

# Images

Containers keep coming back to one thing: the **image** they're launched from. It's worth understanding properly, because almost everything you do in Docker starts with pulling or building an image.

## What an image is

An **image** is the **read-only blueprint a container is created from**. It's a pre-packaged snapshot containing the application, its runtime, system libraries, and configuration - everything the app needs - frozen at build time. When you start a container, Docker takes an image and runs a live, writable instance of it.

> **Image vs container:** the image is the **template**; the container is the **running thing**. One image → many identical containers. (Analogy: an image is a *class*, a container is an *object*; or an image is a cake *recipe*, containers are the *cakes* you bake from it.)

## Images are built in layers

An image isn't one solid blob - it's a **stack of read-only layers**, each representing one step in how the image was built (e.g. "start from Linux," "install Splunk," "copy in config").

- **Cached & reused** - if two images share a base layer (say, the same Linux base), Docker stores it **once** and reuses it. That's why the *second* image you pull is often much faster and smaller on disk.
- **Efficient rebuilds** - change one step and Docker only rebuilds that layer and the ones after it, not the whole image.
- **The container adds one thin writable layer** on top of the read-only image layers. All your runtime changes live there - which is why deleting the container discards those changes but leaves the image untouched.

## Where images come from

- **A registry (pull)** - most images are downloaded from a **registry**, a hosted library of images. **Docker Hub** is the default public one; `docker pull splunk/splunk` fetches the official Splunk image from there.
- **A Dockerfile (build)** - you can also **build your own** image from a **Dockerfile**: a plain-text recipe listing the steps (base image, packages to install, files to copy, config). `docker build` turns that recipe into an image. This is how you'd bake a custom Splunk setup - pre-loaded apps, configs, or sample data - into a reusable image.

## Tags - picking a version

Images are named `repository:tag`, where the **tag** identifies the version.

- `splunk/splunk:9.2` - a specific, pinned version.
- `splunk/splunk:latest` - whatever the maintainer last marked "latest" (convenient, but it can change under you - **pin a real version** for a reproducible lab).

## Why this matters for the lab

- **Reproducibility** - a pinned image guarantees everyone runs the *exact* same Splunk build; no "works on mine" drift.
- **Speed & disk savings** - shared, cached layers mean pulling a second related image is quick and light.
- **Portability** - the image is self-contained, so the same one runs identically on your laptop, a teammate's, or a server.

## In one line

> An **image** is the read-only, layered blueprint - pulled from a registry like Docker Hub or built from a Dockerfile - that containers are launched from: build/pull once, run many identical containers anywhere.

---

# Docker command reference

Before the deploy steps, here's the vocabulary - the handful of `docker` commands you'll actually use, grouped by what they act on. Docker's CLI follows a consistent shape: **`docker <object> <action>`** (e.g. `docker container ls`, `docker image rm`). The most common ones also have **short aliases** (e.g. `docker ps` = `docker container ls`), which is what you'll see in the wild and below.

## Working with images

| Command | What it does |
| --- | --- |
| `docker pull <image>` | Download an image from a registry (e.g. `docker pull splunk/splunk:latest`). |
| `docker images` | List the images you've downloaded locally. |
| `docker rmi <image>` | Remove an image (frees disk space). |
| `docker build -t <name> .` | Build an image from a `Dockerfile` in the current directory. |

## Running & managing containers

| Command | What it does |
| --- | --- |
| `docker run <image>` | Create **and** start a new container from an image (the workhorse - see flags below). |
| `docker ps` | List **running** containers. Add `-a` to include stopped ones. |
| `docker stop <name>` | Gracefully stop a running container. |
| `docker start <name>` | Start a stopped container again. |
| `docker restart <name>` | Stop then start (e.g. to apply a change). |
| `docker rm <name>` | Delete a container (add `-f` to force-remove a running one). |
| `docker logs <name>` | Print a container's logs. Add `-f` to **follow** (live tail). |
| `docker exec -it <name> <cmd>` | Run a command **inside** a running container (e.g. `docker exec -it splunk bash` for a shell). |

## Common `docker run` flags

`docker run` is the one command worth knowing in detail - these are the flags this guide uses:

| Flag | Meaning |
| --- | --- |
| `-d` | **Detached** - run in the background instead of tying up your terminal. |
| `--name <name>` | Give the container a memorable name (otherwise Docker invents one). |
| `-p <host>:<container>` | **Publish a port** - map a container port to one on your machine (e.g. `-p 8000:8000`). |
| `-e KEY="value"` | Set an **environment variable** inside the container (how Splunk is configured). |
| `-v <name>:<path>` | Mount a **volume** for persistent storage (e.g. `-v splunk-var:/opt/splunk/var`). |
| `-it` | Interactive + TTY - keep a terminal attached (used with `exec`/shells, not with `-d`). |

## Housekeeping

| Command | What it does |
| --- | --- |
| `docker volume ls` | List volumes (persistent data stores). |
| `docker system df` | Show how much disk images/containers/volumes are using. |
| `docker system prune` | Reclaim space by deleting **unused** containers, networks, and images (asks first). |

## Dockerfile instructions (the *build* side)

The commands above drive Docker from the outside. A **`Dockerfile`** is the recipe for *building* an image (from [Images → Where images come from](#where-images-come-from)). Its main instructions:

| Instruction | What it does |
| --- | --- |
| `FROM <image>` | The **base image** to build on (every Dockerfile starts here). |
| `RUN <cmd>` | Execute a command **at build time** (e.g. install a package) - creates a new layer. |
| `COPY <src> <dest>` | Copy files from your machine into the image (e.g. custom Splunk config). |
| `ENV KEY=value` | Set a default environment variable baked into the image. |
| `EXPOSE <port>` | Document which port the app listens on. |
| `CMD ["…"]` / `ENTRYPOINT ["…"]` | The default command that runs **when a container starts**. |

You won't need to write a Dockerfile for the basic lab - the official `splunk/splunk` image already has one. It matters once you want to **bake in** custom apps, add-ons, or config so they're part of a reusable image.

---

# Prerequisites

Before the deploy steps, get these in place. This is everything a teammate needs to go from a fresh machine to "ready to run the lab."

## 1. Install Docker

- **Windows / macOS** - install **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**. On **Windows**, Docker Desktop uses the **WSL2** backend - accept the prompt to install/enable WSL2 if asked (or run `wsl --install` once in an admin PowerShell, then reboot).
- **Linux** - install **Docker Engine** (`docker` + the Compose plugin) via Docker's [official install docs](https://docs.docker.com/engine/install/).

## 2. Give Docker enough resources

Splunk is **memory-hungry** - an under-resourced Docker is the single most common cause of the container failing to start or running slowly.

- **RAM:** Splunk needs roughly **2 GB+ free**; give Docker Desktop **at least 4 GB** (Settings → Resources → Memory).
- **CPU:** 2+ cores recommended.
- **Disk:** the `splunk/splunk` image is **~1.5-2 GB**; make sure you have a few GB free.

## 3. Verify the install

Confirm Docker is working before continuing:

```bash
docker --version        # prints the installed version
docker run hello-world  # pulls & runs a tiny test image, then exits
```

If `hello-world` prints a success message, Docker is ready. If you get a "cannot connect to the Docker daemon" error, **make sure Docker Desktop is actually running** (its whale icon should be steady in the tray/menu bar).

---

# Deploying Splunk in Docker

Everything above was groundwork; this is the payoff - standing up a real **Splunk Enterprise** instance in a container in a few minutes. It puts the earlier concepts to work: we **pull an image** (`splunk/splunk`), **run a container** from it, map some **ports**, and pass **configuration via environment variables**.

> **Prerequisites:** this assumes Docker is installed and running with enough resources - see [Prerequisites](#prerequisites) above.

> **Source:** based on Splunk's official guide, *[Deploy and run Splunk Enterprise inside a Docker container](https://help.splunk.com/)* (help.splunk.com, last updated 2025-07-04).

## Step 1 - Pull the Splunk image

Download the official image from Docker Hub:

```bash
docker pull splunk/splunk:latest
```

This fetches the pre-packaged **Splunk Enterprise** image (running as a 60-day trial). `latest` grabs the newest build; for a **reproducible** lab you can pin a version instead (e.g. `splunk/splunk:9.2`) - see [Tags](#tags---picking-a-version).

> **Tip - the image has built-in help.** Run `docker run -it splunk/splunk help` to print the image's own documentation: the supported environment variables, accepted `SPLUNK_START_ARGS`, and configuration options straight from the image itself. Handy when a variable or token changes between releases.

## Step 2 - Run the Splunk container

Start a container from the image:

```bash
docker run -d \
  --name splunk \
  -p 8000:8000 \
  -p 8088:8088 \
  -p 8089:8089 \
  -e SPLUNK_GENERAL_TERMS="--accept-sgt-current-at-splunk-com" \
  -e SPLUNK_START_ARGS="--accept-license" \
  -e SPLUNK_PASSWORD="Password123!" \
  splunk/splunk:latest
```

**What each part does:**

| Flag | Meaning |
| --- | --- |
| `-d` | **Detached** - run in the background, freeing your terminal. |
| `--name splunk` | Give the container a friendly name (so you can `docker stop splunk` etc.). |
| `-p 8000:8000` | **Splunk Web UI** - the browser interface (`host:container`). |
| `-p 8088:8088` | **HTTP Event Collector (HEC)** - the endpoint apps/services POST data to. |
| `-p 8089:8089` | **Management port** - the REST API / admin interface. |
| `-e SPLUNK_GENERAL_TERMS=...` | Accept Splunk's general terms (required to start). |
| `-e SPLUNK_START_ARGS="--accept-license"` | Accept the software licence non-interactively. |
| `-e SPLUNK_PASSWORD="Password123!"` | Set the **admin** password (must meet Splunk's complexity rules - 8+ chars). **Change this** for anything non-throwaway. |
| `splunk/splunk:latest` | The image to run (from Step 1). |

> **Ports recap:** `8000` = web UI, `8088` = HEC (data in), `8089` = management API. Mapping `host:container` makes each reachable on your machine at `localhost:<port>`.

> **Heads-up on the terms token:** the exact `SPLUNK_GENERAL_TERMS` value (`--accept-sgt-current-at-splunk-com`) signals acceptance of Splunk's **General Terms** and **can change between image releases**. If a new image rejects this string on startup, check the current value in Splunk's official [docker-splunk documentation](https://splunk.github.io/docker-splunk/) / the [deploy guide](https://help.splunk.com/) and use the token it specifies.

### On Windows (PowerShell)

The `\` line-continuation above is **bash** syntax. In **PowerShell** use the backtick `` ` `` - or just put it on one line:

```powershell
docker run -d --name splunk `
  -p 8000:8000 -p 8088:8088 -p 8089:8089 `
  -e SPLUNK_GENERAL_TERMS="--accept-sgt-current-at-splunk-com" `
  -e SPLUNK_START_ARGS="--accept-license" `
  -e SPLUNK_PASSWORD="Password123!" `
  splunk/splunk:latest
```

Splunk takes a **minute or two** to initialise on first run. Check progress with `docker logs -f splunk` (look for `Ansible playbook complete` / a "ready" message), then `Ctrl+C` to stop tailing.

**Verify it's ready.** The `splunk/splunk` image ships with a built-in **healthcheck**, so you don't have to guess - run:

```bash
docker ps
```

and look at the **STATUS** column. While it boots you'll see `(health: starting)`; once Splunk is fully up it flips to **`(healthy)`** - that's your reliable "it worked" signal:

```
CONTAINER ID   IMAGE                 STATUS                   PORTS                    NAMES
a1b2c3d4e5f6   splunk/splunk:latest  Up 2 minutes (healthy)   0.0.0.0:8000->8000/tcp   splunk
```

Only once you see `(healthy)` should you expect the web UI at `localhost:8000` to respond. If it's stuck on `(unhealthy)`, see [Troubleshooting](#troubleshooting).

## Step 3 - Log in to Splunk

1. Open a browser and go to **`http://localhost:8000`**.
2. Log in with:
   - **Username:** `admin`
   - **Password:** the value you set in `SPLUNK_PASSWORD` (`Password123!` above).

If the page won't load yet, Splunk is probably still starting - give it another minute and recheck the logs.

## Step 4 - Test and explore

You're in. Poke around the Splunk UI:

- Run a search - try `index=_internal` to see Splunk's own logs (proof the search pipeline works).
- Browse **Settings → Data inputs** to see how data gets onboarded (ties back to the [ingest methods](README.md#how-does-splunk-onboard--ingest-data)).
- Try the [SPL examples](README.md#spl-by-example) from the Splunk handbook against a dataset.

### Managing the container

| Command | Does |
| --- | --- |
| `docker ps` | List running containers (confirm `splunk` is up). |
| `docker logs -f splunk` | Follow the startup/runtime logs. |
| `docker stop splunk` | Stop the container (keeps it). |
| `docker start splunk` | Start it again. |
| `docker rm -f splunk` | Remove it (deletes the container). |

> **⚠️ Data persistence:** as run above, deleting the container **loses your Splunk data and config** (that's the disposable-container behaviour from earlier). To keep data across restarts/rebuilds, mount **volumes** for Splunk's data and etc directories, e.g. add `-v splunk-var:/opt/splunk/var -v splunk-etc:/opt/splunk/etc` to the `run` command. For a throwaway lab, the version above is fine.

## Troubleshooting

The most common ways the container fails to come up, and how to fix each. Start by checking status and logs:

```bash
docker ps -a          # is the container running, exited, or unhealthy?
docker logs splunk    # what did it say on the way down?
```

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| **Container exits seconds after `docker run`** (`docker ps -a` shows `Exited`); logs mention the password | `SPLUNK_PASSWORD` **doesn't meet complexity rules** (min 8 characters) | Remove it and re-run with a stronger password: `docker rm -f splunk`, then the `run` command with e.g. `SPLUNK_PASSWORD="Password123!"`. |
| **`run` fails immediately** with *"port is already allocated"* / *"bind: address already in use"* | **Port 8000 is already taken** - often a previous `splunk` container or another app | Remove the old container (`docker rm -f splunk`), or map a **different host port**: `-p 8001:8000` (then browse to `localhost:8001`). |
| **`docker ps` shows `(unhealthy)`** | Splunk is **still initialising**, or it **ran out of resources** | Wait 2-3 minutes and recheck - first start is slow. If it stays unhealthy, check `docker logs splunk` and confirm Docker has enough RAM (see below). |
| **Container is killed / keeps restarting**; slow; logs show OOM; **exit code 137** | **Out of memory** - Docker Desktop isn't allocated enough RAM | Increase Docker Desktop memory to **≥ 4 GB** (Settings → Resources → Memory), then re-run. See [Prerequisites](#prerequisites). |
| **`localhost:8000` won't load** but the container is up | Splunk **hasn't finished starting** yet | Give it another minute; tail `docker logs -f splunk` and wait for the "Ansible playbook complete" / ready message. |
| **Login rejected** at the web UI | Wrong credentials | Username is always **`admin`**; the password is the exact value you set in `SPLUNK_PASSWORD`. If unsure, remove the container and recreate it with a known password. |

> **When in doubt, start clean:** `docker rm -f splunk` removes the container, then re-run the [Step 2](#step-2---run-the-splunk-container) command. (Data isn't preserved unless you added volumes - see the persistence note above.)

---

# Reproducible setup with Docker Compose

The `docker run` command above works, but it's a long line to remember, retype, and get right every time. **[Docker Compose](https://docs.docker.com/compose/)** captures the exact same setup in a **version-controlled file** so the whole lab comes up with **one command** - and it's the natural thing to **commit and share** with the team. This is the reproducibility the earlier concepts kept pointing at, delivered.

## 1. Create `docker-compose.yml`

Save this in the repo root. It's the Step 2 `docker run` translated into a file - same image, ports, and environment variables - with **persistent volumes** added so your data survives restarts:

```yaml
services:
  splunk:
    image: splunk/splunk:latest
    container_name: splunk
    hostname: splunk
    ports:
      - "8000:8000"   # Web UI
      - "8088:8088"   # HTTP Event Collector (HEC)
      - "8089:8089"   # Management API
    environment:
      SPLUNK_GENERAL_TERMS: "--accept-sgt-current-at-splunk-com"
      SPLUNK_START_ARGS: "--accept-license"
      SPLUNK_PASSWORD: "${SPLUNK_PASSWORD}"   # read from the .env file below
    volumes:
      - splunk-var:/opt/splunk/var   # indexed data
      - splunk-etc:/opt/splunk/etc   # config

volumes:
  splunk-var:
  splunk-etc:
```

## 2. Keep the password out of the file

Notice the password isn't hard-coded - it's `${SPLUNK_PASSWORD}`, which Compose reads from a **`.env`** file sitting next to `docker-compose.yml`:

```dotenv
# .env
SPLUNK_PASSWORD=Password123!
```

> **Don't commit secrets.** Add `.env` to your **`.gitignore`** so the password never lands in git history. Commit the `docker-compose.yml` (safe, no secrets) and, optionally, a `.env.example` with a placeholder so teammates know what to set. For production you'd go further - Docker **secrets** or a secrets manager rather than an env file.

## 3. Bring the lab up (and down)

From the folder containing the file:

| Command | What it does |
| --- | --- |
| `docker compose up -d` | Create and start the lab in the background (pulls the image first time). |
| `docker compose ps` | Show status - wait for `(healthy)`, same signal as before. |
| `docker compose logs -f` | Follow the startup/runtime logs. |
| `docker compose stop` | Stop the container but keep it and its volumes. |
| `docker compose down` | Stop **and remove** the container (volumes are **kept** - data survives). |
| `docker compose down -v` | Remove the container **and its volumes** - a full clean wipe. |

Then log in exactly as in [Step 3](#step-3---log-in-to-splunk): `localhost:8000`, user `admin`, and the password from your `.env`.

> **Why this is the better default:** one file, one command, data persisted across restarts, and no 8-line command to fat-finger. It's `docker run` made repeatable - commit it and any teammate gets the identical lab.

---
