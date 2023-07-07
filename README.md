**Equational Proofs for Distributed Cryptographic Protocols**

In cryptography, interactive, distributed cryptographic protocols are most often proved secure using the simulation paradigm, wherein the protocol of interest is proved (approximately) equivalent to an idealization. The simulation paradigm is extremely powerful, as it allows a wide range of security properties to be captured under one definition. On the other hand, while expressive, the simulation paradigm presents extra complications for formally verifying security proofs. Proving equivalences between distributed protocols in general requires heavyweight techniques based on manually constructing so-called bisimulations (suitable relational invariants), which creates a barrier to entry for formal methods. 

We lower this barrier to entry with IPDL, or Interactive Probabilistic Dependency Logic, a new process calculus for cryptographic protocols. IPDL includes an approximate equational logic that allows computationally sound reasoning about protocols in a manner both close to the simulation paradigm and amenable for formal verification. Using IPDL, we deliver short, simulation-based proofs of variety of cryptographic protocols. 

**Installation instructions**

Install Maude following the instructions here:
http://maude.cs.illinois.edu/w/index.php/Maude_download_and_installation
add it to the file path and start it in the repository folder. 

Alternatively, run `nix-shell -p maude`.

The [src](https://github.com/kristinas/IPDL-Maude/tree/main/src "src") folder contains the implementation of the equality and typing judgements for reactions and protocols, in the file [syntax.maude](https://github.com/kristinas/IPDL-Maude/blob/main/src/syntax.maude "syntax.maude"). Actual proofs are written using the strategies introduced in [strategies.maude](https://github.com/kristinas/IPDL-Maude/blob/main/src/strategies.maude "strategies.maude").

The [doc](https://github.com/kristinas/IPDL-Maude/tree/main/doc) folder contains a [technical report](https://github.com/kristinas/IPDL-Maude/blob/main/doc/tech_report_popl.pdf) describing the theory behind the tool,
and updated version of the syntax together with a description of the [Maude implementation of IPDL](https://github.com/kristinas/IPDL-Maude/blob/main/doc/tech_report_maude.pdf) and a detailed presentation of the [case studies](https://github.com/kristinas/IPDL-Maude/blob/main/doc/case_studies.pdf):
- Symmetric-Key Encryption: CPA Security (Sec. 1) for approximate equality of protocols;
- Symmetric-Key Encryption: CPA$-To-CPA Security (Sec. 2) for approximate equality of protocols;
- Symmetric-Key Encryption: Authenticated-To-Secure Channel (Sec. 3) for approximate equality of protocols;
- Authenticated-To-Secure Channel: Diffie-Hellman Key Exchange - DHKE (Sec. 4) for approximate equality of protocols;
- Diffie-Hellman Key Exchange (DHKE) + One-Time Pad (OTP) (Sec. 5) for approximate equality 
of protocols;
- Oblivious Transfer: 1-Out-Of-2 Pre-Processing (Sec. 6) for exact equality of protocols;
- Multi-Party Coin Toss (Sec. 7) for induction;
- Two-Party GMW Protocol (Sec. 8) for induction.

In the [lib](https://github.com/kristinas/IPDL-Maude/tree/main/lib "lib") folder  we have the formalizations of the case studies (if an item of the list above is missing here, its implementation is ongoing):
- `helloWorld.maude` is a simple example, explained in Sec. 2.7 of the
[Maude technical report](https://github.com/kristinas/IPDL-Maude/blob/main/doc/tech_report_maude.pdf) ;
- `cpaSecurity.maude` for CPA Security;
- `pseudoRandom.maude` for CPA$-To-CPA Security;
- `secure.maude` for Authenticated-To-Secure Channel;
- `dhke.maude` for (DHKE) and (DHKE+OTP);
- `preProcessing.maude` for 1-Out-Of-2 Pre-Processing;
- `multipartyCoinToss.maude` for Multi-Party Coin Toss.

To run the examples, call
`Maude> load lib/FILENAME`
in the main folder of the repository, where FILENAME is the name of one of the files in the lib folder.

Our [core logic paper](https://dl.acm.org/doi/10.1145/3571223) on IPDL has been accepted at a premier venue in programming languages research, the Symposium on the Principles of Programming Languages (POPL 2023) in Boston, USA. The open access version can be found [here](https://hal.inria.fr/hal-03917005/file/main.pdf).

**Acknowledgement**

This project was funded through the [NGI Assure Fund](https://nlnet.nl/assure), a fund established by [NLnet](https://nlnet.nl/) with financial support from the European Commission's [Next Generation Internet](https://ngi.eu/) programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 957073.

<div align="right">
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/215262095-ab12d43a-ca8a-4d44-b79b-7e99ab91ca01.png"/>
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/221422192-60d28ed4-10bb-441e-957d-93af58166707.png"/>
    <img height="100px" src="https://user-images.githubusercontent.com/8997731/215262235-0db02da9-7c6c-498e-a3d2-7ea7901637bf.png"/>
</div>


