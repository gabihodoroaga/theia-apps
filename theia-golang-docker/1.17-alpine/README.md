## Theia Go Docker

### Build

```bash
docker build -t theiaide/theia-golang:1.13-alpine .
```

### Run

Run on http://localhost:3000 with the current directory as a workspace:

```bash
docker run --security-opt seccomp=unconfined -it --init -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia-golang:1.17-alpine
```

Options:
- `--security-opt seccomp=unconfined` enables running without [the default seccomp profile](https://docs.docker.com/engine/security/seccomp/) to allow Go debugging
