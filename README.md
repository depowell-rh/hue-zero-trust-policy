# Ansible Automation Platform + Event-Driven Ansible: Hue Zero Trust Demo
## Complete Setup Guide

### Demo Overview

**What This Demonstrates:**
- **Closed-loop automation** - System detects, analyzes, and remediates issues without human intervention
- **Zero Trust security model** - Continuous state verification and automatic policy enforcement
- **Event-Driven Ansible (EDA)** - Real-time response to infrastructure changes
- **Edge/disconnected operations** - Logic can run at the tactical edge

**The Story:**
A Philips Hue light represents a mission-critical system that must maintain a "green" (operational) state. When someone changes the light color using the Hue app (simulating an unauthorized change or failure), EDA detects the drift and automatically triggers Ansible to restore the green baseline.

**Customer Value Propositions:**
- **MTTR Reduction**: Mean Time to Repair drops from minutes to milliseconds
- **Disconnected Operations**: Can run entirely on local RHEL/OpenShift at the edge
- **Zero Trust**: Continuous validation and automatic policy enforcement
- **Self-healing infrastructure**: No human intervention required for remediation

---

## Hardware Requirements

| Component | Purpose | Notes |
|-----------|---------|-------|
| **Philips Hue Bridge** | Represents the control plane for your "mission system" | Any generation works |
| **Philips Hue Light Bulb** | Represents the mission-critical system being monitored | Color bulb required |
| **Portable Travel Router** | Creates isolated local network | Any portable router (TP-Link, GL.iNet, etc.) |
| **iPhone/Mobile Device** | Used to manually trigger "failures" via Hue app | Philips Hue app installed |
| **Laptop** | Runs ngrok tunnel | Must be on same network as Hue Bridge |

---

## Software/Access Requirements

- **AAP Environment**: Provisioned from Red Hat Demo Platform (RHDP)
- **ngrok Account**: Free tier works - https://ngrok.com
- **GitHub Access**: To fork/clone this repository
- **Philips Hue App**: Installed on mobile device

---

## Complete Setup Procedure

### Phase 1: Hardware Setup

#### Step 1: Configure the Travel Router
1. Power on your portable travel router
2. Note the WiFi SSID and password (usually on a sticker)
3. Configure the router with a simple network (example: `192.168.8.x` subnet)
4. Ensure DHCP is enabled

#### Step 2: Connect Hue Bridge to Router
1. Connect Hue Bridge to router via Ethernet cable
2. Power on the Hue Bridge
3. Wait for the lights on the bridge to stabilize (about 1 minute)

#### Step 3: Connect Devices to Router Network
1. Connect your laptop to the router's WiFi
2. Connect your mobile device to the router's WiFi
3. Verify connectivity between devices

#### Step 4: Set Up Hue Light in Hue App
1. Open Philips Hue app on mobile device
2. Allow it to discover the Hue Bridge
3. Pair your light bulb with the bridge
4. Set the light to **green** color
5. Set brightness to **maximum**
6. Note the light ID (usually "1" for the first light)

---

### Phase 2: Obtain Hue Bridge API Credentials

#### Step 1: Find Your Hue Bridge IP Address

**Option A - Using Hue App:**
- Open Hue app → Settings → Hue Bridges → (i) icon
- Note the IP address (e.g., `192.168.8.100`)

**Option B - Using Discovery Service:**
```bash
# Must be on the same network as the bridge
curl https://discovery.meethue.com
```

**Option C - Check Router DHCP Client List:**
- Log into router admin interface
- Look for device named "Philips Hue"

#### Step 2: Create API Username

1. **Physically press the button on top of your Hue Bridge**
2. **Within 30 seconds**, run this command from your laptop:

```bash
curl -X POST http://YOUR_BRIDGE_IP/api \
  -H "Content-Type: application/json" \
  -d '{"devicetype":"AAP_EDA_Demo"}'
```

3. You should receive a response like:
```json
[{"success":{"username":"YOUR_USERNAME_RESPONSE"}}]
```

4. **Save this username** - this is your `api_key`

#### Step 3: Test API Access

```bash
# Replace with your actual values
curl http://YOUR_BRIDGE_IP/api/YOUR_API_USERNAME/lights/1
```

You should see JSON output describing your light's current state.

#### Step 4: Document Your Values

