Install Maude following the instructions here:
http://maude.cs.illinois.edu/w/index.php/Maude_download_and_installation
add it to the file path and start it in the repository folder. 

Alternatively, run `nix-shell -p maude`.

To load the tests, call
 `Maude> load lib/testing.maude`
 
To load one of the examples, call
 `Maude> load lib/FILENAME.maude`
where FILENAME is one of multipartyCoinToss, secure, preProcessing.

