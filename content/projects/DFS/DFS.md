---
title: "Distributed Content-Addressable Storage (DFS)"
date: 2026-07-11
summary: "A distributed, peer-to-peer, encrypted, and content-addressable file system written in Go from scratch."
draft: false
tags: ["Go", "Networking", "Distributed Systems"]
image: "/projects/DFS/dfs1.png"
---

# Distributed Content-Addressable Storage (DFS)

[Repository](https://github.com/keshavbansal015/DFS)

A modular, peer-to-peer (P2P), encrypted, and Content-Addressable Storage (CAS) distributed file system written in Go. This project implements low-level network transports, message framing, AES-CTR streaming encryption, and hashed directory structures from scratch without relying on complex external frameworks.

---

## Architecture Overview

The system is designed as a decentralized network of autonomous storage nodes (peers) communicating over TCP. Every node functions as both a client (uploading and requesting files) and a server (persisting files on disk and streaming them to other peers on demand).

---

## Core Characteristics

### 1. Distributed File System (DFS)
The project demonstrates core distributed systems capabilities:
* **Decentralized Topology**: Peers discover and connect to each other using bootstrap nodes configured on startup.
* **Network-Wide Replication**: When a node stores a file, it simultaneously writes a local copy and streams the encrypted bytes to all connected peers in its routing table.
* **Network Retrieval**: If a node queries a file that has been deleted locally, it broadcasts a retrieval request to the P2P network, pulls the encrypted stream from a peer, decrypts it, and restores it locally.

### 2. Content-Addressable Storage (CAS)
Rather than identifying files by arbitrary user-defined paths or filenames, files are addressed by their cryptographic hashes.
* **Deterministic Hashing**: Files are written to disk using a path generated from their key's SHA-1 hash.
* **Directory Partitioning**: To prevent folders from growing too large (which degrades OS filesystem performance), the 40-character hex string hash is split into 5-character segments, creating a nested directory structure.
  
  For example, if the file key is `"bestpicture"`, it is handled as follows:
  * **SHA-1 Hash**: `71056ad8aa24742ea41ea36fa2e3452a31636e82`
  * **Resolved Path**: `71056/ad8aa/24742/ea41e/a36fa/2e345/2a316/36e82`
  * **Resolved Filename**: `71056ad8aa24742ea41ea36fa2e3452a31636e82`

---

## Data & Communication Flow

### Store Protocol (Replication)
When you store a file on a node, it is saved locally in plaintext and transmitted encrypted across the network.

![Store Protocol](/projects/DFS/dfs2.jpeg)

### Get Protocol (Retrieval)
If a local lookup fails, the node requests the file from the network.

![Get Protocol](/projects/DFS/dfs3.jpeg)

---

## Cryptography

The project uses symmetric cryptography (AES-256 in CTR mode) to protect files:
1. **Encryption (`copyEncrypt`)**:
   * Generates a unique 16-byte cryptographically secure random Initialization Vector (IV).
   * Writes the 16-byte IV to the head of the stream.
   * Encrypts the remaining plaintext with the cipher block and streams it.
2. **Decryption (`copyDecrypt`)**:
   * Reads the first 16 bytes from the stream to extract the IV.
   * Initializes the AES-CTR cipher stream using the extracted IV and the symmetric key.
   * Decrypts the remaining stream and writes it to the destination.

---

## Custom Connection-Locking Protocol

To avoid mixing RPC structure parsing (using Go's `gob` decoder) and raw binary streaming over the same TCP connection:
1. The transport read loop continuously decodes 1-byte headers from the peer connection.
2. **`IncomingMessage` (0x01)**: The payload is treated as a standard serialized RPC message (Gob).
3. **`IncomingStream` (0x02)**: The transport read loop encounters the streaming flag, spawns a `WaitGroup.Add(1)` block, and **pauses reading**.
4. The higher-level `FileServer` reads exactly `msg.Size` bytes of raw file stream directly from the connection socket via `io.LimitReader`.
5. Once the file transmission is complete, the `FileServer` calls `peer.CloseStream()`, releasing the `WaitGroup` and resuming the connection's read loop.

---

## Execution Guide

### Prerequisites
* Go 1.18 or higher
* Make utility (optional but recommended)

### Commands

* **Build the Project**:
  ```bash
  make build
  ```
  This compiles the source code into the `bin/dfs` executable.

* **Run the Simulation**:
  ```bash
  make run
  ```

* **Run Tests**:
  ```bash
  make test
  ```
  Runs the complete suite of store, transport, and encryption unit tests.

---

## Gaps & Incomplete Features

While functional as a proof of concept, several severe architectural limitations must be addressed before this system can be deployed in production:


### 1. Lack of Key Exchange Protocol
> Nodes generate random 32-byte symmetric encryption keys independently on startup (`newEncryptionKey()`). In order for peers to successfully decrypt files pulled from other nodes, all nodes must share or negotiate the exact same symmetric key. A key exchange protocol (such as Diffie-Hellman) or secure configuration management is needed.

### 2. Missing Network Handshake Verification
> The `HandshakeFunc` defaults to `NOPHandshakeFunc`. In a secure distributed file system, nodes must execute handshakes to exchange protocol versions, discover other active peer lists, and authenticate via cryptographic tokens.

---

- This project is based on the course by [AnthonyGG](https://github.com/anthdm)
