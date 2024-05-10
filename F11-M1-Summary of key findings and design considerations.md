
# Summary of key findings and design considerations

## Contents
- [Why do bridges fail?](#why-do-bridges-fail)
    - [Reasons for Hacks on Cryptocurrency Bridges](#reasons-for-hacks-on-cryptocurrency-bridges)
    - [Reflections and Lessons Learned](#reflections-and-lessons-learned)
- [Detailed Description of the Wormhole Bridge Hack](#detailed-description-of-the-wormhole-bridge-hack)
    - [Incident Overview](#incident-overview)
    - [Analysis of the Issue and Cause](#analysis-of-the-issue-and-cause)
    - [Reflections and Lessons Learned](#reflections-and-lessons-learned)
- [Detailed Description of the Nomad Bridge Hack](#detailed-description-of-the-nomad-bridge-hack)
    - [Incident Overview](#incident-overview)
    - [Analysis of the Issue and Cause](#analysis-of-the-issue-and-cause)
    - [Reflections and Lessons Learned](#reflections-and-lessons-learned)

- [Detailed Description of the Ronin Bridge Hack](#detailed-description-of-the-ronin-bridge-hack)
    - [Incident Overview](#incident-overview)
    - [Analysis of the Issue and Cause](#analysis-of-the-issue-and-cause)
    - [Reflections and Lessons Learned](#reflections-and-lessons-learned)
- [Rosen's design principles](#rosens-design-principles)
- [Verification in other bridges](#verification-in-other-bridges)
- [How does Rosen Work?](#how-does-rosen-work)
- [Technical specification and deployment](#technical-specification-and-deployment)


# Why do bridges fail?

Bridges can fail due to a variety of reasons, including but not limited to, false deposits, validator flaws, validator takeover, smart contract vulnerabilities, centralization, insufficient security practices, insufficient risk management, integration issues, social engineering attacks, phishing and malware, human factor and social engineering, complexity of multi-chain bridges, and economic incentives.


In general, attack vectors can be summerized as:
- Smart contracts
- False verification
- Proof/Signature fabrication
- Forks and reorgs
- Cryptographic libraries and implementation
- Network attacks, secret exposure, DoS, …
- Weak security assumptions
    - small guardian set
    - improper distribution of keys
    - centralization

In the following, we will investigate these attack vectors and their implications on bridges' security characteristics.

### Reasons for Hacks on Cryptocurrency Bridges

- Smart Contract Vulnerabilities: One common reason for hacks on cryptocurrency bridges like Wormhole, Ronin, Nomad, and Qbridge is smart contract vulnerabilities. These bridges rely on smart contracts to facilitate the transfer of assets between different blockchains. If the smart contracts are not properly audited or have coding flaws, hackers can exploit these vulnerabilities to manipulate the bridges and steal assets.
    - Code Flaws: Bugs or flaws in the smart contract code can be exploited by attackers. These contracts handle complex interactions and transactions, and even minor oversights can lead to significant vulnerabilities.
    - Logic Errors: Incorrect implementation of the business logic of the bridge can allow attackers to manipulate transaction rules to their advantage.

- Centralization: Some cryptocurrency bridges may have elements of centralization, which can make them more susceptible to hacks. Centralized control points within the bridge infrastructure can become targets for hackers looking to compromise the entire system.

- Insufficient Security Practices: Bridges often have complex architectures which require thorough and frequent security audits. Skimping on or rushing these can leave security gaps.
    - Lack of Proper Security Audits: In the rush to launch new bridges and keep up with the latest trends, security audits may be skipped or not conducted thoroughly. Without comprehensive security audits, potential vulnerabilities in the bridge's code may go undetected, providing opportunities for hackers to exploit.
    - Lack of Real-time Monitoring: Failure to implement continuous monitoring systems to detect and respond to suspicious activities in real-time.


- Insufficient Risk Management: In some cases, cryptocurrency bridges may not have robust risk management practices in place. This can lead to inadequate monitoring of transactions, failure to detect unusual patterns, or inadequate response mechanisms in the event of a security breach.

- Integration Issues: Cryptocurrency bridges often need to interact with multiple blockchains and external systems. Integration issues between these different components can create weak points that hackers can target to compromise the bridge's security.

- Social Engineering Attacks: Sometimes, hacks on cryptocurrency bridges are the result of social engineering attacks rather than technical vulnerabilities. Hackers may trick individuals with access to critical systems or sensitive information into revealing their credentials or performing actions that compromise the bridge's security.

- Phishing and Malware: Hackers can also target users of cryptocurrency bridges through phishing attacks or by distributing malware. If users unknowingly provide their credentials

- Human Factor and Social Engineering
    - Phishing Attacks: Attackers might trick key personnel into revealing sensitive credentials or executing malicious transactions.
    - Internal Threats: Mismanagement or insider threats due to lack of adequate controls or oversight.

- Complexity of multi-chain bridges:  The need to maintain compatibility and interact with multiple blockchains increases complexity and the potential for security oversights. Adding a new chain to a multi-chain bridge will result in a non-linear complexity. Each new chain will have different security assumptions, execution environment, smart contract model, and specific requirements. The new chain's security model and specific requirements, should be cross-compatible with each of the existing chains and probably will result in unknown security vulnerabilities and effects on existing chains.

- Economic Incentives
    - High-value Targets: The high volume of funds passing through bridges makes them lucrative targets for hackers.
    - Reward Systems: Some bridges feature mechanisms that can be exploited for profit, like manipulating the tokenomics to benefit from arbitrage opportunities.




# Detailed Description of the Wormhole Bridge Hack
Wormhole is one of the leading cryptocurrency bridges, connecting various blockchains like Ethereum, Solana, and others to enable the transfer of assets across these platforms. In February 2022, Wormhole suffered a significant hack resulting in the loss of approximately $320 million worth of cryptocurrencies. Here's how the incident unfolded:

## Incident Overview
- Vulnerability Exploitation: The attacker exploited a specific vulnerability in the Wormhole network. This vulnerability was in the bridge's token swap and validation system.
- Signature Verification Issue: The core of the hack lay in the improper verification of the "guardian's" signatures, which are supposed to ensure that a transaction moving assets between chains is legitimate.
- Crafting of Fake Messages: The attacker was able to craft a message that falsely claimed a wallet had deposited 120,000 wrapped Ethereum (wETH) into Solana, without actually locking up the Ethereum. With this message, they claimed an equivalent amount of wETH on Solana.
- Extraction and Conversion: Once the attacker minted the wETH on Solana, they quickly began converting the assets across different platforms and withdrawing them, thereby laundering the stolen funds.

### Analysis of the Issue and Cause

#### Key Issues and Causes

- Improper Signature Validation: The primary issue was the flawed validation mechanism within the Wormhole bridge smart contracts. These contracts failed to adequately verify the signatures that authenticate transaction requests across chains.

- Smart Contract Design Flaw: The smart contract was designed in a way that it trusted the execution of cross-chain messages if they appeared to be from a legitimate guardian, without rigorous checks to validate the authenticity of the transaction.

- Lack of Robust Security Protocols: The bridge lacked sufficient security protocols and fail-safes that could have either prevented the false transaction or flagged it for review.

- Delayed Response: Once the hack was executed, the response to mitigate the fallout was not swift enough, allowing the hacker ample time to launder a significant portion of the stolen funds.

#### Reflections and Lessons Learned

- Increased Security Measures: Following the hack, Wormhole developers worked to address the security flaws by revising their smart contracts and enhancing the security measures and validation protocols within the bridge.
- Auditing and Continuous Monitoring: This incident underscored the importance of regular and thorough security audits by independent third parties. Continuous monitoring for unusual transaction patterns could also have mitigated the risk.
- Faster Incident Response: Developing a rapid response plan for potential security breaches is critical in minimizing damage and starting immediate recovery actions.

The Wormhole bridge hack serves as a powerful reminder of the potential vulnerabilities inherent in complex decentralized finance (DeFi) systems and the continuous need for advancements in cybersecurity measures within this space. Such high-profile incidents highlight the critical importance of ensuring the highest security standards and operational best practices to safeguard digital assets.


# Detailed Description of the Nomad Bridge Hack

Nomad is a popular cryptocurrency bridge that facilitates the transfer of tokens across different blockchains. In August 2022, the Nomad bridge was hacked, resulting in the loss of nearly $200 million in various cryptocurrencies. This incident was particularly notable for the manner in which it was executed.

## Incident Overview

- Initial Vulnerability Trigger: The hack was initiated by the exploitation of a smart contract vulnerability that was inadvertently introduced during a routine update to the bridge's smart contracts.
- Validation Logic Issue: The bridge's validation logic was flawed, allowing for the validation and processing of unauthorized transactions. Specifically, the update made it possible to spoof transactions without depositing any real funds.
- Mass Exploitation: Unlike typical breaches that are usually carried out by a single entity or a coordinated group, the Nomad hack unusual because after the initial exploitation was publicized via social media and blockchain data, numerous opportunists began to use the same vulnerability to drain funds from the bridge.
- Copying Transactions: Hackers and opportunistic users replicated the initial attack transaction, merely altering addresses to redirect the illicit funds to themselves. This type of activity propagated rapidly, leading to a massive and fast-paced drain of funds.

### Analysis of the Issue and Cause

#### Key Issues and Causes

- Smart Contract Coding Error: The primary cause was a critical mistake in the smart contract code introduced during an update. This error significantly altered the message verification logic, reducing it to a state where no real validation was conducted.

- Lack of Comprehensive Testing: The update that introduced the vulnerability appears to have been inadequately tested. Comprehensive testing, including security-focused regression tests, could have potentially caught this egregious error before deployment.

- Rapid Spread Due to Public Exploit: Once the vulnerability was discovered and utilized, details of the exploit were quickly shared across social media platforms, encouraging a wide range of actors to participate in the theft, which exacerbated the scale of the loss.

#### Reflections and Lessons Learned

- Enhanced Update Protocols: It is crucial for blockchain operations, particularly those handling cross-chain communications and transfers, to have stringent protocols for updates, including multiple stages of testing and reviews.
- Immediate Response Plans: The incident highlighted the need for having rapid response mechanisms in place. These plans should include ways to halt operations immediately upon detection of a potential security breach to prevent similar cascading effects.
- Community and Communication Management: Managing how information about vulnerabilities is communicated is crucial. In the case of Nomad, the widespread public sharing of the exploit method essentially turned a breach into a free-for-all, magnifying the damage.
- Education on Ethical Conduct: This incident also raises questions about community behavior in decentralized networks. Efforts should be made to foster a community that upholds ethical standards, especially in crisis scenarios.

The Nomad bridge hack serves as a stark reminder of the risks associated with smart contract vulnerabilities, particularly in the context of complex systems like blockchain bridges. It underscores the need for rigorous security protocols, extensive testing pre-deployment, and the establishment of rapid response mechanisms to address vulnerabilities responsibly when they are discovered.




# Detailed Description of the Ronin Bridge Hack
Ronin is a blockchain network designed specifically for the game Axie Infinity, which allows players to earn cryptocurrency through gameplay. In March 2022, the Ronin bridge, which enables users to transfer assets between Ethereum and Ronin networks, was compromised, resulting in a staggering loss of about $620 million.

## Incident Overview

- Social Engineering Attack: The attack involved a sophisticated social engineering technique aimed at a small group of Ronin and Axie Infinity employees.
- Compromised Private Keys: Through the social engineering scheme, attackers were able to compromise the private keys associated with five out of the nine validator nodes on the Ronin Network. By controlling five validators, the attackers had the supermajority needed to authorize fraudulent withdrawals.
- Execution of Unauthorized Transactions: Once in possession of the necessary validator approvals, the attackers issued two transactions: one withdrawing 173,600 Ethereum and the other 25.5 million USDC, all of which was routed out of the Ronin bridge.

### Analysis of the Issue and Cause

#### Key Issues and Causes

- Reliance on a Small Number of Validators: The primary security vulnerability in the Ronin bridge was its reliance on a limited number of validators to approve transactions. This setup created a situation where compromising a relatively small number of nodes could result in total control over the network.

- Insufficient Security Protocols for Validators: The validators compromised were part of the same organization (Sky Mavis, the developer of Axie Infinity), indicating a lack of sufficient independent oversight and security among validators.

- Effective Social Engineering: The attackers were able to successfully carry out a social engineering attack, indicating potential weaknesses in the employee security training and operational security measures within Sky Mavis.

- Delayed Detection and Response: The hack was not immediately detected; it took several days before Sky Mavis became aware of the unauthorized withdrawals, highlighting a lack of adequate monitoring systems for unusual transaction activities.

#### Reflections and Lessons Learned

- Diversification and Decentralization of Validators: To mitigate risks, it is crucial for networks like Ronin to increase the number of validators and ensure they are not controlled by a single entity. This spread helps prevent single points of failure.

- Enhanced Security Training and Protocols: Organizations must prioritize comprehensive security training for their employees to recognize and resist social engineering attempts.

- Real-time Monitoring and Alerts: Implementing advanced monitoring systems that can detect unusual patterns or unauthorized transactions in real time is critical for quickly addressing potential breaches.

- Rapid Incident Response: Developing and testing incident response plans is essential for minimizing damage and responding effectively to security breaches.

The Ronin bridge hack underscores several vital cybersecurity lessons, particularly the need for robust security practices around node validators, the importance of vigilant monitoring systems, and the continuous education of staff against sophisticated social engineering tactics. This incident serves as a significant reminder of the vulnerabilities associated with blockchain and crypto-related technologies, especially as they become more integral to financial systems worldwide. 





# Rosen's design principles
- Keep it simple
- A series of off-chain verification and on-chain actions
- Verification
- Off-chain (instead of smart contract based verification)
- Individual and independent
- Multi-channel
- Majority consensus
- Delayed
- Recurring
- Decentralized
- Decentralized watchers; open to everyone
- Decentralized guard set, medium sized and permissioned
- Open source
- Smart contracts on Ergo
- No need for SC on other chains; possible when necessary
- Threshold/Multi-sig wallet on other chains
- Reducing attack vector
- Linear expansion complexity (Ergo-centric)
- Stateless
- Cold storage, reduced incentive/gain
- Skin in the game, reward and penalty for good/bad behavior
- Low cost


# Verification in other bridges

- Smart-contract based verification (destination chain) of a proof of lock (in the origin chain)
    - Requirement: state of the origin chain in the target chain and a verification mechanism.
    - In most cases, this state is provided by decentralized relays.
    - In practice, this method has been compromised several times in the last year.
    - One major issue: verifying a proof only once. Prone to forks, proof fabrications, smart contract bugs,…
- Indirect verification 
    - Verifiers verify a proof made by other verifiers, not the event itself.
    - For example, in Gravity and Wormhole, guards in origin chain create a proof and guards on the destination chain verify that proof, not the event itself.
- Optimistic
    - Oracle updates are posted on the destination chain and will be valid if no one alerts for fraudulent update in a period of time.
    - Used in Nomad, in combination with the above methods.
    - Why? Reducing gas fees.
    - Vulnerable to liveness.


# How does Rosen Work?

Rosen's key entities are:
- Guard set
    - A set of well-known entities,with some Rosen token locked
    - Multi-sig wallet and contracts on Ergo side
    - Threshold wallet on Cardano side
    - Threshold wallet on Ethereum side


- Watcher set
    - Each origin chain has a watcher set, monitoring the events on that network
    - Watchers act independently
    - Each watcher has a limit of unresolved reported events. Permits are given by locking Rosen token

- Wallets
    - A multi-sig wallet on Ergo
    - A threshold wallet on Cardano
    - A threshold wallet on Ethereum
    - A token map for supported tokens
    - Token issue is unlocking from the bank
    - A multi-sig cold wallet on each network


## watcher's process

Each watcher locks some Rosen token on Ergo and receives a few permit tokens. With each permit token, a watcher can report an event.
When event is finalized, permit token is returned. If an event is not resolved, the permit token is being collected.
Kind of DoS protection and enforcing good behavior.

Watchers are monitoring the wallet (origin net).
When an event occurs, watchers begin creating commitments on Ergo.
When there are enough commitments, one watcher reveals the commitment by spending all of them and creating an event trigger box. This box contains the event data.
Monitor (off)-> Commit (on)-> Compare (off)-> Reveal and trigger (on). 


## Guards' process

-Guards look for event trigger boxes. 
Each guard individually and independently verifies the reported event against what’s available on the origin chain (off-chain).
One guard (initiator) creates an unsigned payment txn and notify others.
Each other guard creates the desired txn and compares with that txn.
Each guard sends (dis)agreement message to the initiator guard.
If enough agreement received, the signing procedure begins.
When enough signatures are collected, txn will be broadcasted.

A lot of recurring verifications and checks are being done during a guard’s process:
- Enough confirmations 
- Message verification (authenticity of messages from other guards)
- Event verification
- Double spending (two reports for one event, two payment for a single report,…)
- Protection against partial unresolved txns (e.g.when a malicious guard doesn’t sign the last sig and starts another round of sig. Or even a group of colluding guards.)

After payment txn is done and confirmed enough, the reward payment is generated. 
On ergo, payment and reward txns are combined.
Reward to involved watchers and all guards
A combination of collected fees and Rosen token
Configurable by guards

# Technical specification and deployment
For a detailed description on how it works and the technical description please read [Rosen Bridge Introduction](https://github.com/rosen-bridge/.github/blob/master/profile/README.md) and [Rosen Bridge Contracts and Deployment](https://github.com/rosen-bridge/docs/blob/master/readme/rosen/contracts/contract.md). Also, please refer to this [step-by-step toturial](https://github.com/rosen-bridge/docs/blob/master/README%20(1).md) on how to deploy watchers and guards.



