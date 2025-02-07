# Build stage: Compile the Go application
FROM golang:1.23.6-alpine AS builder  

# Install necessary tools and dependencies
RUN apk add --no-cache git curl

# Set the working directory inside the container
WORKDIR /app

# Copy go.mod and go.sum first (to cache dependencies)
COPY go.mod go.sum ./

# Download dependencies (this step will be cached)
RUN go mod download

# Install CompileDaemon for automatic reloads
RUN go install github.com/githubnemo/CompileDaemon@latest

# Copy only source files (avoids copying test files)
COPY . ./

# Final runtime stage: Use a lightweight image
FROM golang:1.23.6-alpine  

# Install necessary tools for runtime
RUN apk add --no-cache ca-certificates curl

# Set the working directory inside the runtime container
WORKDIR /app

# Copy the compiled binary from the builder stage
COPY --from=builder /go/bin/CompileDaemon /usr/local/bin/CompileDaemon
COPY --from=builder /app /app

# Expose the port your application will use
EXPOSE 8080

# Command to run the application in development mode
CMD ["CompileDaemon", "--build", "go build -o backend-app main.go", "--command=./backend-app"]
