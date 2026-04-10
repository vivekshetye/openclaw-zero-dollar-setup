# Setup OpenClaw with Gemini 3.1 Flash-Lite + SearXNG for $0: Ultimate Cheat Sheet

### 1. The Environment Setup
* **UTM (macOS):** [mac.getutm.app](https://mac.getutm.app/).
* **Alternatives:** Use **VirtualBox**, **VMware**, or **WSL2** if you are on Windows/Linux.
* **ISO Image:** Download **Ubuntu 24.04 LTS (Noble Numbat)**.
    * *Apple Silicon:* Use **ARM64**.
    * *Intel/AMD:* Use **AMD64**.

### 2. UTM Configuration
1.  **Create New VM:** Click **+** → **Virtualize** → **Linux**.
2.  **Mount ISO:** Browse and select your `ubuntu-24.04-server-arm64.iso`.
3.  **Hardware Specs:**
    * **Memory:** 6GB minimum.
    * **CPU:** 4 Cores.
4.  **Storage:** 40GB+ 
5.  **Save:** Name it and hit Save.

### 3. Get Your Brain (Gemini API Key)
We use **Gemini 3.1 Flash-Lite**  for its speed and massive free tier.

1.  Go to [Google AI Studio](https://aistudio.google.com/).
2.  Click **"Get API key"**.
3.  Click **"Create API key in new project"**.
4.  Give it a Name.
5.  **Copy the key** and save it—you'll need it for the OpenClaw onboarding.

### 4. The Installation Loop
1.  **Boot:** Start the VM and select **"Try or Install Ubuntu Server"**.
2.  **Defaults:** Follow the prompts (Language, Keyboard).
3.  **Storage:** Choose "Use an entire disk" and select the virtual drive.
4.  **Profile:** Set your username and a password.
5.  **SSH:** Check the box for **"Install OpenSSH server"**—you’ll want this for remote  access.
6.  **Reboot & Clean:** Once it says "Installation Complete," **shut down** the VM.
    * **Crucial:** In the UTM sidebar, go to the **CD/DVD** section and click **Clear**. If you don't, it will just boot into the installer again.

---

### 5. Post-Install 
Now that you are logged into the terminal, let's turn this into a workstation.

#### Step A: Install the Ubuntu Desktop
To install GUI, run this:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ubuntu-desktop -y
```
*Note: This takes about 5-10 minutes. Once done, `sudo reboot` to see the login screen.*

#### Step B: Verify systemctl (Service Manager)

```bash
# Check version
systemctl --version

# If not found:
sudo apt update && sudo apt install systemd -y
```

---

### 6. Setup Private Search (SearXNG)
SearXNG is a privacy-respecting search engine that plugs into OpenClaw for $0.

#### Step A: Install Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and log back in for group changes!
```

#### Step B: Install SearXNG
```bash
# 1. Create directory structure
mkdir -p ./searxng/core-config/
cd ./searxng/

# 2. Fetch Compose and Environment files
curl -fsSL -O https://raw.githubusercontent.com/searxng/searxng/master/container/docker-compose.yml \
             -O https://raw.githubusercontent.com/searxng/searxng/master/container/.env.example

# 3. Configure environment
cp .env.example .env
# Generate a secret key and add it to .env
# (The transcript mentioned replacing SEARXNG_SECRET)
sed -i "s/SEARXNG_SECRET=.*/SEARXNG_SECRET=$(openssl rand -hex 32)/" .env

# 4. Fetch default settings and enable JSON output (Required by OpenClaw)
curl -fsSL -o ./core-config/settings.yml https://raw.githubusercontent.com/searxng/searxng/refs/heads/master/searx/settings.yml

# Tweak settings.yml to support JSON
sed -i 's/formats: \[html\]/formats: [html, json]/' ./core-config/settings.yml

# 5. Start SearXNG
sudo docker compose up -d
```
*Note: Ensure SearXNG is running on port **8080**. If configured correctly, your agent will use this to browse the web.*

---

### 7. Install & Onboard OpenClaw
OpenClaw is the proactive AI agent framework that ties everything together.

#### Step A: One-Command Installation
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

#### Step B: Important Onboarding Configuration
The installer will automatically start the onboarding process. Follow these selections:
1.  **Manual Setup:** Select `Manual`.
2.  **Local Gateway:** Select `Yes`.
3.  **AI Provider:** Select `Google`.
4.  **API Key:** Paste your **Gemini API Key** from Step 2.
5.  **Model:** Select `gemini-3.1-flash-lite`.
6.  **Gateway Port:** Keep default `18789`.
7.  **Gateway Bind:** Select `Loopback`.
8. **Gateway Auth:** Select `Token`
9. **Tailscale Exposure:** Select `Off`
10. **How do you want to provide the gateway token:** Select `Generate/Store plaintext token`
11. **Configure Chat Channels:** Select `Yes`
12. **Select Chat Channels:** Select `Telegram`
13. **Telegram Bot:** 
    *   Find `@BotFather` on Telegram.
    *   Type `/newbot`, name it, and get your **API Token**.
    *   Paste the token into the OpenClaw prompt.
14.  **DM Access:** 
    *   Find `@userinfobot` on Telegram.
    *   Get your **User ID** and paste it into the `allowlist`.
15. **Web Search:** Select `SearXNG Search`
16.  **SearXNG Base URL:** 
    *   URL: `http://localhost:8080` (Ensure the port is **8080**).
17.  **Skills:** Skip it you can add later.
18. Select `No` for api keys for all other services.
19. **Configure Plugins:** Select `@openclaw/searxng-plugin` 
20.  **Enable Hooks:** Hit Enter to enable all hooks and services.
21. **Install Gateway Service:** Select `Yes`
22. **Gateway Service Runtime:** Select `Node`

<i> Once Gateway is started Open using Web UI</i>


---

### 8. Prompts

#### Recommended Personality Prompt
> "I am <YOUR_NAME>. You will be my personal AI assistant called Claw-AI. You need to be concise, direct and always do thorough research and also criticize my thoughts while doing research and not be always agreeable to everything"

#### Memory Management Prompt
> "Maintain a clear separation between short-term and long-term memory (e.g., distinct memory/ structures). For each request, load memory selectively and efficiently—only retrieve information that is directly relevant to the current context. Prioritize cost efficiency by minimizing unnecessary memory access and avoiding redundant data loading.
> 
> Strictly adhere to all security instructions at all times, these must never be ignored or bypassed."

#### AI Research Agent Task Prompt
```markdown
I want you to act as an autonomous research agent and build me a structured knowledge base.

Topic: "How AI Agents are transforming software development in 2026"

Your job is to:
1. Search the web for high-quality and recent sources (blogs, articles, research, discussions).
2. For each useful result, scrape the FULL content (not just snippets).
3. Extract and synthesize insights across sources:
   - Key trends
   - Popular tools/frameworks
   - Real-world use cases
   - Developer pain points
   - Challenges and limitations

Then organize everything into a set of well-structured markdown files.

Create the following files:
- overview.md → high-level summary and why this topic matters
- trends.md → top trends with supporting insights
- tools.md → important tools/frameworks with descriptions
- use_cases.md → real-world applications and examples
- challenges.md → risks, limitations, open problems
- future_predictions.md → what’s coming next in 2–3 years
- README.md → explain the structure of this knowledge base

Important instructions:
- Always prefer scraping full content over search snippets
- Combine insights across multiple sources (don’t just summarize one page)
- Avoid hallucinations — rely only on extracted data
- Keep the writing clean, structured, and professional
- Use memory to store intermediate findings before writing files
- Make sure all files are consistent and well-organized

Final goal:
Produce a mini research repository with multiple markdown files that I can directly use.
```

