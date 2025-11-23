---
title: "Changing web hosts and domain transfer to Cloudflare"
description: "changeinmotion.tech is transitioning!"
pubDate: "Nov 18 2025"
heroImage: "/images/logo/post1.png"
---

- This is a test page.  Need to check the various MDX functions of Markdown and it's embedding of JavaScript XML.

Getting started here! How simple a process it was to transition over from bluehost to Cloudflare.  Of course there was a few small steps I had to learn but overall fairly simple.  

The bluehost interface allowed me to manage the entire process from their website.  I used CPanel to download my whole Wordpress site backup file. I was a bit surprised since the ICANN site showed my registar was a company called NETWORK SOLUTIONS, LLC. Thought I may have to pay to transfer from them as well.

![blog placeholder](/public/images/ICANN.png)

```markdown
> helm repo update  
> kubectl get pv,pvc,svc,sc -A --show-labels -o wide
```
A nice script that I made for zsh. Sadly it needs more work to encompass other shells I work in such as BASH.

````markdown
#!/bin/bash

echo "Enter the full path to your Talos config directory:"
read -e -p "Directory path: " DIR

if [ ! -d "$DIR" ]; then
  echo "❌ Directory not found: $DIR"
  exit 1
fi

KUBECONFIG_PATH="$DIR/alternative-kubeconfig.yaml"
TALOSCONFIG_PATH="$DIR/talosconfig"

# Create the env file
mkdir -p ~/.config
ENV_FILE=~/.config/talos-env.sh
cat <<EOF > $ENV_FILE
export KUBECONFIG="$KUBECONFIG_PATH"
export TALOSCONFIG="$TALOSCONFIG_PATH"
EOF

# Ensure it's sourced in .zshrc
if ! grep -q "source ~/.config/talos-env.sh" ~/.zshrc; then
  echo "source ~/.config/talos-env.sh" >> ~/.zshrc
  echo "✅ Added sourcing line to ~/.zshrc"
fi

echo "✅ Environment configuration saved!"
echo "Run: source ~/.zshrc  (or restart your terminal)"
````

Wow what incredible markup!