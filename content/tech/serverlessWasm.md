---
title: Notes from the serverless wasm meetup
date: 2023-06-10
description: My notes from the serverless bangalore June meetup!
---

## Matt Bucher

- Made helm, ex microsoft, kubernetes early days contributor great guy

# Definition of Serverless:

- Pattern in which the dev does not write a server process(sockets or daemons), entry point is an event handler
- Deus, open source competitor to heroku PaaS
- Explored kubernetes there, ms accquires deus, built AKS there
- Had to write new dockerfiles and images for diff images and OSs
- VMs big workhorse of the cloud
- Most functions as a service are built as a VM, supposed to be super fast, but was built on tech too bloated
- VM package the entire OS, serveral minutes to start
- Containers easier to use than VMs, dockerfiles good, dev exp for containers was great. Took 10-20 sec always less than a min
- Made no sense to have a shortlived process, but wait for 10-20 secs
- Functions:
  - Event handler function love
  - Too slow(200 msec)
  - Fixed OS and Arch -> serverless?
  - Difficult dev experience

# Motivation at fermyon:

- Want fast start times
- Don't need to know about OS and arc
- Fermyon Spin: 52ms, AWS lambda 250ms
- If you hit the 100ms threshold to load a page SEO penalties
- AWS cappin sayin 200-500ms was good
- To hit good page speed, you'd have to focus a lot on optimisation and stuff
- WASM: near native start times
- See, google page speed ranking
- spin cold starts are like 1ms

# WASM:

- What is wasm? Its just another bytecode format that is platfrom and arch independent
- Designed so that in low power environments it can run as interpreted
- You can run it in JIT mode
- Make it execute at native speed
- Pre initilisation when js is used. Shaved off time doing this
- Mozilla putting scenes with wasm
- They did development in the open
- Cross browser support straight away
- People involved in standardising js were involved with wasm
- Ruby was supposed to be a shell like language, but then ruby on rails came out

# WASM strengths:

- Security sandboxing
- Cross platform and cross architecture
- Fast
- Ideally supports compile to all languages
- Containers weaker because of shared kernel, wasm has much tighter security, it restricts access to sys calls

# Use cases:

- BBC and amazon wrote their players on wasm, they have articles check it out

- With WASM you can pick the app you like! It takes communities to make WASM successful, compiler and toolchain of each lang needs to do it

- Mozilla gambled big with wasm, check redmonk to see wasm support rankings.

# Scripting vs compiled:

- Scripting needs to be interpreted. Approach was to compile the interpreter itself
- Compile interpreter itself into wasm

#### Bombardier Load test command

- every single request runs in an isolated sandbox
- every single request has its own memory address
- Less than ms performance
- Once you've built that wasm module you can run it anywhere, entirely isolated, near native performance
- visit kwasm.sh
