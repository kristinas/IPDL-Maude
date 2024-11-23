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
relies heavily on [SpeX](https://gitlab.com/ittutu/spex).
Details about the repository's content will follow soon.

Our [core logic paper](https://dl.acm.org/doi/10.1145/3571223) on IPDL has been accepted at a premier venue in programming languages research, the Symposium on the Principles of Programming Languages (POPL 2023) in Boston, USA. The open access version can be found [here](https://hal.inria.fr/hal-03917005/file/main.pdf).

**Acknowledgement**

This project was funded through the [NGI Assure Fund](https://nlnet.nl/assure), a fund established by [NLnet](https://nlnet.nl/) with financial support from the European Commission's [Next Generation Internet](https://ngi.eu/) programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 957073.

<div align="right">
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/215262095-ab12d43a-ca8a-4d44-b79b-7e99ab91ca01.png"/>
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/221422192-60d28ed4-10bb-441e-957d-93af58166707.png"/>
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/215262235-0db02da9-7c6c-498e-a3d2-7ea7901637bf.png"/>
</div>


