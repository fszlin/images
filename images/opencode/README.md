# opencode

Node-based image with the [opencode](https://opencode.ai) CLI.

    docker build -t opencode images/opencode
    docker run --rm -it -v "$PWD":/workspace opencode

Pass credentials at runtime (env vars or mounted files); never bake them in.
