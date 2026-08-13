---
title: Source Stage
---

## Source Stage

The Source stage is all about pointing the pipeline to the GitHub repository. Setting a default branch tells CodePipeline exactly which version of the code to use. Real-world projects usually have lots of different branches, so specifying the main branch ensures we only deploy the code that is actually ready for production.

## Webhooks

This stage is also where I enabled webhook notifications. The moment a code commit hits GitHub, the webhook detects the change and signals CodePipeline to start a new execution right away.