Create a note with these values (you'll need them later):

```
Bridge IP: 192.168.8.100
API Username: YOUR_USERNAME_RESPONSE
Light ID: 1
```

---

### Phase 3: Set Up ngrok Tunnel

#### Step 1: Install ngrok

**On macOS:**
```bash
brew install ngrok
```

**On Linux:**
```bash
# Download from https://ngrok.com/download
sudo tar xvzf ~/Downloads/ngrok-*.tgz -C /usr/local/bin
```

**On Windows:**
- Download from https://ngrok.com/download
- Extract to a folder in your PATH

#### Step 2: Authenticate ngrok

```bash
# Get your auth token from https://dashboard.ngrok.com/get-started/your-authtoken
ngrok config add-authtoken YOUR_NGROK_TOKEN
```

#### Step 3: Start ngrok Tunnel to Hue Bridge

```bash
# This creates a tunnel to your Hue Bridge HTTP API
ngrok http http://YOUR_BRIDGE_IP:80 --domain=YOUR_STATIC_DOMAIN
```

**For free tier users (dynamic URLs):**
```bash
ngrok http http://YOUR_BRIDGE_IP:80
```

#### Step 4: Note the ngrok URL

ngrok will display output like:
```
Forwarding   https://unmystic-jonnie-nonterminatively.ngrok-free.dev -> http://192.168.8.100:80
```

**Save this forwarding URL** - this is your `bridge_url` for AAP.

#### Step 5: Test ngrok Tunnel

From any device with internet access (not on the local router network):

```bash
curl https://YOUR_NGROK_URL/api/YOUR_API_USERNAME/lights/1 \
  -H "ngrok-skip-browser-warning: true"
```

You should see the same JSON response as before.

**Important:** Keep this terminal window open during your demo - closing it stops the tunnel.

---

### Phase 4: Provision AAP Environment

#### Step 1: Order AAP from RHDP

1. Log in to https://demo.redhat.com
2. Navigate to **Services → Catalog**
3. Search for "Ansible Automation Platform"
4. Select **"Ansible Platform Demos"** or **"OpenShift Blank Environment" (you'd just install the AAP operator here)**
5. Click **Order** and fill in details
6. Submit the order
7. Wait for provisioning email (usually 15-30 minutes)

#### Step 2: Access Your AAP Environment

1. Open the email with subject "Your lab environment is ready"
2. Note these URLs:
   - **AAP Controller URL**: `https://...`
   - **AAP EDA Controller URL**: `https://...`
   - **Username**: Usually `admin`
   - **Password**: Provided in UI
3. Log in console to verify access

---

### Phase 5: Configure AAP Controller

#### Step 1: Create Inventory

1. Navigate to **Resources → Inventories**
2. Click **Add → Add inventory**
3. Configure:
   - **Name**: `Local Hardware`
   - **Organization**: `Default`
4. Click **Save**

#### Step 2: Add Host to Inventory

1. Click on **Local Hardware** inventory
2. Go to **Hosts** tab
3. Click **Add**
4. Configure:
   - **Name**: `hue_bridge`
5. Click **Save**

#### Step 3: Add Project

1. Navigate to **Resources → Projects**
2. Click **Add**
3. Configure:
   - **Name**: `Hue EDA Core`
   - **Organization**: `Default`
   - **Source Control Type**: `Git`
   - **Source Control URL**: `https://github.com/depowell-rh/hue-zero-trust-policy`
   - **Update Revision on Launch**: ✓ (checked)
4. Click **Save**
5. Wait for the project to sync (green status indicator)

#### Step 4: Create Job Template

1. Navigate to **Resources → Templates**
2. Click **Add → Add job template**
3. Configure:
   - **Name**: `Policy-Enforcement-Green`
   - **Job Type**: `Run`
   - **Inventory**: `Local Hardware`
   - **Project**: `Hue EDA Core`
   - **Playbook**: `remediate_green.yml`
   - **Credentials**: Leave empty (or use `Demo Credential` if available)
   - **Variables**: Add the following in YAML format:

```yaml
bridge_url: "https://YOUR_NGROK_URL"
api_key: "YOUR_HUE_API_USERNAME"
light_id: "1"
```

   - **Options**: 
     - ✓ Enable Webhook
     - ✓ Prompt on Launch (for Variables) - Optional

4. Click **Save**

#### Step 5: Test the Job Template

1. Click **Launch** on the `Policy-Enforcement-Green` template
2. Watch the job execution
3. Verify your Hue light turns green at maximum brightness
4. If it fails, check:
   - ngrok is still running
   - Variables are correct
   - Bridge API key is valid

---

### Phase 6: Configure EDA Controller

#### Step 1: Access EDA Controller

1. Navigate to the **AAP EDA Controller URL** from your RHDP email
2. Log in with the same credentials as AAP Controller

#### Step 2: Create Decision Environment (if none exists)

1. Navigate to **Decision Environments**
2. If `de-supported` or similar already exists, skip this step
3. Otherwise, click **Create decision environment**:
   - **Name**: `de-supported`
   - **Image**: `quay.io/ansible/ansible-rulebook:v1.0.0`

#### Step 3: Create Project in EDA

1. Navigate to **Projects**
2. Click **Create project**
3. Configure:
   - **Name**: `Hue EDA Core`
   - **SCM Type**: `Git`
   - **SCM URL**: `https://github.com/depowell-rh/hue-zero-trust-policy`
4. Click **Create project**
5. Wait for sync to complete

#### Step 4: Create Token for Controller Access

1. Navigate to **User Access → Users**
2. Click on your username (`admin`)
3. Go to **Tokens** tab
4. Click **Create token**
5. **Copy the token value** - you'll need it in the next step
6. Save it somewhere safe (you can't view it again)

#### Step 5: Create Controller Token Credential

1. Navigate to **User Access → Credentials**
2. Click **Create credential**
3. Configure:
   - **Name**: `AAP Controller Token`
   - **Credential Type**: `Red Hat Ansible Automation Platform`
   - **Red Hat Ansible Automation Platform URL**: Your AAP Controller URL
   - **Authentication Token**: Paste the token from Step 4
   - **Verify SSL**: ✓ (or uncheck if using self-signed certs)
4. Click **Create credential**

#### Step 6: Update Rulebook with Current ngrok URL

**IMPORTANT:** Your rulebook has a hardcoded ngrok URL that needs updating.

1. Go to your GitHub repository: https://github.com/depowell-rh/hue-zero-trust-policy
2. Navigate to `rulebooks/hue_rulebook.yml`
3. Click **Edit** (pencil icon)
4. Update line 7 with your current ngrok URL:

**Replace:**
```yaml
          - "https://YOUR_NGROK_TUNNUL_URL/api/YOUR_USERNAME_RESPONSE/lights/1"
```

**With:**
```yaml
          - "https://YOUR_CURRENT_NGROK_URL/api/YOUR_API_USERNAME/lights/1"
```

5. Commit the change
6. Go back to EDA Controller → Projects → `Hue EDA Core` → Click **Sync** to pull the update

#### Step 7: Create Rulebook Activation

1. Navigate to **Rulebook Activations**
2. Click **Create rulebook activation**
3. Configure:
   - **Name**: `Hue ZT` (or `Hue Zero Trust`)
   - **Project**: `Hue EDA Core`
   - **Rulebook**: `rulebooks/hue_rulebook.yml`
   - **Decision Environment**: `de-supported`
   - **Credentials**: Select `AAP Controller Token`
   - **Restart policy**: `On failure`
4. Click **Create rulebook activation**

#### Step 8: Enable the Activation

1. Find your `Hue ZT` activation in the list
2. Toggle the switch to **Enabled** (green)
3. Click on the activation name to view details
4. Go to **History** tab to watch for events

---

## Phase 7: Run the Demonstration

### Pre-Demo Checklist

- [ ] Travel router is powered on
- [ ] Hue Bridge is connected and operational
- [ ] Laptop is connected to router WiFi
- [ ] Mobile device is connected to router WiFi
- [ ] ngrok tunnel is running (`ngrok http http://BRIDGE_IP:80`)
- [ ] Hue light is currently green at full brightness
- [ ] AAP Controller is accessible
- [ ] EDA Controller is accessible
- [ ] Rulebook activation `Hue ZT` is **Enabled** (green)
- [ ] Variables in `Policy-Enforcement-Green` template are correctly set

### Demo Execution

#### 1. Show the Baseline State

**Say to audience:**
> "We have a mission-critical system represented by this green light. In a Zero Trust model, we continuously verify this system maintains its approved configuration baseline."

**Show:**
- The physical green Hue light
- AAP EDA Controller → Rulebook Activations → `Hue ZT` → History tab (monitoring events)

#### 2. Trigger the "Failure"

**Say to audience:**
> "Now imagine an unauthorized change occurs—perhaps an attacker, a configuration drift, or a system fault. Watch as our system detects and automatically remediates this in real-time."

**Action:**
- Open Philips Hue app on your phone
- Change the light to **red** or **blue** (any color except green)

#### 3. Show the Detection and Remediation

**Within 5 seconds, the following happens automatically:**

1. **EDA detects the drift** (url_check source polls every 5 seconds)
2. **Rulebook fires** (condition: `status == "up"` is met)
3. **Job template launches** (Policy-Enforcement-Green)
4. **Light returns to green** (remediation complete)

**Show:**
- EDA Controller → History tab showing the event
- AAP Controller → Jobs → Latest `Policy-Enforcement-Green` job (successful)
- The physical light returning to green

**Time from change to remediation:** ~5-10 seconds

#### 4. Explain the Architecture

**Say to audience:**
> "This demonstrates closed-loop automation. No human intervention was required. The system detected, analyzed, and remediated the issue autonomously."

**Show/Explain:**
- **Detection Layer**: EDA rulebook polling the Hue API
- **Decision Layer**: Rule condition evaluating the state
- **Remediation Layer**: Ansible playbook enforcing the baseline
- **Optional**: Show the rulebook YAML and playbook YAML in GitHub

#### 5. Discuss Mission Value

**Key talking points:**

- **MTTR Reduction**: "Traditional MTTR for this type of issue might be 15-30 minutes if someone has to notice, log in, and fix it. Here it's 5 seconds."

- **Scale**: "Imagine this across thousands of edge devices on ships, aircraft, or tactical vehicles—all self-healing without connectivity to a central NOC."

- **Zero Trust**: "We're not just monitoring for alerts; we're continuously enforcing policy. Any drift is immediately corrected."

- **Disconnected Operations**: "While I'm using ngrok for this demo, this exact rulebook could run on a local RHEL system at the edge with no cloud connectivity required."

---

## Troubleshooting Guide

### Issue: Variables undefined error

**Error:**
```
'bridge_url' is undefined
```

**Solution:**
1. Verify variables are set in the `Policy-Enforcement-Green` job template
2. Check that ngrok URL is correct and accessible
3. Ensure no extra spaces or quotes in variable values

---

### Issue: ngrok tunnel not accessible

**Error:**
```
Failed to connect to ngrok URL
```

**Solution:**
1. Verify ngrok is still running (check terminal)
2. Test the ngrok URL manually with curl
3. Check that you're using `https://` not `http://`
4. Verify the `ngrok-skip-browser-warning: "true"` header is present

---

### Issue: Rulebook activation fails to start

**Error:**
```
Activation status: Failed
```

**Solution:**
1. Check Decision Environment is available
2. Verify Controller Token credential is valid
3. Review activation logs for specific error messages
4. Ensure rulebook YAML syntax is correct

---

### Issue: Job template runs but light doesn't change

**Error:**
```
Job shows success but light stays red
```

**Solution:**
1. Verify `bridge_url` uses ngrok URL, not local IP
2. Check `api_key` matches your Hue Bridge API username
3. Confirm `light_id` is correct (usually "1")
4. Test API access manually:
   ```bash
   curl -X PUT https://YOUR_NGROK_URL/api/YOUR_API_KEY/lights/1/state \
     -H "Content-Type: application/json" \
     -H "ngrok-skip-browser-warning: true" \
     -d '{"on":true,"bri":254,"xy":[0.409,0.518]}'
   ```

---

### Issue: EDA not detecting changes

**Error:**
```
Light changes but no job is triggered
```

**Solution:**
1. Check rulebook activation is **Enabled** (green toggle)
2. Verify the URL in `hue_rulebook.yml` matches your current ngrok URL
3. Check EDA activation History tab for events
4. Ensure the Hue Bridge is responding to API calls
5. Confirm the rulebook condition logic (currently triggers on `status == "up"`)

---

## Post-Demo Teardown

### Step 1: Stop EDA Activation
1. EDA Controller → Rulebook Activations → `Hue ZT`
2. Toggle to **Disabled**

### Step 2: Stop ngrok
- Press `Ctrl+C` in the terminal running ngrok

### Step 3: Power Down Hardware
- Turn off Hue light
- Disconnect Hue Bridge
- Power off travel router

### Step 4: Delete RHDP Environment (Optional)
- RHDP environments auto-delete after the runtime expires
- Or manually delete from RHDP dashboard

---

## Important Notes for Future Demos

### ngrok URL Changes

**Free tier ngrok URLs change every time you restart ngrok.** This means you need to:

1. Update the `bridge_url` variable in the `Policy-Enforcement-Green` job template
2. Update the URL in `rulebooks/hue_rulebook.yml` (line 7)
3. Re-sync the project in EDA Controller

**To avoid this:** Consider upgrading to ngrok's paid tier for a static domain.

---

### Variables Quick Reference

When setting up AAP, you need these three variables:

| Variable | Value | Example |
|----------|-------|---------|
| `bridge_url` | ngrok tunnel URL | `https://abc123.ngrok-free.dev` |
| `api_key` | Hue Bridge API username | `YOUR_USERNAME_RESPONSE` |
| `light_id` | Light number in Hue system | `1` |

**Where to set them:** AAP Controller → Resources → Templates → Policy-Enforcement-Green → Variables section

---

### Hue API Endpoints Reference

**Get light status:**
```bash
curl https://YOUR_NGROK_URL/api/YOUR_API_KEY/lights/1 \
  -H "ngrok-skip-browser-warning: true"
```

**Set light to green:**
```bash
curl -X PUT https://YOUR_NGROK_URL/api/YOUR_API_KEY/lights/1/state \
  -H "Content-Type: application/json" \
  -H "ngrok-skip-browser-warning: true" \
  -d '{"on":true,"bri":254,"xy":[0.409,0.518]}'
```

**Get all lights:**
```bash
curl https://YOUR_NGROK_URL/api/YOUR_API_KEY/lights \
  -H "ngrok-skip-browser-warning: true"
```

---

## Demo Variations & Advanced Scenarios

### Variation 1: More Specific Drift Detection

Modify the rulebook condition to only trigger when the light is NOT green:

```yaml
rules:
  - name: Detect Non-Green State
    condition: event.url_check.status == "up" and event.url_check.body.state.xy != [0.409, 0.518]
    action:
      run_job_template:
        name: "Policy-Enforcement-Green"
        organization: "Default"
```

This prevents continuous enforcement and only remediates actual drift.

### Variation 2: Multiple Lights / Systems

Add additional lights to represent different systems:
- Green = Operational
- Red = Critical failure
- Blue = Maintenance mode
- Yellow = Degraded state

Create separate templates and rulebooks for each.

### Variation 3: Edge Deployment (No ngrok)

For fully disconnected demos:
1. Run AAP/EDA Controller on a local RHEL VM or OpenShift cluster
2. Connect it to the same network as the Hue Bridge
3. Use local IP addresses instead of ngrok URLs
4. Shows true tactical edge capability

---

## Additional Resources

- **Philips Hue API Documentation**: https://developers.meethue.com/
- **Event-Driven Ansible Documentation**: https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/
- **ngrok Documentation**: https://ngrok.com/docs
- **Red Hat Demo Platform**: https://demo.redhat.com

---

## Quick Start Checklist for Repeat Demos

**Before the demo:**
- [ ] Charge travel router
- [ ] Bring Hue Bridge, bulb, and Ethernet cable
- [ ] Install/update ngrok on laptop
- [ ] Order RHDP environment 24 hours in advance
- [ ] Verify Hue app is installed on phone

**Day of demo:**
- [ ] Set up hardware (router → bridge → light)
- [ ] Get Hue Bridge IP and API key
- [ ] Start ngrok tunnel
- [ ] Update AAP variables with current ngrok URL
- [ ] Update GitHub rulebook with current ngrok URL
- [ ] Enable EDA activation
- [ ] Test end-to-end once before presenting

**Duration:** Setup takes ~30 minutes once familiar with the process.

---

## Repository Structure

```
hue-zero-trust-policy/
├── README.md                          # High-level overview
├── remediate_green.yml                # Playbook to enforce green state
└── rulebooks/
    └── hue_rulebook.yml               # EDA rulebook for monitoring
```

---

## Version History

- **v1.0**: Initial working demo with continuous enforcement
- **Current**: Documented setup, ngrok integration, AAP variables configured

---

## Contact & Support

For questions about this demo setup:
- Repository: https://github.com/depowell-rh/hue-zero-trust-policy
- Internal Red Hat: Reach out via Slack or email

---

**Last Updated:** 2026-06-09
**Author:** Delano Powell, Red Hat Public Sector
**Demo Environment:** AAP 2.x + EDA Controller
