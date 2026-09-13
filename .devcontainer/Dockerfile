FROM alpine:3.20

# Basics + Node + npm
RUN apk add --no-cache bash ca-certificates git nodejs npm

# Install opencode (adjust package name if yours differs)
RUN npm i -g opencode-ai

# Non-root user
RUN adduser -D -u 1000 dev
USER dev

WORKDIR /workspaces
CMD ["bash"]