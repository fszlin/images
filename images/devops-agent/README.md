# devops-agent

Builds on the `opencode` image and adds DevOps tooling (kubectl, jq, ssh).
Extend the Dockerfile with the tools you need.

    docker build -t devops-agent --build-arg OPENCODE_IMAGE=opencode images/devops-agent
