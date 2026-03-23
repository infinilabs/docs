---
title: "Pipeline Processors"
date: 0001-01-01
summary: "Processor #  A processor performs specific actions on the input document when it flows through the pipeline:
[Doc input] --&gt; [Processor] --&gt; [Doc output] Pipeline #  A pipeline is basically multiple processors chained together, users can achieve sophisticated document processing via pipelines:
[Doc input] --&gt; [Processor A] --&gt; [Processor B] --&gt; [Processor C] --&gt; [Doc output] "
---


# Processor

A processor performs specific actions on the input document when it flows 
through the pipeline:

```text
[Doc input] --> [Processor] --> [Doc output]
```

# Pipeline

A pipeline is basically multiple processors chained together, users can achieve
sophisticated document processing via pipelines:

```text
[Doc input] --> [Processor A] --> [Processor B] --> [Processor C] --> [Doc output]
```