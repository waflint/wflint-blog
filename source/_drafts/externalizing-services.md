---
title: Externalizing docker services with using cloudflared and Traefik
date: 2026-08-10 17:59:07
categories: Homelab
tags: ["server stuff", "homelab"]
description: "using cloudflared and traefik to externalize "
---

<!-- more -->

## Internal First

### The "Basics"

{% mermaid flowchart LR %}

    subgraph intranet
    direction LR
        subgraph host
            subgraph services
            traefik -- websecure 
            route--> application
            end
        end
    user --> DNS
    user --> traefik
    end
style host stroke-width:2px,stroke-dasharray: 5 5
style services stroke-width:2px,stroke-dasharray: 5 5

{% endmermaid %}

### Inter-stack Network

### Websecure and TLS

## External Second

{% mermaid flowchart LR %}

    subgraph intranet
        subgraph host
            subgraph services
            traefik -- websecure 
            route--> application
            
            end
        end
    intuser[user] --> intDNS[DNS]
    intuser[user] --> traefik
    end
    subgraph internet
        extuser[user] --> extDNS[DNS]
        extuser --> Cloudflare
        Cloudflare <-- cloudflared 
        tunnel --> traefik
    end
    
style host stroke-width:2px,stroke-dasharray: 5 5
style services stroke-width:2px,stroke-dasharray: 5 5
{% endmermaid %}

### Cloudflared

### Middlewares in Traefik