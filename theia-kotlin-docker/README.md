## Theia Go Docker

### Build

```bash
docker build -t theiaide/theia-kotlin:alpine .
```

### Run

Run on http://localhost:3000 with the current directory as a workspace:

```bash
docker run -it --init -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia-kotlin:alpine
```
