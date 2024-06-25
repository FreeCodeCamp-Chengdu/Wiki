---
title: Kubernetes Threat Matrix
authorURL: ""
originalURL: https://kubernetes-threat-matrix.redguard.ch/
translator: ""
reviewer: ""
---

[![](/images/kubernetes-security-2572f24b.png)][1]

<!-- more -->

# [Kubernetes Threat Matrix][2]

| Initial access | Execution | Persistence | Privilege escalation | Defense evasion | Credential access | Discovery | Lateral movement | Collection | Impact |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [Using Cloud][3] | [Exec into Container][4] | [Backdoor Container][5] | [Privileged Container][6] | [Clear Container Logs][7] | [List K8s secrets][8] | [Access the K8s API server][9] | [Access cloud resources][10] | [Images from a private repository][11] | [Data Destruction][12] |
| [Compromised images in registry][13] | [bash/cmd in container][14] | [Writable hostPath mount][15] | [Cluster-admin binding][16] | [Delete k8s events][17] | [Mount service principal][18] | [Access Kubelet API][19] | [Container service account][20] |  | [Ressource hijacking][21] |
| [Kubeconfig file][22] | [New container][23] | [Kubernetes CronJob][24] | [hostPath mount][25] | [Pod / container name similarity][26] | [Access container service account][27] | [Network mapping][28] | [Cluster internal networking][29] |  | [Denial of Service][30] |
| [Application vulnerability][31] | [Application exploit (RCE)][32] | [Malicious admission controller][33] | [Access cloud resources][34] | [Connect from proxy server][35] | [Applications credentials in configuration files][36] | [Access Kubernetes dashboard][37] | [Applications credentials in configuration files][38] |  |  |
| [Exposed sensitive interfaces][39] | [SSH server running in inside container][40] |  | [Disable Namespacing][41] |  | [Access managed identity credentials][42] | [Instance metadata API][43] | [Writable volume mounts on the host][44] |  |  |
|  | [Sidecar injection][45] |  |  |  | [Malicious admission controller][46] |  | [CoreDNS poisoning][47] |  |  |
|  |  |  |  |  |  |  | [ARP poisoning and IP spoofing][48] |  |  |

#### What is the _Kubernetes Threat Matrix_?

The MITRE ATT&CK® framework is a knowledge base of known tactics and techniques that are involved in cyberattacks. Started with coverage for Windows and Linux, the matrices of MITRE ATT&CK cover the various stages that are involved in cyberattacks (tactics) and elaborate the known methods in each one of them (techniques). Those matrices help organizations understand the attack surface in their environments and make sure they have adequate detections and mitigations to the various risks. MITRE ATT&CK framework tactics include:

-   Initial access
-   Execution
-   Persistence
-   Privilege escalation
-   Defense evasion
-   Credential access
-   Discovery
-   Lateral movement
-   Impact

Many attack techniques are different in the context of Kubernetes than those that target Linux or Windows, the tactics on the other hand are actually similar. For example, a translation of the first four tactics from OS to container clusters would look like 1. “initial access to the computer” becomes “initial access to the cluster”, 2. “malicious code on the computer” becomes “malicious activity on the containers”, 3. “maintain access to the computer” becomes “maintain access to the cluster”, and 4. “gain higher privileges on the computer” becomes “gain higher privileges in the cluster”.

Microsoft therefore created the first Kubernetes attack matrix: an ATT&CK-like matrix comprising the major techniques that are relevant to container orchestration security, with focus on Kubernetes. Redguard has extended the version of the _Kubernetes Attack Matrix_, especially by adding specific examples to simulate the techniques and references to learn even more about them and related topics.

#### Thanks

Thanks to Microsoft for creating the initial version of the _Kubernetes Threat Matrix_ ❤️  
We really appreachiate your work which we further extended and made more accessible.

-   [Microsoft Blog Post: Threat matrix for Kubernetes][49]
-   [Microsoft Blog Post: Secure containerized environments with updated threat matrix for Kubernetes][50]

_powered by [Redguard AG][51]_

[1]: /
[2]: /
[3]: /initial-access/using-cloud/
[4]: /execution/exec-into-container/
[5]: /persistence/backdoor-container/
[6]: /privilege-escalation/privileged-container/
[7]: /defense-evasion/clear-container-logs/
[8]: /credential-access/list-k8s-secrets/
[9]: /discovery/access-the-k8s-api-server/
[10]: /lateral-movement/access-cloud-resources/
[11]: /collection/images-from-a-private-repository/
[12]: /impact/data-destruction/
[13]: /initial-access/compromised-images-in-registry/
[14]: /execution/bash-cmd-in-container/
[15]: /persistence/writable-hostpath-mount/
[16]: /privilege-escalation/cluster-admin-binding/
[17]: /defense-evasion/delete-k8s-events/
[18]: /credential-access/mount-service-principal/
[19]: /discovery/access-kubelet-api/
[20]: /lateral-movement/container-service-account/
[21]: /impact/ressource-hijacking/
[22]: /initial-access/kubeconfig-file/
[23]: /execution/new-container/
[24]: /persistence/kubernetes-cronjob/
[25]: /privilege-escalation/hostpath-mount/
[26]: /defense-evasion/pod-container-name-similarity/
[27]: /credential-access/access-container-service-account/
[28]: /discovery/network-mapping/
[29]: /lateral-movement/cluster-internal-networking/
[30]: /impact/denial-of-service/
[31]: /initial-access/application-vulnerability/
[32]: /execution/application-exploit-rce/
[33]: /persistence/malicious-admission-controller/
[34]: /privilege-escalation/access-cloud-resources/
[35]: /defense-evasion/connect-from-proxy-server/
[36]: /credential-access/applications-credentials-in-configuration-files/
[37]: /discovery/access-kubernetes-dashboard/
[38]: /lateral-movement/applications-credentials-in-configuration-files/
[39]: /initial-access/exposed-sensitive-interfaces/
[40]: /execution/ssh-server-running-in-inside-container/
[41]: /privilege-escalation/disable-namespacing/
[42]: /credential-access/access-managed-identity-credentials/
[43]: /discovery/instance-metadata-api/
[44]: /lateral-movement/writable-volume-mounts-on-the-host/
[45]: /execution/sidecar-injection/
[46]: /credential-access/malicious-admission-controller/
[47]: /lateral-movement/coredns-poisoning/
[48]: /lateral-movement/arp-poisoning-and-ip-spoofing/
[49]: https://www.microsoft.com/en-us/security/blog/2020/04/02/attack-matrix-kubernetes/
[50]: https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/
[51]: https://www.redguard.ch/