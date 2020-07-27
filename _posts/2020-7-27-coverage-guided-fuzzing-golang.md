---
layout: post
title: Coverage Guided Fuzzing in Go 
categories: [Go,Fuzzing]
---

Recently I had the need to explore coverage guided fuzzing in [Go](https://golang.org/). Whilst there is a bit of information scattered around on multiple different sites, as someone who is fairly new to Go, I couldn't find a good concise source of information on what is already out there and the current state of play of fuzzer tooling within the Go world. 

# Introduction

To build secure and resilient systems, then it is important it is important to have the tools available to detect issues in code. Humans are not good at identifying complex edge cases and perform reasoning under assumptions when writing code. For many years fuzzing has been commonly used to find bugs within programs written in C/C++, however, only recently these techniques have been started to get applied more to managed languages (Go/Rust/Swift etc). As these languages offer memory safety, fuzzing managed languages leads to other bug classes being identified. In Go this typically exhibits in either a panic, crash of the program, out of memory condition or a hang. There is also the technique of differential fuzzing, where providing the same input to a set of similar programs and observing the results, can lead to semantic or logic bugs being discovered. This article focuses on the former and the tools which can be used to find these issues in Go programs. 

Whilst I was looking into this area a new draft design for the Go language to integrate fuzzing as a first class citizen was published. This [draft design](https://golang.org/s/draft-fuzzing-design) is still under discussion and the aim of it is to collect feedback before an intended proposal. Perhaps one day there will be fuzzing as a first class citizen within Go, however, until then a more custom approach will likely need to be used. 

# Fuzzers

## Go-Fuzz

The most famous and original coverage-guided fuzzer for Go is [Go-Fuzz](https://github.com/dvyukov/go-fuzz). Go-fuzz's acts similar to go tool build and provides source-to-source transformation to add coverage instrumentation. Go-fuzz was pretty much the de-facto fuzzer in the Go world and has found a significant amount of [bugs](https://github.com/dvyukov/go-fuzz#trophies). 

However, some issues arise due to the difficulty in integration with other build systems, difficulty in instrumentation with  source-to-source tranformation and producing slower code. More in depth discussions of these pitfalls can be found in this [thread](https://github.com/golang/go/issues/14565). 

Go-fuzz also provides the ability to produce an archive in which it is possible to link in Clang [libfuzzer](https://llvm.org/docs/LibFuzzer.html]). 

## cmd/compile  

In Go 1.14, native compiler instrumentation for libfuzzer was added by [mdempsky](https://twitter.com/mdempsky) in these two commits [one](https://github.com/golang/go/commit/e341e93c519ef22ed4759fd0b4643a30321b9222), [two](https://github.com/golang/go/commit/ea0b4e7c7db8c5d376e77fd3e6741d94685073ac). This code coverage instrumentation within the compiler provides the basis for tools to be written which make use of the feedback.   

Using `-gcflags=all=-d=libfuzzer -buildmode=c-archive` as arguments to Go build we can produce a c-archive which can be linked in with libfuzzer manually. 

[go114-fuzz-build](https://github.com/mdempsky/go114-fuzz-build) can be used as a wrapper to simplify this process and then then resulting c-archive can be linked in with the libfuzzer driver. 

## Fzgo 

Finally we have a prototype of [cmd/go: make fuzzing a first class citizen, like tests or benchmarks](https://github.com/golang/go/issues/19109) called [Fzgo](https://github.com/thepudds/fzgo). This makes use of Go-Fuzz to integrate it into `go test` functionality. 

# Build Systems and CI 

When performing fuzzing for the purposes of vulnerability research, it is often enough to just run fuzzers standalone and against a single version of the software. However, to support a scalable secure software development lifecycle in large scale projects, then it is important that fuzzing is integrated as parts of the build system and CI to provide continuous fuzzing. 

It is also important to make fuzzing as easy to do and as seamless as possible, so by making it more accessible to "normal" engineering, rather than just security specialists, will lead to much higher quality software being built. This is why build systems which provide this functionality, will make it easier for developers to integrate fuzz testing into their development practices.  

## OSS-Fuzz

[OSS-Fuzz](https://github.com/google/oss-fuzz) is well known and provides continious fuzzing for open source projects. For Go, OSS-Fuzz previously used to make use of Go-Fuzz for performing fuzzing of Go code, however was switched in April to make use of [native cmd/compile libfuzzer instrumentation](https://github.com/google/oss-fuzz/pull/3633). 

OSS-Fuzz currently makes use of [go114-fuzz-build](https://github.com/mdempsky/go114-fuzz-build) to compile and link with libfuzzer. 

There are a number of projects making use of this for fuzzing Go, for example [Kubernates](https://github.com/google/oss-fuzz/tree/master/projects/kubernetes) or the Go lang project itself [Go](https://github.com/google/oss-fuzz/blob/master/projects/golang/). 

## Bazel 

[Bazel](https://bazel.build/) is often used within the Go world as a scalable build system. Whilst there have been [Bazel libfuzzer rules](https://github.com/nelhage/rules_fuzzer) written in the past for fuzzing, there currently appears to be an on-going Google intern project to create [Bazel fuzzing rules](https://github.com/googleinterns/bazel-rules-fuzzing). 

A few projects were also found to contain custom Bazel rules for fuzzing:

* [Envoy](https://www.cncf.io/blog/2018/09/28/gsoc-2018-extending-envoys-fuzzing-coverage/)

* [Prysm](https://github.com/prysmaticlabs/prysm/tree/master/fuzz)

## Fuchsia

[Fuchsia](https://fuchsia.dev) is an open-source capability-based operating system being developed by Google. Fuchsia appears to have integrated fuzzing support for Go. This was previously documented [here](https://fuchsia.dev/fuchsia-src/development/testing/fuzzing/libfuzzer_go), however, this page seems to be unavailable currently. Digging into the source, however, we can see the [build rules](https://fuchsia.googlesource.com/fuchsia/+/master/build/fuzzing/) and how this has been integrated into the project. 

## Hosted Continious Fuzzing

There are also a number of hosted continious fuzzing services which support Go fuzzing. These are:

* [fuzzit.dev](https://fuzzit.dev/)
* [fuzzbuzz.io](https://fuzzbuzz.io/)

As these services are "fuzzing as a service" I have not had to chance to use them, however they deserve a mention as continious fuzzing platform providers. 