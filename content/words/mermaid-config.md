+++
title = "Mermaid diagrams on GitHub"
author = ["Sam Pillsworth"]
date = 2024-09-18T09:32:00-04:00
tags = ["reference", "mermaid", "github"]
draft = false
creator = "Emacs 29.2 (Org mode 9.8 + ox-hugo)"
+++

While working on a sequence diagram for a particular flow I lost a lot of time
trying to get a mermaid sequence diagram to not look chaotic, to not be so tiny
on my small laptop, and to have cute colours. I'm saving this for future
reference.

```mermaid
---
config:
  theme: base
  themeVariables:
    textColor: "#777"
    primaryColor: "#d33682"
  fontSize: 22
---
sequenceDiagram
    participant A
    participant B
    participant C
    A-->>+B: request to B
    B-->>-B: response from B
    A-->>+C: <br/><br/> request to C
    C-->>-A: response from C
    par [many requests in parallel]
    note right of B: remember!
    B-->>+C: <br/><br/> request to C
    B-->>-C: response from C
    end
```


## Notes about this config, in no particular order {#notes-about-this-config-in-no-particular-order}

-   I couldn't get `background` to work on GitHub to set a specific colour for the display window
    -   This is why I had to set the font colour to a midtone grey, to try to support both a light and dark background
-   the `<br/><br/>` adds some visual space, so that a request and it's response are clustered together compared to distinct requests
-   the `par` action text is not readable on the dark background, what font colour do I need to change for that one?
-   changing `fontSize` at the top level worked, but I couldn't get any of the sequence specific font configs to work
