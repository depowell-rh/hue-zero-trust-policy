# Event-Driven Ansible (EDA): Closed-Loop Resiliency Demo
This repository provides the blueprints, rulebooks, and playbooks to deliver a high-impact Closed-Loop Automation demo. The goal of this demo is to show how Ansible can enforce a baseline in a disconnected or shipboard environmen, remediating unintended changes at the "speed of the mission" without human intervention.
(In a further iteration of this demonstration, EDA will enforce a baseline(s) upone specific color changes)

This demo showcases a "Zero Trust / Self-Healing" loop:

Detection: A mission system goes "Dark" (simulated by changing a Hue light from green).

Analysis: EDA identifies the state change as a violation of the "Desired State." (Future state - currently, this setup solely pushes a baseline of "Green")

Remediation: Ansible automatically re-provisions the service/light to "Green" (Mission Ready).

### Necessary Hardware
To replicate this demo, you will need:

Philips Hue Bridge + LED Lightbulb: Your "Mission System."

Portable Travel Router: To provide a local subnet for the Bridge and your laptop.

Mobile Device: Running the Philips Hue App to "induce" failures manually.

ngrok: To tunnel the local hardware events to your AAP instance.

## Logic Architecture
1. The Ingress (ngrok)
Since your AAP environment (RHDP) is likely hosted in the cloud, but your hardware is local, use ngrok to bridge the gap:

### Bash
- On your local machine connected to the Hue router
ngrok http 5000
Point your event source/webhook to the provided ngrok URL. This allows the cloud-hosted EDA Controller to "reach back" to your local hardware.


## Setup Instructions for Red Hatters
Step 1: Provision your Environment
Order an AAP on OpenShift or AAP on RHEL lab from RHDP. While the underlying platform is robust, for the demo, simply treat it as your centralized Command & Control hub.

Step 2: Configure the Hardware
Connect your laptop and Hue Bridge to your travel router.

Find the IP of the Hue Bridge and generate an API Username (see docs/hue_setup.md).

Set your Hue Bridge IP and API Key as Extra Vars or Credentials in your AAP Controller.

Step 3: Launch the Activation
Push this repo to your project in AAP.

Create an EDA Activation using the rulebook in this repo.

Use the ngrok URL as your webhook endpoint.

### Mission Value for the Customer
MTTR Reduction: Show, don't just tell, how Mean Time to Repair drops from minutes to milliseconds.

Disconnected Operations: While the demo uses a tunnel for convenience, the logic can reside entirely on a local RHEL/OpenShift node at the Edge (e.g., a shipboard rack).

Zero Trust: Emphasize that the system is continuously verifying the state of the infrastructure and enforcing the security policy automatically.
