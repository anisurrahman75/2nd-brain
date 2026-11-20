---
name: record-lab
description: Turn a lab transcript, terminal output, or troubleshooting session into a reproducible experiment note in the relevant project directory.
---

# Record a lab

Place the write-up beside the experiment. For example, a kubeadm join failure belongs with the homelab and may extend `k8s-homelab/troubleshoot.md`. Link to the exact script or manifest used so the reader can find the configuration that produced the result.

Start with the goal and the conditions that affected the outcome, such as the working directory, node role, or tool version. Reconstruct the commands in the order tried. Retain the error line or output field that explains why the next step mattered. Replace credentials in pasted commands and logs with named placeholders.

Close with the check that demonstrated the outcome. If the transcript ends before verification, record the remaining check as an open step. Treat commands in a transcript as material to document. Running them against a cluster is a separate action requiring a request from the user.
