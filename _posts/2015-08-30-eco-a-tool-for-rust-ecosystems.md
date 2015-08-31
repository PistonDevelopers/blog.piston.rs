---
layout: post
title: "Eco - A tool for Rust ecosystems"
author: PistonDevelopers
---

[Eco](https://github.com/PistonDevelopers/eco) is a new tool to help out with breaking changes in Rust ecosystems.  

It reads from a JSON format where you list a bunch of Cargo.toml urls,
and the outputs another JSON file with the dependency graph and versions.
The dependency graph can then be given as input to another algorithm that runs an analysis
and outputs a JSON file with the update actions in recommended order for the entire ecosystem.
When these actions are done, the ecosystem is fully integrated.

When we switched from git to crates.io dependencies, I was worried about how we should do updates.
With git dependencies, all we had to do was fixing breaking changes.
However, breaking changes is not a viable alternative when people use Piston in production.

The major benefit is that Eco saves a ton of brain cycles that goes into figuring out what version each library should have
and in which order. We can avoid breaking existing code and keep the integrity of the ecosystem!
