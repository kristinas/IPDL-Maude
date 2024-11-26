**Equational Proofs for Distributed Cryptographic Protocols**

In cryptography, interactive, distributed cryptographic protocols are most often proved secure using the simulation paradigm, wherein the protocol of interest is proved (approximately) equivalent to an idealization. The simulation paradigm is extremely powerful, as it allows a wide range of security properties to be captured under one definition. On the other hand, while expressive, the simulation paradigm presents extra complications for formally verifying security proofs. Proving equivalences between distributed protocols in general requires heavyweight techniques based on manually constructing so-called bisimulations (suitable relational invariants), which creates a barrier to entry for formal methods. 

We lower this barrier to entry with IPDL, or Interactive Probabilistic Dependency Logic, a new process calculus for cryptographic protocols. IPDL includes an approximate equational logic that allows computationally sound reasoning about protocols in a manner both close to the simulation paradigm and amenable for formal verification. Using IPDL, we deliver short, simulation-based proofs of variety of cryptographic protocols. 

**Installation instructions**

Install Maude following the instructions here:
http://maude.cs.illinois.edu/w/index.php/Maude_download_and_installation
then add it to the file path and start it in the repository folder. 

Alternatively, run `nix-shell -p maude`.

**IPDL**

We are currently in the middle of reorganizing the repository due to switching 
from a low-level Maude specific syntax to a user-friendly notation, and we are
translating the case studies to the new syntax. The new implementation
relies heavily on integrating IPDL as a new language in [SpeX](https://gitlab.com/ittutu/spex).

The [IPDL-Spex](https://github.com/kristinas/IPDL-Maude/tree/main/IPDL-Spex "IPDL-Spex") folder
contains the implementation of the concrete syntax. The [IPDL-AS](https://github.com/kristinas/IPDL-Maude/tree/main/IPDL-AS "IPDL-AS") folder contains the old implementation.

The [doc](https://github.com/kristinas/IPDL-Maude/tree/main/IPDL-AS/doc) folder contains a [technical report](https://github.com/kristinas/IPDL-Maude/blob/main/IPDL-AS/doc/POPL2023.pdf) describing the theory behind the tool,
and updated version of the syntax together with a description of the [Maude implementation of IPDL](https://github.com/kristinas/IPDL-Maude/blob/main/doc/IPDL-AS/tech_report.pdf) and a detailed presentation of the [case studies](https://github.com/kristinas/IPDL-Maude/blob/main/doc/IPDL-AS/case_studies.pdf):
- Symmetric-Key Encryption: CPA Security (Sec. 1) for approximate equality of protocols;
- Symmetric-Key Encryption: CPA$-To-CPA Security (Sec. 2) for approximate equality of protocols;
- Symmetric-Key Encryption: Authenticated-To-Secure Channel (Sec. 3) for approximate equality of protocols;
- Authenticated-To-Secure Channel: Diffie-Hellman Key Exchange - DHKE (Sec. 4) for approximate equality of protocols;
- Diffie-Hellman Key Exchange (DHKE) + One-Time Pad (OTP) (Sec. 5) for approximate equality 
of protocols;
- Public-Key Encryption: El Gamal (Sec. 6) for approximate equality 
of protocols;
- Oblivious Transfer: 1-Out-Of-2 Pre-Processing (Sec. 7) for exact equality of protocols;
- Multi-Party Coin Toss (Sec. 8) for induction;
- Two-Party GMW Protocol (Sec. 9) for induction;
- Multi-Party GMW Protocol (Sec. 10) for induction.

In the [lib](https://github.com/kristinas/IPDL-Maude/tree/main/IPDL-AS/lib "lib") folder  we have the formalizations of the case studies (if an item of the list above is missing here, its implementation is ongoing):
- `helloWorld.maude` is a simple example, explained in Sec. 2.7 of the
[Maude technical report](https://github.com/kristinas/IPDL-Maude/blob/main/doc/IPDL-AS/tech_report.pdf) ;
- `cpaSecurity.maude` for CPA Security;
- `pseudoRandom.maude` for CPA$-To-CPA Security;
- `secure.maude` for Authenticated-To-Secure Channel;
- `dhke.maude` for (DHKE) and (DHKE+OTP);
- `preProcessing.maude` for 1-Out-Of-2 Pre-Processing;
- `multipartyCoinToss.maude` for Multi-Party Coin Toss;
- `gmw2.maude` for Two-Party GMW Protocol;
- `gmwN.maude` for Multi-Party GMW Protocol.

To run the examples, call
`Maude> load lib/FILENAME`
in the `IPDL-AS` folder, where FILENAME is the name of one of the files in the lib folder.


Some of the case studies have been already ported to the new notation.
They can be found in the [src](https://github.com/kristinas/IPDL-Maude/tree/main/IPDL-Spex/src "IPDL-Spex") folder of the new implementation.
To test them, run
```
cd IPDL-Spex/src
maude -no-banner -allow-files run-SpeX
```
then at the SpeX prompt enter
`load dhke.ipdl`
or other file with extension `.ipdl` from the `IPDL-Spex/src` folder.

Our [core logic paper](https://dl.acm.org/doi/10.1145/3571223) on IPDL has been accepted at a premier venue in programming languages research, the Symposium on the Principles of Programming Languages (POPL 2023) in Boston, USA. The open access version can be found [here](https://hal.inria.fr/hal-03917005/file/main.pdf).

**Acknowledgement**

This project is funded through [NGI Zero Core](https://nlnet.nl/core), a fund established by [NLnet](https://nlnet.nl) with financial support from the European Commission's [Next Generation Internet](https://ngi.eu) program. Learn more at the [NLnet project page](https://nlnet.nl/project/IPDL-II).

[<img src="https://nlnet.nl/logo/banner.png" alt="NLnet foundation logo" width="20%" />](https://nlnet.nl)
[<img src="https://nlnet.nl/image/logos/NGI0_tag.svg" alt="NGI Zero Logo" width="20%" />](https://nlnet.nl/core)




