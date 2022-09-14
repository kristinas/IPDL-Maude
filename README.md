**Equational Proofs for Distributed Cryptographic Protocols**

In cryptography, interactive, distributed cryptographic protocols are most often proved secure using the simulation paradigm, wherein the protocol of interest is proved (approximately) equivalent to an idealization. The simulation paradigm is extremely powerful, as it allows a wide range of security properties to be captured under one definition. On the other hand, while expressive, the simulation paradigm presents extra complications for formally verifying security proofs. Proving equivalences between distributed protocols in general requires heavyweight techniques based on manually constructing so-called bisimulations (suitable relational invariants), which creates a barrier to entry for formal methods. 

We lower this barrier to entry with IPDL, or Interactive Probabilistic Dependency Logic, a new process calculus for cryptographic protocols. IPDL includes an approximate equational logic that allows computationally sound reasoning about protocols in a manner both close to the simulation paradigm and amenable for formal verification. Using IPDL, we deliver short, simulation-based proofs of variety of cryptographic protocols. 

**Installation instructions**

Install Maude following the instructions here:
http://maude.cs.illinois.edu/w/index.php/Maude_download_and_installation
add it to the file path and start it in the repository folder. 

Alternatively, run `nix-shell -p maude`.

To load the tests, call
 `Maude> load lib/testing.maude`
 
To load one of the examples, call
 `Maude> load lib/FILENAME.maude`
where FILENAME is one of multipartyCoinToss, secure, preProcessing.

**Acknowledgement**

This project was funded through the [NGI Assure Fund](https://nlnet.nl/assure), a fund established by [NLnet](https://nlnet.nl/) with financial support from the European Commission's [Next Generation Internet](https://ngi.eu/) programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 957073.


