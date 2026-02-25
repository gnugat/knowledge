# Package Manager Security in 2025: What's Next?

> **Talk Abstract**
>
> Package managers are critical infrastructure and prime targets for supply chain attacks. This talk examines recent security incidents across npm, PyPI, and other ecosystems to understand what threats apply to Composer and Packagist.
>
> We'll explore emerging security standards including SLSA, trusted publishing, build provenance attestations, and reproducible builds. You'll learn how GitHub Actions has become an attack vector, what organizations like OpenSSF and its Package Repository Working Group are doing, which improvements other language packaging ecosystems recently introduced and how two new projects funded by Germany's Sovereign Tech Agency will improve security for the PHP ecosystem.
>
> Whether you maintain packages or just run composer install, you'll gain practical insights into supply chain security threats and the tools being built to address them.

Speaker: Nils Adermann
* [Slides](https://naderman.de/slippy/slides/2025-11-28-SymfonyCon-Amsterdam-2025-Package-Manager-Security-in-2025-Whats-Next.pdf)
* [Similar video](https://www.youtube.com/watch?v=ldVFYHcUJJg)

## Recap

Nils Adermann opens with a clarification: this talk is not just about Composer or Packagist. He wants to look outward at other language ecosystems, understand what is happening to them, and bring that context back to the PHP world. The starting point is a graph from Sonatype's annual supply chain report showing a clear and steep upward trend in the number of supply chain attacks. The numbers are not specific to one registry or one language. Every major report tells the same story.

To make the threat concrete, Nils walks through a handful of recent incidents, focusing specifically on events from 2025.

### Recent attacks in the wild

The first example is **Nx S1ngularity** (August 2025), an attack on the Nx build system published to npm. The attacker gained access to a maintainer account and replaced the package with malware that ran as an npm post-install hook. The hook scanned the environment for secrets: crypto wallets, SSH keys, GitHub tokens, npm credentials, and more. What made this one novel is that it also checked whether a local LLM was installed and, if so, simply asked it to find and exfiltrate private data. Rather than hard-coding what to look for, the malware outsourced that decision to the model. The stolen data was published to public GitHub repositories where anyone could find and use it.

![npm: Nx S1ngularity (August 2025): malicious postinstall script scanning for secrets, publishing to public GitHub, and using a local LLM to extract more data](./007-package-manager-security-in-2025-whats-next/00-nx-s1ngularity.png)

The second example is the **qix incident** (September 2025). The attacker used a phishing site hosted at `npmjs.help`, a domain that looked legitimate enough to fool the target, combined with a convincing two-factor authentication reset email. The account holder reset their 2FA and the attacker gained full access. Eighteen widely-used packages were compromised, including `debug`, `chalk`, and `ansi-styles`, collectively downloaded 2.6 billion times per week. The malicious versions were live for only two hours, but that was enough: they had already been pulled into front-end builds and deployed to production systems. Browsers loaded malicious code that rewrote cryptocurrency transaction recipients and approval flows, redirecting funds to the attacker. The key lesson, as Nils puts it, is that social engineering can bypass two-factor authentication.

![npm: The "qix" Incident (September 2025): social engineering via fake npmjs.help domain and 2FA reset email, 18 packages compromised including debug, chalk, ansi-styles](./007-package-manager-security-in-2025-whats-next/01-qix.png)

The third example attracted significant press coverage: the **Shai-Hulud worm** (September 2025), named after the creature from Dune. It is a self-replicating worm that targets CI systems. Like the others, it scans for credentials, GitHub personal access tokens, and API keys to AWS, GCP, and Azure, then uploads them to a public GitHub repository. What makes it a worm is how it propagates: it uses the npm credentials it finds to replace the package it is running from with a new version containing a post-install script that executes the worm again. It also creates new GitHub Actions workflows in any other repository the stolen tokens can access. Dependent packages that run their next update pick up the infected version, get compromised, publish their own infected versions, and the cycle continues. Within a few hours, more than 500 packages were affected, and therefore thousands of applications.

![npm: Shai-Hulud Worm (September 2025): self-replicating worm scanning for credentials, propagating via npm post-install scripts and GitHub Actions workflows, 500+ packages compromised](./007-package-manager-security-in-2025-whats-next/02-shai-hulud.png)

A second variant appeared just days before this talk: **Sha1-Hulud** (November 24-26). It entered the ecosystem through two projects with insecure GitHub Actions workflow definitions, specifically those using `pull_request_target` and `workflow_run` in ways that allow a pull request author to inject code into the workflow. This version uses a pre-install script instead of post-install, making it more likely to succeed, and if it fails to find anything useful to propagate to, it wipes the home directory of whatever system it is running on. More than 700 packages were compromised, and because the stolen credentials were published publicly, researchers were able to count credentials from at least 26,000 repositories exposed.

![npm: Sha1-Hulud Worm (November 24-26): second version using pull_request_target and workflow_run injection, pre-install scripts, 700+ packages compromised, credentials from 26k+ repositories exposed](./007-package-manager-security-in-2025-whats-next/03-sha1-hulud-second-coming.png)

The fourth example is a campaign on PyPI targeting users of Solana cryptocurrency. Attackers published packages containing malware, then published additional packages that wrapped those as dependencies but offered genuinely useful functionality. They then went to Stack Overflow and recommended those packages as solutions to real questions. The documentation was polished. The library looked helpful. People installed it willingly. It also used social engineering to prompt users to enter their wallet seed phrases directly into the software.

The fifth and most technically inventive example is GlassWorm, a worm found on Open VSX, the Eclipse Foundation's VS Code extension marketplace. It followed a similar pattern of spreading from one extension to the next and harvesting credentials, but added some novel twists. For command and control, instead of a traditional botnet, it encoded instructions in a Solana blockchain transaction: immutable, anonymous, cheap to host, and unremarkable in network traffic. As a backup, it encoded instructions as base64 in the title and description of a Google Calendar event. Most striking of all, the obfuscated malware code used unprintable Unicode characters. Open the file in a code editor and you see nothing. A static analyser sees nothing. Even a human reviewer would have to know what they were looking for to notice that something was wrong.

### Attack vectors and an AI arms race

Stepping back from the examples, Nils identifies the main attack vectors: compromising maintainer accounts, exploiting CI pipelines (particularly GitHub Actions), dependency confusion, and typosquatting. The attack surface has also expanded beyond production servers. CI pipelines contain secrets and have the ability to publish to production systems. Developer machines carry private keys and sensitive data. Preview environments and cloud-based development tools introduce new exposure that did not exist a few years ago.

The problem is compounded by what Nils calls an AI arms race. Malware is increasingly built with AI assistance, can evolve faster, and is harder to analyse. Typosquatting has expanded to cover AI-hallucinated package names: if an LLM suggests a package that does not actually exist, an attacker can register it in advance and wait. On the defensive side, registries and security companies are increasingly using AI to detect malware, but both sides are scaling up simultaneously.

The core goal remains simple: what you install and execute should not attack you. The constraint is equally simple: you cannot manually review every change to every dependency, and you cannot write everything yourself.

### Security standards: SLSA and OpenSSF

The Open Source Security Foundation (OpenSSF), hosted at the Linux Foundation, has been working to define shared standards and a common language for discussing the security of software supply chains.

SLSA (Supply-chain Levels for Software Artifacts) defines a series of levels with two main tracks. The build track, which is the finalized one, starts with documenting how an artifact is built and collecting build logs, progresses to having those logs signed by a reliable build platform, and reaches its highest level when individual builds are isolated from each other and artifacts are cryptographically guaranteed not to have been tampered with. A dependency track is in working draft and maps more directly to package managers: from simply having a lockfile, to running something like `composer audit`, to mirroring dependencies locally, and finally to having automated policies that prevent installation of unverified or vulnerable packages.

A separate working group has published principles for package repository security, aimed at how a registry like Packagist is operated rather than how individual projects are managed. These also define multiple levels across authentication, authorization, general capabilities, and CLI tooling.

### GitHub Actions as an attack surface

Nils returns to the Sha1-Hulud entry point to explain the specific GitHub Actions misconfiguration that enabled it. Two workflow trigger types require particular care: `pull_request_target` and `workflow_run`. When these are used, a pull request author can sometimes cause values from their pull request (such as the PR title) to be interpolated directly into a shell command in the workflow definition. This is a template injection vulnerability. A carefully crafted PR title becomes a shell command that the workflow executes.

A static analysis tool called Zizmor can detect these patterns. It reads workflow definitions and flags places where template variables are used in unsafe ways. Nils recommends it to anyone maintaining open source projects with CI.

### What other ecosystems are doing

Two-factor authentication enforcement has been rolling out across registries for several years. npm went further than most: it removed TOTP support entirely and now requires either a hardware token or passkeys. Standard authenticator apps are no longer accepted.

**Trusted publishing** applies the OIDC identity federation concept (familiar from "Sign in with Google") to the act of publishing packages. Instead of storing a long-lived API token as a CI secret, a GitHub Actions workflow requests a short-lived OIDC token from GitHub, presents it to the registry, and the registry verifies the token's authenticity with GitHub's identity provider. The registry then issues a temporary, single-use token that is only valid for uploading one specific package. There are no permanent secrets to leak or rotate, and handover between maintainers is simpler because there are no credentials to transfer. PyPI pioneered this, and it has since come to RubyGems, Crates.io, npm, and NuGet.

**Reproducible builds** is the idea that rebuilding an artifact from its source should produce a byte-for-byte identical result, allowing anyone to verify that the published artifact matches the published source. The project is harder than it sounds: timestamps embedded in zip files, the version of the zip tool used, network access during builds, and countless other environment variables all cause outputs to differ. Google's OSS Rebuild project is attempting to do this at scale for major open source ecosystems. NixOS has a subproject pursuing the same goal. Both are struggling with the complexity.

**Build provenance** is the practical alternative. Rather than verifying the artifact itself, you record metadata about the build: what was built, where, when, by whom, on which platform, using which workflow. Sigstore, specifically its Rekor component, provides a transparency log for this data. A temporary signing key is issued via OIDC, the provenance is signed, and the key is immediately discarded. The result is a public, append-only record of all signatures that can be verified by anyone, with no persistent private key that could be stolen. Kubernetes has used this for years. npm and PyPI have recently added support, and Crates.io, Maven, and RubyGems are at various stages of adoption.

### The PHP ecosystem today

![Q&A: where does PHP/Composer stand compared to other package managers in terms of security?](./007-package-manager-security-in-2025-whats-next/04-how-safe.png)

Before describing what Composer is doing, Nils characterizes where PHP stands relative to these concerns.

Most PHP packages do not have a build step. The source code is the artifact. Composer retrieves packages by hitting a GitHub endpoint that generates a zip file of the repository at a specific commit or tag. This has a quiet security advantage: the build process for a library cannot be attacked by its own dependencies, because there is no build process that runs dependencies. The worm-style propagation seen in npm simply does not apply to the library distribution step.

The downside is that these zips are not reproducible (they contain timestamps) and are not immutable (GitHub does not guarantee they will remain unchanged forever). Packagist itself only stores metadata; it does not host code.

Composer also restricts when code can execute during installation. Post-install and pre-install scripts are only allowed in the root project, which is under the developer's control. Dependencies cannot define these hooks. However, packages designated as Composer plugins are an exception: they execute code during installation and require explicit, name-based approval from the developer. If an approved plugin is later compromised and a new malicious version is released, that version will execute on the next install. The trust placed in the plugin's distribution channel therefore needs to be proportionate.

### Composer's existing and upcoming security features

Composer has offered a vulnerability advisory database and the `composer audit` command since 2022, allowing developers to check whether any installed package version has a known CVE.

Composer 2.9, released in November of the year of this talk, added security blocking: Composer will refuse to install or update to a version of a package that appears in the vulnerability database unless you explicitly ignore the specific advisory. This is distinct from `composer audit`, which checks what is already installed. Between the two, you are protected both from inadvertently installing a vulnerable version and from running one that slipped through.

The next planned step is malware blocking. Nils announces a new sponsorship that will provide Composer with a Creative Commons-licensed malware feed, generated through a combination of LLM pre-analysis and human review, and integrated into Composer at no cost to users.

### Two projects funded by the Sovereign Tech Agency

Germany's Sovereign Tech Agency funds security improvements to critical open source infrastructure. Two Composer and Packagist projects are currently underway, organized through the PHP Foundation.

The **Transparency Log for Packages.org** is already partially deployed. Its purpose is to make all significant events on Packagist publicly auditable: metadata changes such as a package's source URL being modified, new versions being published or removed, tag modifications in the underlying Git repository, maintainer additions and removals, ownership transfers, and even password resets on maintainer accounts. The last item matters because a password reset is a common consequence of account takeover through social engineering, and recording it publicly gives security researchers a signal to investigate. The log will be accessible via both a web interface and an API, enabling the community to build tooling around it. This directly advances Packagist toward level three of the OpenSSF principles for package repository security.

The **Organizational Ownership** project addresses a different problem: teams currently have no first-class way to represent their organization on Packagist. Many resort to shared user accounts with shared passwords, which makes 2FA impractical and makes access management difficult when someone joins or leaves a team. There is also no clear owner for a vendor prefix: any maintainer of any package under a given prefix can create new packages in that namespace, and there is no easy way to revoke that access across the board. The solution is to introduce organization entities that can own packages and vendor prefixes, manage membership independently of individual user accounts, and make it possible to eventually enforce 2FA across all accounts. This work is planned for 2026.

### The road ahead

Two more initiatives are on Nils's wish list.

**OIDC authentication for Composer** would allow private package repositories to use the same identity federation mechanism as trusted publishing. A developer's Composer client could authenticate to a corporate repository using their company SSO, or authenticate in CI using GitHub's OIDC provider, without any stored credentials. A Shopware developer volunteered at the conference to send a pull request implementing this.

**Artifact Integrity for Composer and Packagist** is the largest and most consequential project. Today Packagist is metadata-only. If it were to start hosting artifacts, all the attack vectors that affect npm and PyPI would become relevant: malicious uploads, account compromises leading to tampered packages, and so on. Before that can happen safely, Packagist needs mandatory 2FA (hence the organizational ownership work), trusted publishing, and Sigstore build provenance attestations to establish that whatever artifact is stored on Packagist came from the public source code it claims to come from. This project would also eliminate the awkward workarounds that packages with real build steps (like PHPStan or PHPUnit) currently use to distribute their compiled artifacts, and would eventually remove the need for sub-repository splitting in large monorepos like Symfony.

Trusted publishing is already working as a proof of concept in Private Packagist. Scoping and planning for the broader project are ongoing. Funding conversations are in progress with Alpha-Omega, and Bunny.net has committed to providing the CDN for artifact hosting. Companies interested in contributing are encouraged to speak with Nils directly.

The talk closes with an honest assessment: PHP's ecosystem is currently better defended than some others by accident as much as by design, but the threat landscape is evolving fast enough that intentional, systematic improvements are necessary. The work is underway.
